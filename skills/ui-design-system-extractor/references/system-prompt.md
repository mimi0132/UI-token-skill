# System Prompt — Extraction Checklist & Quality Gate

This file is the agent's "second brain" during the override workflow. Load it during
Step 1 (extraction) and Step 5 (verification).

---

## Mindset

You are **NOT** building a component library. You are **NOT** writing new components.
You are producing a **single CSS override file** that, when imported into a project that
uses an existing component library, changes the visual style of every existing component
in that project.

If at any point you feel the urge to write a `Button.vue` or `Button.tsx` — **stop**. The
user wants to keep their existing code. They just want it to look different.

---

## Phase 1: Pre-Work (Always Do)

Before touching any file, confirm:

- [ ] **The user has an existing component library.** If not, this skill does not apply.
      Ask "你项目里现在用的是哪个组件库?" and stop until you have an answer.
- [ ] **You know the design source.** Figma URL / local image path / hand-written tokens file.
- [ ] **You know the override scope.** Full re-skin / color only / custom (user specifies).
- [ ] **You know the target library version.** Element Plus 2.x vs 1.x have different
      variable names. Ask if unclear.

If any of these is missing, ask. Do not guess.

---

## Phase 0: Target Library Discovery (DO THIS FIRST for any lib)

> **This skill supports exactly 3 component libraries**: Element Plus 2.4, Ant Design v5,
> and the 中创 fork (a fork of Element Plus). If the project uses anything else
> (shadcn, Naive UI, MUI, Chakra, Vuetify, etc.), **STOP** and tell the user:
> "This skill supports Element Plus 2.4, Ant Design v5, and 中创 fork. Your project
> uses `{detected}`. Add one of the supported libs, or extend this skill."
>
> Why we limit: every lib has its own variable structure. Supporting more libs means
> maintaining 3+ parallel templates, more bugs, and inconsistent results. Better to be
> excellent at 3 than mediocre at 8.

### Step 1: Ask the user which library they use (mandatory)

The user may not know the version. The user may not know the lib name precisely. That's
fine — ask them directly using AskUserQuestion.

**Question**: "你项目中使用的组件库是？"
**Options**:
- Element Plus 2.4
- Ant Design v5
- 中创 fork

### Step 2: If user is unsure, run optional detection script

If user says "我不知道" or "不确定":

```bash
cd /Users/zhaozihan/Desktop/UI-提取- agent  # or wherever the project root is

# Read all relevant deps from package.json in one shot
node -e "
const p = require('./package.json');
const all = { ...p.dependencies, ...p.devDependencies };
console.log(JSON.stringify({
  'element-plus': all['element-plus'],
  'antd':         all['antd'],
}, null, 2));
"
```

Then classify:

| User response / Detected | Classification                  | Next step                         |
|---------------------------|---------------------------------|-----------------------------------|
| "Element Plus" / "EP"     | Vanilla Element Plus            | Use § 1 (EP) template             |
| "Ant Design" / "AntD"     | Ant Design v5                   | Use § 2 (AntD) template           |
| "中创" / "中创组件库"      | 中创 fork                       | Use § 3 (中创) template            |
| `element-plus` detected   | Vanilla Element Plus (assumed)  | Use § 1 (EP) template             |
| `antd` detected, ≥ 5.0    | Ant Design v5                   | Use § 2 (AntD) template           |
| `antd` detected, < 5.0    | Ant Design v4 (legacy)          | **STOP**, ask user to upgrade to v5 |
| Multiple of above         | Multi-lib project               | **STOP**, ask user which to target |
| None of the 3 supported   | Other / unknown                 | **STOP**, message above           |

To detect the 中创 fork (if element-plus is detected):

```bash
# Check for --cv- variables in compiled CSS, or cv- class names in scss
grep -rh -- "--cv-" node_modules/element-plus/lib/theme-chalk/*.scss 2>/dev/null | head -3
grep -rh "cv-avatar\|cv-button\|cv-message\|cv-popconfirm\|cv-tooltip" \
  node_modules/element-plus/lib/theme-chalk/*.scss 2>/dev/null | head -3
```

If either returns hits → it's the 中创 fork. Otherwise it's vanilla EP.

> **Why the runtime dump below is no longer the primary source of truth**: For the 3
> supported libs, we already maintain canonical templates in `token-spec.md` § 1/2/3.
> The dump is used to **cross-check** the template against the installed version (in
> case the user has a newer EP, e.g. 2.5, where new variables may have been added).

### Step 3: Cross-check variables (optional, if user has lib installed)

If the user has the library installed and wants to verify variables match, use the runtime
dump. Otherwise, skip this step — the built-in templates are already verified.

