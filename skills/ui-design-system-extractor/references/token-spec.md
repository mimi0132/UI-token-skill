# Token Specification Reference

This file is the detailed spec for the 7 token files. Load when the agent needs to know
the exact variable name, value range, or dark-mode mapping.

---

## 0. Color System Derivation Algorithm

> **Critical rule**: The design source is **sampling points**, not a complete palette.
> Figma exports typically give 1-3 stops of the primary and 1-2 swatches of each
> semantic color. The agent's job is to **derive the rest** into a complete,
> perceptually uniform scale that fully covers every lib variable.

### 0.1 The "single anchor + math" approach

Pick **one anchor color per scale** (usually the 500 stop, sometimes the 600). Derive
all 9 other stops using `color-mix()`. Calibrated percentages that produce a
perceptually uniform scale similar to Tailwind / Radix:

```css
:root {
  /* The single anchor — from the design */
  --color-primary-500: #6366F1;

  /* Derived lighter stops (mix with white) */
  --color-primary-50:  color-mix(in srgb, var(--color-primary-500)  8%, white);
  --color-primary-100: color-mix(in srgb, var(--color-primary-500) 16%, white);
  --color-primary-200: color-mix(in srgb, var(--color-primary-500) 28%, white);
  --color-primary-300: color-mix(in srgb, var(--color-primary-500) 44%, white);
  --color-primary-400: color-mix(in srgb, var(--color-primary-500) 68%, white);

  /* Derived darker stops (mix with black) */
  --color-primary-600: color-mix(in srgb, var(--color-primary-500) 88%, black);
  --color-primary-700: color-mix(in srgb, var(--color-primary-500) 76%, black);
  --color-primary-800: color-mix(in srgb, var(--color-primary-500) 60%, black);
  --color-primary-900: color-mix(in srgb, var(--color-primary-500) 44%, black);
}
```

**Change `--color-primary-500` once, all 9 stops re-derive automatically.** No
maintenance burden.

### 0.2 Mixing table (10-stop scale)

| Stop | Mix % of anchor | With    | Use case                                  |
|------|-----------------|---------|-------------------------------------------|
| 50   | 8%              | white   | Subtle background tints, hover wash       |
| 100  | 16%             | white   | Selected background, badges               |
| 200  | 28%             | white   | Disabled background, light borders        |
| 300  | 44%             | white   | Light hover, focus ring at 30% alpha     |
| 400  | 68%             | white   | Secondary buttons, links                  |
| 500  | 100%            | (anchor)| **Primary buttons, links, brand**         |
| 600  | 88%             | black   | Primary hover                             |
| 700  | 76%             | black   | Primary active                            |
| 800  | 60%             | black   | Primary text on light bg (high contrast)  |
| 900  | 44%             | black   | Dark mode primary, high-emphasis text     |

**Why these percentages**: They approximate a perceptually uniform L* curve in
Oklch, matching what Tailwind v3+ and Radix Colors produce. They look the same
across hues (not too aggressive for warm colors, not too subtle for cool).

### 0.3 What to extract vs derive

| Design source gives        | Agent does                                            |
|----------------------------|-------------------------------------------------------|
| Only primary-500           | Use as anchor, derive 50/100/200/300/400/600/700/800/900 |
| Primary-500 + primary-700  | Use both, derive the rest                              |
| All 10 primary stops       | Use all, skip derivation for primary                   |
| Only "main blue" + "hover" | Treat "main" as 500, "hover" as 600, derive the rest   |
| No primary at all          | Pick from user-provided accent OR ask                  |

**The "fill in the gaps" rule**: When the design gives SOME stops explicitly, use
those exact values and only derive the missing ones. When the design gives NONE,
derive everything from the anchor. Never mix hand-picked hex into a derived scale
unless the design explicitly specifies it.

### 0.4 Neutral scale derivation

Designs rarely specify neutrals. Derive from the primary hue, fully desaturated:

```css
:root {
  /* Anchor: a cool gray that harmonizes with the primary */
  --color-neutral-h: 220;        /* hue = same as primary's hue, or +10° for cooler */
  --color-neutral-s: 0%;         /* fully desaturated */
  --color-neutral-l: 50%;

  --color-neutral-50:  hsl(var(--color-neutral-h) var(--color-neutral-s) 97%);
  --color-neutral-100: hsl(var(--color-neutral-h) var(--color-neutral-s) 94%);
  --color-neutral-200: hsl(var(--color-neutral-h) var(--color-neutral-s) 88%);
  --color-neutral-300: hsl(var(--color-neutral-h) var(--color-neutral-s) 78%);
  --color-neutral-400: hsl(var(--color-neutral-h) var(--color-neutral-s) 65%);
  --color-neutral-500: hsl(var(--color-neutral-h) var(--color-neutral-s) 50%);
  --color-neutral-600: hsl(var(--color-neutral-h) var(--color-neutral-s) 40%);
  --color-neutral-700: hsl(var(--color-neutral-h) var(--color-neutral-s) 30%);
  --color-neutral-800: hsl(var(--color-neutral-h) var(--color-neutral-s) 16%);
  --color-neutral-900: hsl(var(--color-neutral-h) var(--color-neutral-s)  8%);
}
```

**Tuning knobs**:
- Add 5-10% saturation for a "warm" feel (e.g., `--color-neutral-s: 8%`)
- Shift hue ±20° to harmonize with the primary's hue
- If the design explicitly provides neutrals (e.g., "text gray is #1F2937"), use them as
  the 800/900 stops and derive the rest

### 0.5 Semantic color derivation

For success / warning / danger / info, the design often gives one swatch each. Use
the same anchor+math approach. If the design gives NONE, use these defaults which
work for 90% of B2B products:

```css
:root {
  --color-success-500: #10B981;   /* green */
  --color-warning-500: #F59E0B;   /* amber */
  --color-danger-500:  #EF4444;   /* red */
  --color-info-500:    #3B82F6;   /* blue (or use --color-primary-500 for "info = primary") */
}
```

If the design gives explicit semantic colors (e.g., "error red is #DC2626"), use
those as the 500 anchor and derive the rest.

### 0.6 State colors (hover / active / disabled / focus)

Don't add a new scale — consume from the primary scale. Map every state to an
existing stop:

| State              | Token                              | Element Plus variable    |
|--------------------|------------------------------------|--------------------------|
| Primary default    | `--color-primary-500`              | `--el-color-primary`     |
| Primary hover      | `--color-primary-600`              | `--el-color-primary-light-3` |
| Primary active     | `--color-primary-700`              | `--el-color-primary-dark-2`  |
| Primary disabled   | `--color-neutral-200`              | `--el-color-primary-light-9` (or override with neutral) |
| Focus ring         | `--color-primary-500` at 30% alpha | direct CSS `:focus-visible` |

This way, "change the primary" automatically changes all states — no separate
state variables to maintain.

### 0.7 The full lib coverage check

After deriving, verify every color variable the lib exposes is mapped. For
Element Plus, the complete list (~40 vars):

```
--el-color-primary, --el-color-primary-light-3, --el-color-primary-light-5,
--el-color-primary-light-7, --el-color-primary-light-8, --el-color-primary-light-9,
--el-color-primary-dark-2,

--el-color-success (same light/dark set, 7 vars),
--el-color-warning (7 vars),
--el-color-danger  (7 vars),
--el-color-info    (7 vars),

--el-text-color-primary, --el-text-color-regular, --el-text-color-secondary,
--el-text-color-placeholder, --el-text-color-disabled,

--el-border-color, --el-border-color-light, --el-border-color-lighter,
--el-border-color-extra-light, --el-border-color-dark, --el-border-color-darker,
--el-border-color-hover,

--el-fill-color, --el-fill-color-light, --el-fill-color-lighter,
--el-fill-color-extra-light, --el-fill-color-blank, --el-fill-color-dark,
--el-mask-color,
```

If your `colors.css` doesn't have a token for each of these, **add it before
generating the override file**. Gaps here = "前边换了后边没变".

---

## 0.5 Other Derivable Variable Categories

The "single anchor + math" pattern isn't unique to colors. **Six other categories**
share this property. Knowing which ones are derivable (and which are conventional)
lets the agent fill gaps the same way it does for colors.

### The "derivable" matrix

