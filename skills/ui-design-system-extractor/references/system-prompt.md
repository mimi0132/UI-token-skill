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

> **Most "my own component library" cases are forks of Element Plus / Ant Design
> / Naive UI with a different prefix, some variables renamed, others extended.**
> The first job of this skill is to figure out **what variables the lib actually
> exposes and uses** — not to assume it's a vanilla Element Plus.

### Step 1: Find the lib and confirm its lineage

```bash
# 1. Find the package
cat package.json | grep -E '"(@?[a-z0-9-]+/)?[a-z0-9-]+-?ui"|"element-plus"|"antd"|"naive-ui"|"@shadcn/ui"'
# → e.g. "@my-company/ui": "2.3.1"

# 2. Find the dist
ls node_modules/@my-company/ui/dist/
# → e.g. dist/index.css, dist/index.esm.js

# 3. Check if it kept the upstream prefix or changed it
grep -oE '\-\-[a-z0-9-]+' node_modules/@my-company/ui/dist/index.css | head -20
# → '--el-color-primary' (kept)  OR  '--my-color-primary' (renamed)  OR  mixed
```

### Step 2: Dump the full variable list (runtime, source-of-truth)

Browser DevTools console on a page that uses the lib:

```js
// 1. All variables defined in :root
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

// 2. Variables actually referenced (var(--xxx) usages)
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
console.log('Intersection (the ones we need to override):', used.size > 0 ? [...used].sort().join('\n') : 'cross-origin stylesheets blocked — fall back to grep')

// 3. Save to clipboard for the agent
copy([...used].sort().join('\n'))
```

**Save the output.** This is the "manifest" the agent uses to generate the override file.

### Step 3: Categorize the variables

Group the dumped variables:

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

This tells you **how many variables of each type** the lib actually uses — usually
uneven (40 colors, 12 sizes, 5 shadows, etc.).

### Step 4: Map the prefix

| Lib declares variables starting with | Classification                              | Action                                 |
|--------------------------------------|---------------------------------------------|----------------------------------------|
| `--el-`                              | Vanilla Element Plus (or unchanged fork)    | Use the existing Element Plus mapping  |
| `--ant-`                             | Vanilla Ant Design (or unchanged fork)      | Use the existing Ant Design mapping    |
| `--my-`, `--company-`, `--dc-`, etc. | **Renamed fork**                            | Create a custom mapping (see § 4)       |
| Mixed (`--el-` and `--my-`)          | Partial fork (kept upstream + added own)    | Map both, prioritize the company's own |
| Variables defined inline (no `--`)   | SCSS-only lib (Element Plus 1.x, Vuetify)   | Need to either upgrade to v2, or write CSS variable wrappers |

### Step 5: Create the custom mapping (if fork renamed the prefix)

The agent should:
1. Take the Element Plus mapping from `override-patterns.md` as a template
2. Replace `--el-` with the new prefix
3. Verify each rename against the dumped variable list (some vars may be removed, some added)
4. Generate the override file using the new prefix

**Example**: company lib renamed `--el-color-primary` to `--my-color-brand`:

```js
// Take this from override-patterns.md
const elMapping = {
  '--el-color-primary':           'var(--color-primary-500)',
  '--el-color-primary-light-3':   'var(--color-primary-400)',
  // ... 100 more
}

// Apply a prefix transform
const prefix = '--my-'
const myMapping = Object.fromEntries(
  Object.entries(elMapping).map(([k, v]) => [
    k.replace(/^--el-/, prefix),
    v
  ])
)
// → { '--my-color-primary': 'var(--color-primary-500)', ... }
```

### The "I don't have node access" fallback

If the agent can't read `node_modules/` (sandbox, CI, browser-only context), it
should:
1. Use the runtime dump (Step 2) as the source of truth
2. Ask the user to paste the package name and version
3. Use the grep approach to find the package path
4. If all else fails, ask the user to provide a sample CSS file from the lib

