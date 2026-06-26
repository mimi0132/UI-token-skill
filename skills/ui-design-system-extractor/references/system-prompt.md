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
| Easing            | All transitions     | `--ease-default` (cubic-bezier(0.4, 0, 0.2, 1)) |
| Easing emphasized | Hover, focus        | `--ease-emphasized` (cubic-bezier(0.2, 0, 0, 1)) |
| Duration fast     | Micro-interactions  | `--duration-fast` (150ms) |
| Duration base     | Default transitions | `--duration-base` (250ms) |
| Duration slow     | Page-level          | `--duration-slow` (400ms) |

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
# 1. Five token files exist
test -f tokens/theme.css && \
test -f tokens/colors.css && \
test -f tokens/typography.css && \
test -f tokens/spacing.css && \
test -f tokens/radius.css

# 2. Override file exists and imports tokens
test -f overrides/<lib>-theme-override.css && \
grep -q "@import" overrides/<lib>-theme-override.css

# 3. No raw hex anywhere in override (everything should be var(--*))
! grep -E "#[0-9a-fA-F]{3,8}" overrides/<lib>-theme-override.css

# 4. No raw px in override
! grep -E "[0-9]+px" overrides/<lib>-theme-override.css

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