```js
// Browser DevTools console on a page that uses the lib
const allVars = {}
for (const sheet of document.styleSheets) {
  try {
    for (const rule of sheet.cssRules) {
      if (rule.style && (rule.selectorText === ':root' || rule.selectorText === ':host')) {
        for (const prop of rule.style) {
          if (prop.startsWith('--')) allVars[prop] = rule.style.getPropertyValue(prop)
        }
      }
    }
  } catch (e) {}
}

const used = new Set()
for (const sheet of document.styleSheets) {
  try {
    for (const rule of sheet.cssRules) {
      const text = rule.cssText || ''
      for (const m of text.matchAll(/var\(\s*(--[a-z0-9-]+)/g)) used.add(m[1])
    }
  } catch (e) {}
}

console.log('Defined variables:', Object.keys(allVars).length)
console.log('Referenced variables:', used.size)
console.log('Variables in template but NOT in installed lib (template is too new):',
  [...used].sort().join('\n'))
```

**If the installed lib has variables not in the template** (e.g. EP 2.5 added new ones):
add them to `token-spec.md` § 1 before generating tokens. **Never silently skip them.**

**If the template has variables not in the installed lib**: remove them from the
generated token file — they'd be dead code.

### Step 4: Categorize the variables (informational only)

```js
const groups = { color: [], size: [], radius: [], shadow: [], space: [], motion: [], font: [], border: [], other: [] }
for (const v of used) {
  if (/color|bg|fill|stroke/i.test(v))          groups.color.push(v)
  else if (/size|height|font-size|width/i.test(v)) groups.size.push(v)
  else if (/radius|rounded/i.test(v))           groups.radius.push(v)
  else if (/shadow|elevation/i.test(v))         groups.shadow.push(v)
  else if (/space|padding|margin|gap/i.test(v)) groups.space.push(v)
  else if (/motion|transition|duration/i.test(v)) groups.motion.push(v)
  else if (/font|line-height|letter/i.test(v))  groups.font.push(v)
  else if (/border/i.test(v))                   groups.border.push(v)
  else                                          groups.other.push(v)
}
console.table(Object.fromEntries(Object.entries(groups).map(([k,v]) => [k, v.length])))
```

This tells you **how many variables of each type** the lib actually uses — useful for
sizing the token file before generating.

### Step 4: Map the prefix — only 3 cases, no custom mapping needed

| Lib declares variables starting with | Classification                  | Template             |
|--------------------------------------|---------------------------------|----------------------|
| `--el-`                              | Vanilla Element Plus            | § 1 (EP)             |
| `--el-` + `--cv-`                    | 中创 fork                       | § 3 (中创)            |
| `--ant-`                             | Ant Design v5                   | § 2 (AntD)           |
| `--my-`, `--company-`, `--dc-`, etc. | **Renamed fork of unsupported lib** | **STOP** — not supported |

The previous version of this skill supported "renamed prefix" (e.g. `--my-color-primary`)
as a fork of Element Plus. **We removed this support** because the 中创 fork is the
only known EP fork in our user's portfolio, and supporting arbitrary prefix renames
doubles the template maintenance cost. If a renamed fork appears, tell the user the
limitation and offer to add it as a 4th template.

### Step 5: Pick the template, fill it from the design source

1. Open `references/token-spec.md`.
2. For EP / 中创, use § 1 (or § 3 for cv- extensions). For AntD, use § 2.
3. Fill the template with the colors / sizes / radii / etc. from the design source.
4. Use `color-mix()` to derive missing stops (see § 0.5 in token-spec).
5. Generate the override file (see `override-patterns.md` for the per-lib template).

### The "I don't have node access" fallback

If the agent can't read `node_modules/` (sandbox, CI, browser-only context), it
should:
1. Use the runtime dump (Step 3) as the source of truth for variable names
2. Ask the user to paste the package name and version
3. Use the grep approach to find the package path
4. If all else fails, ask the user to provide a sample CSS file from the lib

**Never assume vanilla Element Plus / Ant Design.** Always verify via the auto-detect.

---

## Phase 1.5: Color System Extension (the "fill the gaps" rule)

> **The design source is sampling points, not a complete palette.** A Figma frame
> typically shows you 1-3 stops of the primary and 1-2 swatches of each semantic
> color. Element Plus 2.4 / Ant Design v5 need **40-100 color variables** to fully
> re-skin — but their **structures are different** (EP = 7 stops, AntD = 10 stops).
> The gap between "what the design gives" and "what the lib needs" is what the agent
> closes. **The lib's structure (not a generic 10-stop scale) determines the variables.**

### The rule: one anchor + math, matched to the lib's structure

For each color scale (primary, neutral, success, warning, danger, info), pick **one
anchor** matching what the lib calls it. Derive the other stops using `color-mix()`.
The number of stops and their names **MUST match the lib** (see
[references/token-spec.md § 1/2/3](token-spec.md)):

- **Element Plus 2.4 / 中创**: anchor = `base`, derive `light-3/5/7/8/9` + `dark-2` = **7 stops**
- **Ant Design v5**: anchor = 6 stop, derive `1/2/3/4/5/7/8/9/10` = **10 stops**