| Category        | Derivable? | Anchor(s)                    | Derivation method              |
|-----------------|------------|------------------------------|--------------------------------|
| **Color**       | ✅ Yes     | `--color-primary-500`        | `color-mix(in srgb, …, white\|black)` |
| **Spacing**     | ✅ Yes     | `--space-base` (4 or 8)      | `calc(var(--space-base) * N)`  |
| **Radius**      | ✅ Yes     | `--radius-base` (4 or 6)     | `calc(var(--radius-base) * N)` |
| **Font size**   | ✅ Yes     | `--font-size-base` + `--font-size-ratio` | `calc(var(--font-size-base) * pow(var(--font-size-ratio), N))` |
| **Line height** | ✅ Yes     | `--line-height-base`         | direct value or `calc(...)`     |
| **Border width**| ✅ Yes     | `--border-width-base` (1px)  | `calc(var(--border-width-base) * N)` |
| **Motion dur.** | ✅ Yes     | `--motion-duration-base`     | `calc(var(--motion-duration-base) * N)` |
| Font family     | ❌ No      | —                            | must be specified              |
| Font weight     | ❌ No      | —                            | industry fixed 4 values         |
| Shadow          | ❌ No      | —                            | 3-5 discrete tokens (degrees of freedom too high) |
| Grid columns    | ❌ No      | —                            | industry fixed 12/16/24         |
| Breakpoints     | ❌ No      | —                            | industry fixed                  |
| Motion easing   | ❌ No      | —                            | industry fixed 3-4 curves       |

**Rule of thumb**: if the property has a **single dimension** (size, length, time),
it can be derived. If it has **multiple dimensions** (shadow = offset x/y + blur + spread + color + opacity), use a small set of named tokens.

### Why this matters

The user will say "I gave you 16px body text and 24px heading, why aren't all my
text sizes set?" — because the design gave 2 of 8 and the lib needs all 8. Same
answer as color: "we derived the rest from a base + ratio." Make it explicit in
the README so the user knows the magic happened for **every category**, not just
color.

### Tunable knobs

| Anchor             | Common values          | When to pick                           |
|--------------------|------------------------|----------------------------------------|
| `--space-base`     | 4px (compact) / 8px (default) / 4px (Material) | Density of the design source |
| `--radius-base`    | 2px (sharp) / 4px (default) / 6px (rounded) / 12px (iOS-style) | Visual personality |
| `--font-size-base` | 14px (compact) / 16px (default) / 18px (spacious) | Body text size in the design |
| `--font-size-ratio`| 1.125 (subtle) / 1.25 (Material) / 1.333 (strong) / 1.5 (dramatic) | Heading contrast strength |
| `--border-width-base` | 1px (always)        | Almost never change                    |
| `--motion-duration-base` | 150ms (snappy) / 200ms (default) / 300ms (leisurely) | Brand tempo |

---

## Naming Convention

```
--{category}-{role}-{scale}
--{category}-{role}-{variant}
```

| Category   | Examples                                              |
|------------|-------------------------------------------------------|
| `color`    | `--color-primary-500`, `--color-text-secondary`      |
| `font`     | `--font-family-heading`, `--font-size-md`, `--font-weight-semibold` |
| `space`    | `--space-4`, `--space-component-padding-md`          |
| `radius`   | `--radius-sm`, `--radius-md`                         |
| `shadow`   | `--shadow-sm`, `--shadow-md`, `--shadow-overlay`     |
| `motion`   | `--motion-duration-fast`, `--motion-easing-standard`  |

Rules:
- **kebab-case** only
- **Semantic** names, not visual (`--color-primary-500` not `--blue-500`)
- **No magic numbers** in component files — only token references

---

## 1. `colors.css`