**Never assume vanilla Element Plus / Ant Design.** Always verify.

---

## Phase 1.5: Color System Extension (the "fill the gaps" rule)

> **The design source is sampling points, not a complete palette.** A Figma frame
> typically shows you 1-3 stops of the primary and 1-2 swatches of each semantic
> color. Element Plus / Ant Design / shadcn need ~40 color variables to fully
> re-skin. The gap between "what the design gives" and "what the lib needs" is
> what the agent closes.

### The rule: one anchor + math, not hand-picked hex

For each color scale (primary, neutral, success, warning, danger, info), pick
**one anchor** (usually the 500 stop, sometimes 600). Derive the other 9 stops
using `color-mix()`. This is **not optional** — hand-picking 9 hex values for
each of 6 scales is 54+ decisions the agent can get wrong, and any tweak to the
brand color would force you to redo all 54.

The full algorithm + calibrated percentages is in
[references/token-spec.md § 0](token-spec.md#0-color-system-derivation-algorithm).
The key formulas:

```css
/* Lighter: mix anchor with white at calibrated percentages */
--color-primary-50:  color-mix(in srgb, var(--color-primary-500)  8%, white);
--color-primary-100: color-mix(in srgb, var(--color-primary-500) 16%, white);
--color-primary-200: color-mix(in srgb, var(--color-primary-500) 28%, white);
--color-primary-300: color-mix(in srgb, var(--color-primary-500) 44%, white);
--color-primary-400: color-mix(in srgb, var(--color-primary-500) 68%, white);

/* Darker: mix anchor with black at calibrated percentages */
--color-primary-600: color-mix(in srgb, var(--color-primary-500) 88%, black);
--color-primary-700: color-mix(in srgb, var(--color-primary-500) 76%, black);
--color-primary-800: color-mix(in srgb, var(--color-primary-500) 60%, black);
--color-primary-900: color-mix(in srgb, var(--color-primary-500) 44%, black);
```

**Change `--color-primary-500` once, the whole scale re-derives automatically.**

### "What to extract vs derive" decision

| Design source gives              | Agent does                                          |
|----------------------------------|-----------------------------------------------------|
| Only `--color-primary-500`       | Use as anchor, derive the other 9 stops             |
| `--color-primary-500` + `--color-primary-700` | Use both, derive the other 8 stops     |
| All 10 primary stops             | Use all, skip derivation for primary                |
| Only "main blue" + "hover"       | Treat main as 500, hover as 600, derive the rest    |
| No primary at all                | Ask the user, or use a sensible default             |
| Neutral grays not in design      | Derive from primary hue, fully desaturated          |
| Success/warning/danger not in design | Use standard defaults (green/amber/red) + derive |

**Fill in the gaps, don't fabricate**: If the design gives 1 stop explicitly, use
that exact value as the anchor and derive the rest. Don't override the explicitly
given value with a derived one.

### Extension decision tree

```
Read design source
   │
   ├─ Primary color found?
   │    ├─ Yes → use as --color-primary-500, derive 50/100/200/300/400/600/700/800/900
   │    └─ No  → ask user or use --color-primary-500: #6366F1 (sensible default)
   │
   ├─ Neutral grays found?
   │    ├─ Yes → use as --color-neutral-700/800/900 anchors, derive the rest
   │    └─ No  → derive from primary hue (--color-neutral-h: <primary-h>)
   │
   ├─ Semantic colors (success/warning/danger/info) found?
   │    ├─ Yes → use as -500 anchors, derive
   │    └─ No  → use standard defaults, derive
   │
   └─ Surface/background/text colors found?
        ├─ Yes → map to --color-bg-* and --color-text-*
        └─ No  → derive from neutral scale
```

### Quality gate

Before generating the override file, run this check:

```js
// In the browser, on the preview page:
const missing = []
for (const v of [
  '--color-primary-50', '-100', '-200', '-300', '-400', '-500', '-600', '-700', '-800', '-900',
  '--color-neutral-50', '-100', '-200', '-300', '-400', '-500', '-600', '-700', '-800', '-900',
  '--color-success-50', '-500', '-700',
  '--color-warning-50', '-500', '-700',
  '--color-danger-50', '-500', '-700',
  '--color-info-50', '-500', '-700',
  '--color-bg-primary', '--color-bg-secondary', '--color-bg-elevated',
  '--color-text-primary', '--color-text-secondary', '--color-text-disabled',
  '--color-border-default', '--color-border-strong',
]) {
  const fullVar = v.startsWith('--') ? v : '--color-' + v
  if (!getComputedStyle(document.documentElement).getPropertyValue(fullVar).trim()) {
    missing.push(fullVar)
  }
}
if (missing.length) console.error('Missing tokens:', missing)
```

If any are missing, **add them to `colors.css` before generating the override**.
Gaps here = the "前边换了后边没变" failure mode.

### Why this matters

The user will say "I gave you a color, why isn't the rest of the design updated?"
because the lib has 40+ color variables and they only gave you 2. The answer is
"we derived the rest algorithmically" — not "we missed it". Make this derivation
**explicit in the README** so the user knows the magic happened.

---

## Phase 1.6: Multi-Dimension Derivation (color is not the only one)

The "single anchor + math" pattern applies to **6 categories**, not just color.
Apply the same fill-the-gaps principle to each. Full algorithm + tunable knobs in
[references/token-spec.md § 0.5](token-spec.md#05-other-derivable-variable-categories).

### Quick reference

| Category        | Anchor(s)                                         | Math                          |
|-----------------|---------------------------------------------------|-------------------------------|
| **Color**       | `--color-primary-500` / `-success-500` / etc.     | `color-mix(in srgb, …, white\|black)` |
| **Spacing**     | `--space-base: 4px` (or 8px)                      | `calc(var(--space-base) * N)` |
| **Radius**      | `--radius-base: 6px` (or 4px / 12px)              | `calc(var(--radius-base) * M)` |
| **Font size**   | `--font-size-base: 16px` + `--font-size-ratio: 1.25` | `calc(var(--font-size-base) * pow(var(--font-size-ratio), N))` |
| **Line height** | `--line-height-base: 1.5`                         | direct value or `calc(...)`     |
| **Border width**| `--border-width-base: 1px`                        | `calc(var(--border-width-base) * N)` |
| **Motion dur.** | `--motion-duration-base: 200ms`                   | `calc(var(--motion-duration-base) * K)` |

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
const anchors = [
  '--color-primary-500', '--color-success-500', '--color-warning-500', '--color-danger-500', '--color-info-500',
  '--space-base', '--radius-base',
  '--font-size-base', '--font-size-ratio',
  '--line-height-base',
  '--border-width-base',
  '--motion-duration-base',
]
const missing = anchors.filter(a => !getComputedStyle(document.documentElement).getPropertyValue(a).trim())
if (missing.length) console.error('Missing anchors:', missing)
```

If any are missing, ask the user OR pick a sensible default from the design source. **Never generate a `colors.css` / `spacing.css` / etc. with hand-picked values for derivable categories.**

---

## Phase 2: Token Extraction Checklist

When reading the design source, extract the following. **Be concrete, not vague.**
The new design is the source of truth for all visual variables; the existing component
library keeps its own interaction behavior. We are doing **visual re-skin**, not a behavior
replacement.

### Colors

| What to look for     | Where to find it                          | Variable             |
|----------------------|-------------------------------------------|----------------------|
| Primary brand color  | Buttons, links, focus rings               | `--color-primary-500` |
| Primary tints/shades | Hover/active states of primary            | `--color-primary-{50,100,200,300,400,600,700,800,900}` |
| Neutral grays        | Body text, borders, dividers, surfaces    | `--color-neutral-{50...900}` |
| Success              | Success toasts, "OK" badges, valid input  | `--color-success-{50...700}` |
| Warning              | Warning toasts, "caution" indicators      | `--color-warning-{50...700}` |
| Danger / Error       | Error states, destructive buttons         | `--color-danger-{50...700}` |
| Info                 | Info toasts, neutral notifications        | `--color-info-{50...700}` |
| Background surfaces  | Page background, card background          | `--color-bg-{primary,secondary,elevated}` |
| Text colors          | Body, headings, captions, disabled        | `--color-text-{primary,secondary,disabled,inverse}` |
| Borders              | Default, strong, subtle                   | `--color-border-{default,strong,subtle}` |

**Rule**: If Figma only gives you the 500 stop, derive the rest using
`color-mix(in srgb, var(--color-primary-500) X%, white|black)`. Do not hand-pick hex
values for 50/100/200 — use the math.

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
[references/override-patterns.md](override-patterns.md) and verify these are covered:

- [ ] **All primary light/dark stops** (`--<lib>-color-primary-light-3/5/7/8/9` and
      `--<lib>-color-primary-dark-2`). Missing any of these causes hover/active state
      color drift — the most common visual bug.
- [ ] **All semantic colors** (success / warning / danger / info), each with their
      light/dark variants if the lib uses them.
- [ ] **All radius variables** that the lib exposes (e.g. Element Plus has
      `--el-border-radius-base` / `--el-border-radius-small` / `--el-border-radius-round`).
- [ ] **All text color variables** (primary, regular, secondary, placeholder, disabled).
- [ ] **All background variables** (page, surface, container, overlay).
- [ ] **All border color variables** (default, light, lighter, extra-light, dark).
- [ ] **Font family** at minimum.
- [ ] **Dark mode variant** in `[data-theme="dark"]` block.

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
- [ ] Dark mode toggle works (button to flip `data-theme`)
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
| **Dialog / Drawer**    | default + overlay                                                  | `--<lib>-bg-color`, `-bg-overlay`     |
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
  --el-button-bg-color:           var(--color-primary-500);
  --el-button-border-color:       var(--color-primary-500);
  --el-button-hover-bg-color:     var(--color-primary-400);
  --el-button-active-bg-color:    var(--color-primary-600);
  --el-button-disabled-bg-color:  var(--color-bg-secondary);
}
```

This is **belt and suspenders** — usually not needed if Phase 3 is done correctly,
but it eliminates the most common "only 80% of components updated" complaint.

---

## Phase 5: README Checklist

The README must answer, in this order:

1. **3-line integration** (copy-paste ready)
2. **One line of HTML/JS for dark mode toggle**
3. **One line for how to change a token** ("改 `tokens/colors.css` 里的 `--color-primary-500`")
4. **A list of what was NOT overridden** (e.g. "字体、间距沿用原样,因为你选了只换颜色")
5. **Troubleshooting** (the 3 most common issues, see below)

### Top 3 Troubleshooting Issues

| Problem                                          | Cause                                          | Fix                                                |
|--------------------------------------------------|------------------------------------------------|----------------------------------------------------|
| "我改了 token 颜色但组件没变"                   | Override 引入顺序错了                          | 确保先引 lib 的 CSS,再引 `theme.css`,再引 override |
| "暗色模式不切换"                                 | 切换按钮没改 `data-theme` 属性                | `document.documentElement.setAttribute('data-theme', 'dark')` |
| "某个组件不响应"                                  | 该组件用了 hardcoded 颜色,不在 CSS 变量体系里 | 用 DevTools 看 computed style,提 issue             |

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

# 6. Dark mode block exists
grep -q "data-theme" overrides/<lib>-theme-override.css

# 7. Preview file exists
test -f preview/preview.html

# 8. README has integration snippet
grep -q "import" README.md
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