**Why this matters**: hand-picking 9 hex values for each of 6 scales is 54+ decisions
the agent can get wrong, and any tweak to the brand color would force you to redo all 54.

The full algorithm + calibrated percentages is in
[references/token-spec.md § 0.5](token-spec.md#05-color-derivation-algorithm--when-the-design-only-gives-1-stop).

**Element Plus / 中创 7-stop scale** (anchor = base, mixed with white/black):

```css
--color-primary-base:     #6366F1;   /* anchor from design, treated as 500 */
--color-primary-light-3:  color-mix(in srgb, var(--color-primary-base) 75%, white);  /* 300 */
--color-primary-light-5:  color-mix(in srgb, var(--color-primary-base) 90%, white);  /* 100 */
--color-primary-light-7:  color-mix(in srgb, var(--color-primary-base) 96%, white);  /* 50  */
--color-primary-light-8:  color-mix(in srgb, var(--color-primary-base) 98%, white);
--color-primary-light-9:  color-mix(in srgb, var(--color-primary-base) 99%, white);
--color-primary-dark-2:   color-mix(in srgb, var(--color-primary-base) 84%, black);  /* 700 */
```

**Ant Design v5 10-stop scale** (anchor = 6, mixed with white/black):

```css
--color-primary-6: #6366F1;            /* anchor from design */
--color-primary-1: color-mix(in srgb, var(--color-primary-6) 10%, white);
--color-primary-2: color-mix(in srgb, var(--color-primary-6) 20%, white);
--color-primary-3: color-mix(in srgb, var(--color-primary-6) 35%, white);
--color-primary-4: color-mix(in srgb, var(--color-primary-6) 55%, white);
--color-primary-5: color-mix(in srgb, var(--color-primary-6) 80%, white);
--color-primary-7: color-mix(in srgb, var(--color-primary-6) 80%, black);
--color-primary-8: color-mix(in srgb, var(--color-primary-6) 65%, black);
--color-primary-9: color-mix(in srgb, var(--color-primary-6) 45%, black);
--color-primary-10: color-mix(in srgb, var(--color-primary-6) 25%, black);
```

**Change the anchor once, the whole scale re-derives automatically.**

### "What to extract vs derive" decision

| Design source gives              | Agent does                                          |
|----------------------------------|-----------------------------------------------------|
| Only `--color-primary-base` (EP) | Use as anchor, derive light-3/5/7/8/9 + dark-2      |
| Only `--color-primary` (AntD)    | Use as anchor (treated as 6 stop), derive 1-5/7-10  |
| `base` + `light-9` (EP)          | Use both, derive the other 5 stops                  |
| `color-X-6` + `color-X-3` (AntD) | Use both, derive the other 8 stops                  |
| All 7/10 stops                   | Use all, skip derivation                            |
| Only "main blue" + "hover"       | Treat main as base/6, hover as light-5/3, derive    |
| No primary at all                | Ask the user, or use --color-primary-base: #6366F1  |
| Neutral grays not in design      | Derive from primary hue, fully desaturated          |
| Success/warning/danger not in design | Use standard defaults (green/amber/red) + derive |

**Fill in the gaps, don't fabricate**: If the design gives 1 stop explicitly, use
that exact value as the anchor and derive the rest. Don't override the explicitly
given value with a derived one.

### Per-lib token extraction (NOT a generic 50..900 scale)

The exact variable name list comes from the lib template, not from a generic
10-stop scale. Open `references/token-spec.md` and use:

- § 1: Element Plus 2.4 (or 中创 § 3) — 7-stop scale, EP variable names
- § 2: Ant Design v5 — 10-stop scale, AntD seed token names

Fill in the anchor from the design, derive the rest with `color-mix()`. Do **not**
emit `--color-primary-50..900` for EP — EP uses `light-9/8/7/5/3 + dark-2`.

### Quality gate

Before generating the override file, run this check **in the browser, on the
preview page**:

```js
const check = (selector) => {
  const cs = getComputedStyle(document.querySelector(selector))
  const required = []
  // EP / 中创: 6 semantic colors × 7 stops + base
  for (const role of ['primary', 'success', 'warning', 'danger', 'info']) {
    required.push(`--color-${role}-base`)
    for (const stop of ['light-3', 'light-5', 'light-7', 'light-8', 'light-9', 'dark-2']) {
      required.push(`--color-${role}-${stop}`)
    }
  }
  // AntD: 6 colors × 10 stops
  // for (const role of ['primary', 'success', 'warning', 'danger', 'info']) {
  //   for (let i = 1; i <= 10; i++) required.push(`--color-${role}-${i}`)
  // }
  const missing = required.filter(v => !cs.getPropertyValue(v).trim())
  if (missing.length) console.error(`[${selector}] Missing tokens:`, missing)
}
check(':root')
```

If any are missing, **add them to `colors.css` before generating the override**.
Gaps here = the "前边换了后边没变" failure mode.

### Why this matters

The user will say "I gave you a color, why isn't the rest of the design updated?"
because the lib has 40-100 color variables and they only gave you 2. The answer is
"we derived the rest algorithmically to match the lib's structure" — not "we missed it".
Make this derivation **explicit in the README** so the user knows the magic happened.

---

## Phase 1.6: Multi-Dimension Derivation (color is not the only one)

The "single anchor + math" pattern applies to **6 categories**, not just color.
Apply the same fill-the-gaps principle to each. Full algorithm + tunable knobs in
[references/token-spec.md § 0.5](token-spec.md#05-other-derivable-variable-categories).

### Quick reference

| Category        | Anchor(s)                                                              | Math                          |
|-----------------|------------------------------------------------------------------------|-------------------------------|
| **Color (EP)**  | `--color-primary-base` (or success/warning/danger/info base)            | `color-mix(in srgb, …, white\|black)` |
| **Color (AntD)**| `--color-primary-6` (or success/warning/danger/info 6)                 | `color-mix(in srgb, …, white\|black)` |
| **Spacing**     | `--space-base: 4px` (or 8px)                                           | `calc(var(--space-base) * N)` |
| **Radius**      | `--radius-md: 4px` (EP default) / `borderRadius: 6` (AntD, single value) | direct or `calc(...)`         |
| **Font size**   | `--font-size-base: 16px` + `--font-size-ratio: 1.25`                   | `calc(var(--font-size-base) * pow(var(--font-size-ratio), N))` |
| **Line height** | `--line-height-base: 1.5`                                              | direct value or `calc(...)`     |
| **Border width**| `--border-width-base: 1px`                                             | `calc(var(--border-width-base) * N)` |
| **Motion dur.** | `--motion-duration-base: 200ms`                                        | `calc(var(--motion-duration-base) * K)` |

### "Fill the gaps" for each category

Same rule as color. From the design, extract the anchor(s) the design gives
explicitly. Derive the rest.

```js
// Mental model: what does the design typically give?
const typicalDesignGives = {
  color:        '1-3 primary stops + 1-2 semantic swatches',  // most common
  spacing:      '0-3 specific paddings (rest = derive)',         // sparse
  radius:       '1-2 specific radii (sm/md/lg = derive)',        // sparse
  fontSize:     'body + 1-2 heading sizes (rest = derive)',      // moderate
  lineHeight:   'rarely specified, derive or use 1.2/1.5/1.75', // very sparse
  borderWidth:  'almost never specified, default to 1px',        // absent
  motion:       'rarely specified, default to 150/200/300',      // absent
}
```

**Two categories cannot be derived** — they have multiple dimensions or are
industry-fixed. For these, use a small named set:

| Category      | Why not derivable                         | What to do                          |
|---------------|-------------------------------------------|-------------------------------------|
| Font family   | Design decision, not mathematical         | Pick from design or use a system stack |
| Font weight   | Industry fixed (400/500/600/700)          | Use the 4 values directly           |
| Shadow        | 5 dimensions (x, y, blur, spread, color)  | Use 3-5 named tokens (sm/md/lg/overlay) |
| Grid columns  | Industry fixed (12/16/24)                 | Use the value the design implies    |
| Breakpoints   | Industry fixed (640/768/1024/1280/1536)   | Use the value the design implies    |
| Motion easing | 3-4 industry curves                       | Pick from standard set              |

### Worked example: "design only shows 16px body text"

```css
/* User gave: 16px body */
:root {
  --font-size-base:  16px;        /* extracted from design */
  --font-size-ratio: 1.25;        /* picked: Material major third */

  --font-size-xs:   calc(var(--font-size-base) / var(--font-size-ratio) / var(--font-size-ratio));   /* 10.24 */
  --font-size-sm:   calc(var(--font-size-base) / var(--font-size-ratio));                            /* 12.8  */
  --font-size-md:   var(--font-size-base);                                                           /* 16    */
  --font-size-lg:   calc(var(--font-size-base) * var(--font-size-ratio));                            /* 20    */
  --font-size-xl:   calc(var(--font-size-base) * var(--font-size-ratio) * var(--font-size-ratio));   /* 25    */
  /* ... 5xl/6xl auto-derived if needed */
}
```

User changes mind → "we want bigger headings": bump `--font-size-ratio: 1.25` → `1.5`. Whole scale re-derives. No touching 10 values.

### Quality gate (multi-dimension version)

Before generating the override file, check all anchors are defined:

```js
// EP / 中创 (7 stops): anchor 是单色 + 派生 light-3/5/7/8/9 + dark-2
const anchors = [
  '--color-primary', '--color-success', '--color-warning', '--color-danger', '--color-info',
  '--space-base', '--radius-base',
  '--font-size-base', '--font-size-ratio',
  '--line-height-base',
  '--border-width-base',
  '--motion-duration-base',
]
const missing = anchors.filter(a => !getComputedStyle(document.documentElement).getPropertyValue(a).trim())
if (missing.length) console.error('Missing anchors:', missing)
// AntD 走 10 档,独立 anchor 表(--color-primary-50..900),见 references/token-spec.md § 2
```

If any are missing, ask the user OR pick a sensible default from the design source. **Never generate a `colors.css` / `spacing.css` / etc. with hand-picked values for derivable categories.**

---

## Phase 2: Token Extraction Checklist

> **Important**: this checklist below is the **generic / lib-agnostic** view. The
> actual variable names to emit come from the lib template in
> [references/token-spec.md](token-spec.md):
> - **Element Plus 2.4 / 中创 fork** → § 1 (or § 3) — 7-stop `light-9..dark-2`, `--el-*` override targets
> - **Ant Design v5** → § 2 — 10-stop `1..10`, seed-token names
>
> The checklist below is what to **look for in the design source**. The lib template
> is what to **write to the output file**. **Do not blindly emit `--color-primary-50..900`
> for an EP project** — that is the "重新创建组件库" failure mode.

When reading the design source, extract the following. **Be concrete, not vague.**
The new design is the source of truth for all visual variables; the existing component
library keeps its own interaction behavior. We are doing **visual re-skin**, not a behavior
replacement.

### Colors

| What to look for     | Where to find it                          | EP / 中创 variable (7 stops)               | Ant Design v5 variable (10 stops)   |
|----------------------|-------------------------------------------|--------------------------------------------|-------------------------------------|
| Primary brand color  | Buttons, links, focus rings               | `--color-primary-base` (anchor)            | `--color-primary-6` (anchor)        |
| Primary tints/shades | Hover/active states of primary             | `--color-primary-{light-3,5,7,8,9,dark-2}` | `--color-primary-{1..10}`           |
| Neutral grays        | Body text, borders, dividers, surfaces    | `--color-neutral-{light-9..dark-2}`        | `--color-neutral-{1..10}`           |
| Success              | Success toasts, "OK" badges, valid input   | `--color-success-{base,light-3..9,dark-2}` | `--color-success-{1..10}`           |
| Warning              | Warning toasts, "caution" indicators      | `--color-warning-{base,light-3..9,dark-2}` | `--color-warning-{1..10}`           |
| Danger / Error       | Error states, destructive buttons         | `--color-danger-{base,light-3..9,dark-2}`  | `--color-danger-{1..10}`            |
| Info                 | Info toasts, neutral notifications        | `--color-info-{base,light-3..9,dark-2}`    | `--color-info-{1..10}`              |
| Background surfaces  | Page background, card background          | `--color-bg-{page,surface,container,overlay}` | `--color-bg-{container,elevated,layout}` |
| Text colors          | Body, headings, captions, disabled        | `--color-text-{primary,regular,secondary,placeholder,disabled}` | `--color-text-{heading,primary,secondary,tertiary,disabled,placeholder}` |
| Borders              | Default, strong, subtle                   | `--color-border-{default,light,lighter,extra-light,dark}` | `--color-border-{secondary,default,colorSplit,colorBgContainer}` |

**Rule**: If Figma only gives you the base/6 stop, derive the rest using
`color-mix(in srgb, var(--color-primary-base) X%, white|black)` for EP, or
`color-mix(in srgb, var(--color-primary-6) X%, white|black)` for AntD. Do not
hand-pick hex values for the other stops — use the math.

### Typography

| What to look for | Where to find it       | Variable           |
|------------------|------------------------|--------------------|
| Heading font     | Largest text on screen | `--font-family-heading` |
| Body font        | Paragraphs, buttons    | `--font-family-body` |
| Mono font        | Code blocks            | `--font-family-mono` |
| Body size        | Default paragraph      | `--font-size-md` (16px) |
| Sizes            | H1-H6 hierarchy        | `--font-size-{xs,sm,md,lg,xl,2xl,3xl,4xl}` |
| Weights          | regular / medium / semibold / bold | `--font-weight-{regular,medium,semibold,bold}` |
| Line heights     | heading / body / long-form | `--line-height-{tight,normal,relaxed}` |

### Spacing

| What to look for | Where to find it           | Variable        |
|------------------|----------------------------|-----------------|
| Component padding | Button / input padding   | `--space-2` to `--space-4` |
| Element gap       | Between buttons in a group | `--space-2` to `--space-4` |
| Section gap       | Between form sections     | `--space-8` to `--space-16` |
| Page padding      | Outer page margins        | `--space-6` to `--space-12` |

**Rule**: Round everything to a multiple of 4. If the design says 13px padding, use 12.

### Radius

| What to look for | Where to find it        | Variable       |
|------------------|------------------------|----------------|
| Sharp / 0        | Some enterprise UIs    | `--radius-none` |
| Subtle           | Inputs, tags           | `--radius-sm` (2-4px) |
| Standard         | Buttons, cards, modals | `--radius-md` (6-8px) |
| Pronounced       | Hero cards, large CTAs | `--radius-lg` (12-16px) |
| Pill             | Avatars, tags, status  | `--radius-full` (9999px) |

### Shadow

| What to look for | Where to find it          | Variable       |
|------------------|--------------------------|----------------|
| Subtle           | Input borders, resting   | `--shadow-sm` |
| Standard         | Buttons, cards, hover    | `--shadow-md` |
| Pronounced       | Dropdowns, popovers      | `--shadow-lg` |
| Overlay          | Modals, dialogs          | `--shadow-overlay` |
| Colored          | Glow effects, brand tints| `--shadow-{primary,success,danger}-glow` |

### Border

| What to look for | Where to find it          | Variable       |
|------------------|--------------------------|----------------|
| Default width    | Inputs, cards, dividers  | `--border-width-default` (1px) |
| Strong width     | Emphasized containers    | `--border-width-strong` (2px) |
| Style            | Solid / dashed / dotted  | `--border-style-default` |

### Layout / Grid

| What to look for        | Where to find it        | Variable                  |
|-------------------------|-------------------------|---------------------------|
| Grid columns            | Main page layouts       | `--grid-columns` (12 / 16 / 24) |
| Gutter                  | Between grid items      | `--grid-gutter` (16 / 24 / 32) |
| Container max width     | Page container          | `--container-max-width` (1200 / 1440 / 1600) |
| Container padding       | Inner page padding      | `--container-padding` (16 / 24 / 32) |
| Breakpoint - sm         | Mobile / tablet         | `--breakpoint-sm` (640) |
| Breakpoint - md         | Tablet                  | `--breakpoint-md` (768) |
| Breakpoint - lg         | Desktop                 | `--breakpoint-lg` (1024) |
| Breakpoint - xl         | Wide desktop            | `--breakpoint-xl` (1280) |
| Breakpoint - 2xl        | Ultra-wide              | `--breakpoint-2xl` (1536) |

**Rule**: If the design uses a 24-column grid (Material/IBM style), preserve it even
if the original lib uses 12 columns. We override the relevant lib variables
(`--ant-grid-columns`, `--el-grid-columns`, etc.) or use direct CSS.

### Icon / Sizing

| What to look for | Where to find it      | Variable            |
|------------------|-----------------------|---------------------|
| Icon size sm     | Inline icons, button  | `--icon-size-sm` (12) |
| Icon size md     | Default icons         | `--icon-size-md` (16) |
| Icon size lg     | Standalone icons      | `--icon-size-lg` (20) |
| Icon size xl     | Hero / feature icons  | `--icon-size-xl` (24) |
| Control height sm| Compact controls      | `--control-height-sm` (24) |
| Control height md| Default controls      | `--control-height-md` (32) |
| Control height lg| Emphasized controls   | `--control-height-lg` (40) |

### Motion (Optional but Recommended)

| What to look for  | Where to find it    | Variable            |
|-------------------|---------------------|---------------------|
| Easing standard   | All transitions     | `--motion-easing-standard` (cubic-bezier(0.4, 0, 0.2, 1)) |
| Easing emphasis   | Hover, focus        | `--motion-easing-emphasis` (cubic-bezier(0.2, 0, 0, 1)) |
| Duration fast     | Micro-interactions  | `--motion-duration-fast` (150ms) |
| Duration base     | Default transitions | `--motion-duration-base` (200ms) |
| Duration slow     | Page-level          | `--motion-duration-slow` (300ms) |

**Note**: Some libs (Element Plus, Ant Design) hardcode durations. We can override
specific selector animations but not all motion. Be explicit in the README about
which motions are theme-controlled and which inherit from the lib.

---

## Phase 3: Override File Checklist

Before writing `overrides/<lib>-theme-override.css`, read the corresponding section of
[references/override-patterns.md](override-patterns.md) for your detected lib:

- **Element Plus 2.4** → `§ A. EP override pattern`
- **Ant Design v5** → `§ B. AntD override pattern` (uses `<ConfigProvider theme={{ token }}>` wrapper, not a CSS file)
- **中创 fork** → `§ C. 中创 override pattern` (EP base + `--cv-*` extensions)

These 3 are the only supported libs. If your project uses a 4th lib, **STOP** and
add it to `override-patterns.md` first, or refuse the task.

Verify the override covers:

- [ ] **All primary light/dark stops** (`--<lib>-color-primary-light-3/5/7/8/9` and
      `--<lib>-color-primary-dark-2` for EP / 中创; `colorPrimary-1..10` for AntD).
      Missing any of these causes hover/active state color drift — the most common
      visual bug.
- [ ] **All semantic colors** (success / warning / danger / info), each with their
      light/dark variants if the lib uses them.
- [ ] **All radius variables** that the lib exposes (e.g. Element Plus has
      `--el-border-radius-base` / `--el-border-radius-small` / `--el-border-radius-round`;
      AntD has `borderRadius` + per-component `borderRadiusLG`/`borderRadiusSM`).
- [ ] **All text color variables** (primary, regular, secondary, placeholder, disabled).
- [ ] **All background variables** (page, surface, container, overlay).
- [ ] **All border color variables** (default, light, lighter, extra-light, dark).
- [ ] **Font family** at minimum.

A "complete" override for Element Plus is ~80 lines of CSS. Don't be tempted to trim
it down — every variable exists for a reason.

---

## Phase 4: Preview Checklist

The preview page is the user's acceptance test. Before declaring done, verify:

- [ ] Preview loads without console errors
- [ ] Target component library CSS is loaded (CDN or local)
- [ ] Override file is loaded **after** the lib's CSS
- [ ] At least 5 common components are visible: **Button, Input, Tag, Card, Dialog/Modal**
  - **真实标准**: 8 大类 **50+ 组件必须全覆盖**,完整清单见
    `references/preview-comprehensive.md`。
  - **少一个 = 不算交付**。5 个覆盖率 < 10%,组件库一般 50+ 个,preview 的核心
    目的就是"看到全库都跟着变了",3-5 个远远不够。
  - 每个组件至少展示 2 个状态(default + 1 其他: hover / disabled / focus /
    active / loading)
  - 必须用真实组件类名(`el-button el-button--primary`),**禁止**用 `.btn-primary`
    这样的自定义类(那样无法验证 override 对真实库生效)
- [ ] Each component shows at least: default, hover, disabled, active states
- [ ] Color tokens are visible as swatches (with token names + HEX)
- [ ] Typography tokens are visible as labeled samples
- [ ] Preview works in Chrome / Safari / Firefox (if cross-browser is a concern)

---

## Phase 4.5: Composite Component & Hardcoded Color Audit

The most common failure mode of theme overrides is **"primary buttons changed, but
the disabled / hover / focus / nested states stayed the old color"**. This phase
exists to catch it before declaring done.

### Why cascade doesn't always work

1. **SCSS-derived variables are pre-computed.** `--el-color-primary-light-3` is
   `mix(white, #409EFF, 30%)` baked at SCSS compile time. Changing
   `--el-color-primary` does NOT re-derive the light stops.
2. **Hardcoded hex values exist.** Some components (`el-textarea__inner`, complex
   pickers, animation keyframes) have hardcoded colors that bypass the variable
   system.
3. **Composite components have their own tokens.** `<el-card>` background uses
   `--el-fill-color-blank`, not `--el-color-primary`. If your new design changes
   "page background", you must override card/table/modal-specific tokens too.

### Audit checklist — every component, every state

For each component in the design, verify these states render consistently:

| Component              | States to verify                                                  | Variables that MUST be in the override |
|------------------------|-------------------------------------------------------------------|----------------------------------------|
| **Button (primary)**   | default / hover / active / disabled / focus / loading             | `--<lib>-color-primary`, `-light-3/5/7/8/9`, `-dark-2` |
| **Button (default)**   | default / hover / active / disabled                                | `--<lib>-border-color`, `-text-color-regular`, `-fill-color-blank` |
| **Input**              | default / hover / focus / disabled / error / readonly             | `--<lib>-input-border-color`, `-text-color-regular/placeholder`, `-fill-color` |
| **Tag**                | default + each color (primary / success / warning / danger / info) | All semantic color light/dark stops  |
| **Card**               | default / hover (if interactive)                                   | `--<lib>-bg-color`, `-border-color-lighter` |
| **Dialog / Drawer**    | default + overlay                                                  | `--<lib>-bg-color`, `--<lib>-bg-color-overlay` (dialog 自身背景), `--<lib>-mask-color` (遮罩层) |
| **Table**              | default / striped / hover / selected / empty                       | `--<lib>-table-border-color`, `-table-row-hover-bg-color`, `-table-header-bg-color` |
| **Select / Dropdown**  | default / hover / open / selected / disabled                       | `--<lib>-border-color-hover`, `-fill-color-light`, `-text-color-primary` |
| **Form**               | default / error / disabled                                         | `--<lib>-form-item-label-color`, `-form-item-error-color` |
| **Menu / Submenu**     | default / hover / active                                           | `--<lib>-menu-hover-bg-color`, `-menu-text-color`, `-menu-active-color` |
| **Pagination**         | default / hover / active / disabled                                | `--<lib>-pagination-hover-color`, `-color-primary` |
| **Message / Notification** | default + each type                                              | All semantic color stops              |
| **Loading**            | default                                                            | `--<lib>-color-primary` (spinner)     |
| **Skeleton**           | default                                                            | `--<lib>-skeleton-color`              |

### How to detect hardcoded values (browser audit)

Run this in DevTools console on the preview page:

```js
// Find all elements where some property is a hardcoded hex
// (not coming from a CSS variable)
const hardcoded = []
document.querySelectorAll('*').forEach(el => {
  const cs = getComputedStyle(el)
  for (const prop of ['color', 'background-color', 'border-color', 'fill', 'stroke']) {
    const v = cs[prop]
    if (v && v.startsWith('rgb') && !v.includes('var(')) {
      hardcoded.push({ tag: el.tagName, class: el.className, prop, value: v })
    }
  }
})
console.table(hardcoded.slice(0, 50))
```

If you see hardcoded colors in *target* elements (e.g., a Button showing as
`rgb(64, 158, 255)` after override), add a direct CSS rule to the override file:

```css
:where(.el-button.el-button--primary) {
  background-color: var(--el-color-primary);
  border-color: var(--el-color-primary);
}
```

### Defensive override pattern

For each lib, append a defensive block to the override file that re-asserts the
high-traffic composite components:

```css
/* Element Plus — defensive composite component override */
:root {
  /* Re-assert after lib CSS in case any selector is more specific */
  --el-button-bg-color:           var(--color-primary);
  --el-button-border-color:       var(--color-primary);
  --el-button-hover-bg-color:     var(--color-primary-light-3);
  --el-button-active-bg-color:    var(--color-primary-dark-2);
  --el-button-disabled-bg-color:  var(--color-bg-secondary);
}
```

This is **belt and suspenders** — usually not needed if Phase 3 is done correctly,
but it eliminates the most common "only 80% of components updated" complaint.

---

## Phase 5: README Checklist

The README must answer, in this order:

1. **3-line integration** (copy-paste ready)
2. **One line for how to change a token** ("改 `tokens/colors.css` 里的 `--color-primary`")
3. **A list of what was NOT overridden** (e.g. "字体、间距沿用原样,因为你选了只换颜色")
4. **Troubleshooting** (the 3 most common issues, see below)

### Top 3 Troubleshooting Issues

| Problem                                          | Cause                                          | Fix                                                |
|--------------------------------------------------|------------------------------------------------|----------------------------------------------------|
| "我改了 token 颜色但组件没变"                   | Override 引入顺序错了                          | 确保先引 lib 的 CSS,再引 `theme.css`,再引 override |
| "某个组件不响应"                                  | 该组件用了 hardcoded 颜色,不在 CSS 变量体系里 | 用 DevTools 看 computed style,提 issue             |
| "颜色看着太淡/太深"                              | color-mix 推导的档位和你的设计稿不完全一致     | 在 `tokens/colors.css` 显式覆盖具体档位           |

---

## Quality Gate (Run Before Declaring Done)

```bash
# 1. Seven token files exist
test -f tokens/theme.css && \
test -f tokens/colors.css && \
test -f tokens/typography.css && \
test -f tokens/spacing.css && \
test -f tokens/radius.css && \
test -f tokens/border.css && \
test -f tokens/motion.css

# 2. Override file exists and imports tokens
test -f overrides/<lib>-theme-override.css && \
grep -q "@import" overrides/<lib>-theme-override.css

# 3. No raw hex anywhere in override (everything should be var(--*))
! grep -E "#[0-9a-fA-F]{3,8}" overrides/<lib>-theme-override.css

# 4. No raw px in override
#    Allow-list: control-height / breakpoint / container-max-width / icon-size / focus-ring-width
#    These are static structural values required by lib APIs and cannot use var() derivation.
RAW_PX=$(grep -E "[0-9]+px" overrides/<lib>-theme-override.css | grep -vE "(control-height|breakpoint|container-max-width|icon-size|focus-ring-width)" || true)
[ -z "$RAW_PX" ] || { echo "raw px found in override:"; echo "$RAW_PX"; exit 1; }

# 5. No raw hex in tokens EXCEPT for the 50-900 color scales (where
#    the base 500+ is defined explicitly; lighter/darker shades must
#    be color-mix derivations, not hand-rolled hex)
grep -E "color-mix\(in srgb" tokens/colors.css

# 6. Preview file exists
test -f preview/comprehensive-preview.html

# 7. README has integration snippet
grep -q "import" README.md

# 9. Preview directory strictly contains 1 .html file
#    (前几版会多生成 dashboard-xxx.html / user-list-xxx.html / preview.html, 必须拦截)
PREVIEW_COUNT=$(ls preview/*.html 2>/dev/null | wc -l | tr -d ' ')
[ "$PREVIEW_COUNT" = "1" ] || {
  echo "preview/ 应只有 1 个文件 (comprehensive-preview.html), 实际有 $PREVIEW_COUNT 个:"
  ls preview/
  exit 1
}
```

If any check fails, fix before outputting.

---

## Self-Check Questions

Before responding to the user with "done", ask yourself:

1. Did I write any `.vue` or `.tsx` file? **If yes, delete it.** This skill is theme override only.
2. Did I cover all light/dark variants of the primary color? **If no, add them.**
3. Did I generate a preview that the user can open in a browser? **If no, generate one.**
4. Could a user copy 3 lines from the README and have it work? **If no, simplify.**
5. Did I assume the user's component library instead of asking? **If yes, re-ask.**

If all 5 pass, output the result with a short summary of what was generated and the
exact integration snippet.