> **See [§ 0 Color System Derivation Algorithm](#0-color-system-derivation-algorithm) first.** The 10-stop scales below are derived
> from a single anchor using `color-mix()`. Don't hand-pick hex for the 9 derived stops.

### 1.1 Primary scale (anchor from design, rest derived)

```css
:root {
  /* 🔑 SINGLE ANCHOR — change this one value to re-derive the whole scale */
  --color-primary-500: #6366F1;

  /* Lighter stops: mix with white */
  --color-primary-50:  color-mix(in srgb, var(--color-primary-500)  8%, white);
  --color-primary-100: color-mix(in srgb, var(--color-primary-500) 16%, white);
  --color-primary-200: color-mix(in srgb, var(--color-primary-500) 28%, white);
  --color-primary-300: color-mix(in srgb, var(--color-primary-500) 44%, white);
  --color-primary-400: color-mix(in srgb, var(--color-primary-500) 68%, white);

  /* Darker stops: mix with black */
  --color-primary-600: color-mix(in srgb, var(--color-primary-500) 88%, black);
  --color-primary-700: color-mix(in srgb, var(--color-primary-500) 76%, black);
  --color-primary-800: color-mix(in srgb, var(--color-primary-500) 60%, black);
  --color-primary-900: color-mix(in srgb, var(--color-primary-500) 44%, black);
}
```

### 1.2 Neutral scale (derived from primary hue, desaturated)

```css
:root {
  --color-neutral-h: 220;   /* tune: same as primary's hue, or +10° cooler */
  --color-neutral-s: 0%;    /* tune: 5-8% for warmer feel */

  --color-neutral-50:  hsl(var(--color-neutral-h) var(--color-neutral-s) 97%);
  --color-neutral-100: hsl(var(--color-neutral-h) var(--color-neutral-s) 94%);
  --color-neutral-200: hsl(var(--color-neutral-h) var(--color-neutral-s) 88%);
  --color-neutral-300: hsl(var(--color-neutral-h) var(--color-neutral-s) 78%);
  --color-neutral-400: hsl(var(--color-neutral-h) var(--color-neutral-s) 65%);
  --color-neutral-500: hsl(var(--color-neutral-h) var(--color-neutral-s) 50%);
  --color-neutral-600: hsl(var(--color-neutral-h) var(--color-neutral-s) 40%);
  --color-neutral-700: hsl(var(--color-neutral-h) var(--color-neutral-s) 30%);
  --color-neutral-800: hsl(var(--color-neutral-h) var(--color-neutral-s) 16%);
  --color-neutral-900: hsl(var(--color-neutral-h) var(--color-neutral-s)  8%);
}
```

### 1.3 Semantic scales (one anchor each, rest derived)

```css
:root {
  /* Success — green */
  --color-success-500: #10B981;
  --color-success-50:  color-mix(in srgb, var(--color-success-500)  8%, white);
  --color-success-100: color-mix(in srgb, var(--color-success-500) 16%, white);
  --color-success-200: color-mix(in srgb, var(--color-success-500) 28%, white);
  --color-success-300: color-mix(in srgb, var(--color-success-500) 44%, white);
  --color-success-400: color-mix(in srgb, var(--color-success-500) 68%, white);
  --color-success-600: color-mix(in srgb, var(--color-success-500) 88%, black);
  --color-success-700: color-mix(in srgb, var(--color-success-500) 76%, black);
  --color-success-800: color-mix(in srgb, var(--color-success-500) 60%, black);
  --color-success-900: color-mix(in srgb, var(--color-success-500) 44%, black);

  /* Warning — amber */
  --color-warning-500: #F59E0B;
  --color-warning-50:  color-mix(in srgb, var(--color-warning-500)  8%, white);
  --color-warning-100: color-mix(in srgb, var(--color-warning-500) 16%, white);
  /* ... (apply same formula as success) */

  /* Danger — red */
  --color-danger-500: #EF4444;
  --color-danger-50:  color-mix(in srgb, var(--color-danger-500)  8%, white);
  /* ... */

  /* Info — usually = primary, or its own blue */
  --color-info-500: var(--color-primary-500);   /* common: "info" = primary */
  /* ... */
}
```

### 1.4 Semantic colors (consume from scales)

```css
:root {
  --color-bg-primary:    var(--color-neutral-50);
  --color-bg-secondary:  var(--color-neutral-100);
  --color-bg-elevated:   white;

  /* 5 级文字对比阶梯 (从最强到最弱):
     primary > regular > secondary > placeholder > disabled
     用于覆盖 Element Plus 的 --el-text-color-{primary,regular,secondary,placeholder,disabled} */
  --color-text-primary:     var(--color-neutral-900);
  --color-text-regular:     var(--color-neutral-800);
  --color-text-secondary:   var(--color-neutral-600);
  --color-text-placeholder: var(--color-neutral-500);
  --color-text-disabled:    var(--color-neutral-400);
  --color-text-inverse:     white;

  --color-border-default: var(--color-neutral-200);
  --color-border-strong:  var(--color-neutral-300);
  --color-border-focus:   var(--color-primary-500);
}
```

### 1.5 Dark Mode (mandatory)

```css
[data-theme="dark"] {
  --color-bg-primary:    var(--color-neutral-900);
  --color-bg-secondary:  var(--color-neutral-800);
  --color-bg-elevated:   var(--color-neutral-700);

  /* 暗色下保持 5 级对比阶梯, 但颜色明度反转:
     primary(最亮) > regular > secondary > placeholder > disabled(最暗) */
  --color-text-primary:     var(--color-neutral-50);
  --color-text-regular:     var(--color-neutral-300);
  --color-text-secondary:   var(--color-neutral-400);
  --color-text-placeholder: var(--color-neutral-500);
  --color-text-disabled:    var(--color-neutral-600);

  --color-border-default: var(--color-neutral-700);
  --color-border-strong:  var(--color-neutral-600);
}
```

---

## 2. `typography.css`

### Families (cannot be derived — must be specified)

```css
:root {
  --font-family-heading: 'Inter', system-ui, -apple-system, sans-serif;
  --font-family-body:    'Inter', system-ui, -apple-system, sans-serif;
  --font-family-mono:    'JetBrains Mono', 'SF Mono', Menlo, monospace;
}
```

### Type Scale — derived from base + ratio (geometric progression)

```css
:root {
  /* 🔑 ANCHORS — pick base from design, pick ratio for personality */
  --font-size-base:  16px;     /* body text size from design */
  --font-size-ratio: 1.25;     /* major third (Material); try 1.125/1.333/1.5 */

  --font-size-xs:   calc(var(--font-size-base) / var(--font-size-ratio) / var(--font-size-ratio));   /* 10.24px */
  --font-size-sm:   calc(var(--font-size-base) / var(--font-size-ratio));                            /* 12.8px  */
  --font-size-md:   var(--font-size-base);                                                           /* 16px    */
  --font-size-lg:   calc(var(--font-size-base) * var(--font-size-ratio));                            /* 20px    */
  --font-size-xl:   calc(var(--font-size-base) * var(--font-size-ratio) * var(--font-size-ratio));   /* 25px    */
  --font-size-2xl:  calc(var(--font-size-base) * var(--font-size-ratio) * var(--font-size-ratio) * var(--font-size-ratio)); /* 31.25px */
  --font-size-3xl:  calc(var(--font-size-base) * pow(var(--font-size-ratio), 4));                     /* 39.06px */
  --font-size-4xl:  calc(var(--font-size-base) * pow(var(--font-size-ratio), 5));                     /* 48.83px */
  --font-size-5xl:  calc(var(--font-size-base) * pow(var(--font-size-ratio), 6));                     /* 61.04px */
  --font-size-6xl:  calc(var(--font-size-base) * pow(var(--font-size-ratio), 7));                     /* 76.3px  */
}
```

**When the design gives explicit sizes** (e.g., "h1 is 32px, body is 14px"), use
those as the source of truth and **overwrite** the derived values — don't force
the math to match.

### Weights (cannot be derived — industry fixed 4 values)

```css
:root {
  --font-weight-regular:   400;
  --font-weight-medium:    500;
  --font-weight-semibold:  600;
  --font-weight-bold:      700;
}
```

### Line Heights (derive from base, or specify 3 fixed values)

```css
:root {
  --line-height-base: 1.5;        /* 🔑 ANCHOR */

  --line-height-tight:   1.2;                       /* headings */
  --line-height-normal:  var(--line-height-base);   /* body */
  --line-height-relaxed: calc(var(--line-height-base) + 0.25);  /* 1.75 */
}
```

### Letter Spacing (3 fixed values, tied to size)

```css
:root {
  --letter-spacing-tight:  -0.02em;  /* display sizes (5xl+) */
  --letter-spacing-normal: 0;
  --letter-spacing-wide:   0.05em;   /* labels / uppercase */
}
```

### Component Mapping (consume primitives)

```css
:root {
  --text-heading-1: var(--font-size-4xl) / var(--line-height-tight) var(--font-family-heading);
  --text-heading-2: var(--font-size-3xl) / var(--line-height-tight) var(--font-family-heading);
  --text-heading-3: var(--font-size-2xl) / var(--line-height-tight) var(--font-family-heading);
  --text-body-lg:   var(--font-size-lg)   / var(--line-height-normal) var(--font-family-body);
  --text-body:      var(--font-size-md)   / var(--line-height-normal) var(--font-family-body);
  --text-body-sm:   var(--font-size-sm)   / var(--line-height-normal) var(--font-family-body);
  --text-caption:   var(--font-size-xs)   / var(--line-height-normal) var(--font-family-body);
}
```

---

## 3. `spacing.css`

### Spacing Scale — derived from a single base unit (linear)

```css
:root {
  /* 🔑 ANCHOR — pick 4 (compact/Material) or 8 (default/Bootstrap) */
  --space-base: 4px;

  --space-0:   0;
  --space-1:   calc(var(--space-base) * 1);    /*  4px */
  --space-2:   calc(var(--space-base) * 2);    /*  8px */
  --space-3:   calc(var(--space-base) * 3);    /* 12px */
  --space-4:   calc(var(--space-base) * 4);    /* 16px */
  --space-5:   calc(var(--space-base) * 5);    /* 20px */
  --space-6:   calc(var(--space-base) * 6);    /* 24px */
  --space-8:   calc(var(--space-base) * 8);    /* 32px */
  --space-10:  calc(var(--space-base) * 10);   /* 40px */
  --space-12:  calc(var(--space-base) * 12);   /* 48px */
  --space-16:  calc(var(--space-base) * 16);   /* 64px */
  --space-20:  calc(var(--space-base) * 20);   /* 80px */
  --space-24:  calc(var(--space-base) * 24);   /* 96px */
}
```

**Change `--space-base: 4px` → `--space-base: 8px` and the whole scale doubles.**
This makes "tighter" or "looser" spacing a one-line change.

**Common base choices**:
- `4px` → Material Design, compact, dense data tables
- `8px` → Bootstrap, Tailwind default, balanced
- `4px` (with 1.5x intermediate steps) → Modern iOS-style density

### Semantic Spacing (consume from scale)

```css
:root {
  --space-component-padding-sm: var(--space-2);
  --space-component-padding-md: var(--space-4);
  --space-component-padding-lg: var(--space-6);

  --space-section-gap-sm:       var(--space-6);
  --space-section-gap-md:       var(--space-12);
  --space-section-gap-lg:       var(--space-16);
}
```

---

## 4. `radius.css`

### Radius Scale — derived from a single base unit (multiplicative)

```css
:root {
  /* 🔑 ANCHOR — pick the personality of the design */
  --radius-base: 6px;   /* try 2 (sharp) / 4 (default) / 6 (rounded) / 12 (iOS-style) */

  --radius-none: 0;
  --radius-sm:   var(--radius-base);                     /*  6px */
  --radius-md:   calc(var(--radius-base) * 1.5);         /*  9px */
  --radius-lg:   calc(var(--radius-base) * 2);           /* 12px */
  --radius-xl:   calc(var(--radius-base) * 3);           /* 18px */
  --radius-2xl:  calc(var(--radius-base) * 4);           /* 24px */
  --radius-full: 9999px;                                 /* independent — pills / avatars */
}
```

**Change `--radius-base: 6px` → `--radius-base: 12px` and the whole scale gets
"softer".** This is how the "iOS-style" / "Material" / "shadcn" personality is
captured in one variable.

**Personality presets**:
- `2px` → Material Design 2 (sharp, professional)
- `4px` → shadcn / Linear (subtle rounding)
- `6px` → Tailwind default (balanced)
- `12px` → iOS / Stripe / Modern SaaS (soft)
- `16px` → Extra-soft / Notion-style (pillowy)

---

## 5. `border.css`

### Border Width — derived from base (almost always 1px)

```css
:root {
  --border-width-base: 1px;   /* 🔑 ANCHOR — almost always 1 */

  --border-width-none:    0;
  --border-width-default: var(--border-width-base);                /* 1px */
  --border-width-strong:  calc(var(--border-width-base) * 2);      /* 2px */
  --border-width-heavy:   calc(var(--border-width-base) * 3);      /* 3px */
}

:root {
  --border-style-default: solid;    /* most use cases */
  --border-style-subtle:  dashed;   /* less common */
}
```

---

## 6. `motion.css`

### Duration — derived from base (single anchor + ratio)

```css
:root {
  /* 🔑 ANCHOR — pick the brand tempo */
  --motion-duration-base: 200ms;   /* try 150 (snappy) / 200 (default) / 300 (leisurely) */

  --motion-duration-instant: calc(var(--motion-duration-base) * 0.5);   /* 100ms */
  --motion-duration-fast:    calc(var(--motion-duration-base) * 0.75);  /* 150ms */
  --motion-duration-normal:  var(--motion-duration-base);               /* 200ms */
  --motion-duration-slow:    calc(var(--motion-duration-base) * 1.5);    /* 300ms */
  --motion-duration-slower:  calc(var(--motion-duration-base) * 2.5);    /* 500ms */
}
```

### Easing (cannot be derived — 3 industry-standard curves)

```css
:root {
  --motion-easing-standard:  cubic-bezier(0.4, 0, 0.2, 1);   /* Material standard — most uses */
  --motion-easing-decelerate: cubic-bezier(0, 0, 0.2, 1);     /* entering — ease-out */
  --motion-easing-accelerate: cubic-bezier(0.4, 0, 1, 1);     /* leaving — ease-in */
  --motion-easing-emphasis:  cubic-bezier(0.2, 0, 0, 1);      /* strong entrance / exit */
  --motion-easing-spring:    cubic-bezier(0.34, 1.56, 0.64, 1); /* iOS-style overshoot */
}
```

---

## 7. `theme.css` (entry point)

```css
/* 1. Import tokens in this exact order */
@import './colors.css';
@import './typography.css';
@import './spacing.css';
@import './radius.css';
@import './border.css';
@import './motion.css';

/* 2. Component tokens (consume primitives) */
:root {
  --button-padding-y:  var(--space-2);
  --button-padding-x:  var(--space-4);
  --button-radius:     var(--radius-md);
  --button-bg:         var(--color-primary-500);
  --button-text:       var(--color-text-inverse);

  --input-padding-y:   var(--space-2);
  --input-padding-x:   var(--space-3);
  --input-radius:      var(--radius-md);
  --input-border:      var(--color-border-default);
  --input-bg:          var(--color-bg-elevated);

  --card-padding:      var(--space-6);
  --card-radius:       var(--radius-lg);
  --card-shadow:       var(--shadow-sm);
  --card-bg:           var(--color-bg-elevated);
  --card-border:       var(--color-border-default);
}

/* 3. Shadow tokens (cannot be cleanly derived — use a small fixed set) */
:root {
  --shadow-sm:      0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md:      0 4px 6px rgba(0, 0, 0, 0.07), 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-lg:      0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
  --shadow-overlay: 0 25px 50px rgba(0, 0, 0, 0.15);
}
```

---

## Validation Rules

When validating generated tokens, check:

1. **All 7 files exist** and `theme.css` imports the other 6 in order.
2. **No token is defined twice** in different files.
3. **Dark mode variant exists** for every semantic color.
4. **Scale is complete** — every color scale has 50 → 900.
5. **No raw values in component files** — only `var(--*)` references.
6. **Type scale is geometric** — each step is `--font-size-ratio` from the previous.
7. **Spacing is base-derived** — every `space-*` is `calc(var(--space-base) * N)`.
8. **Radius is base-derived** — every `radius-*` is `calc(var(--radius-base) * M)` (except `--radius-full`).
9. **Anchors are explicit** — `--color-primary-500`, `--space-base`, `--radius-base`, `--font-size-base`/`--font-size-ratio`, `--border-width-base`, `--motion-duration-base` are all defined in `:root`.
10. **Derivable categories use formulas, not hand-picked values** — re-deriving by changing an anchor should affect the right stops.
