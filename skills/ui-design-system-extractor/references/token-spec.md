# Token Specification Reference

This file is the detailed spec for the 5 token files. Load when the agent needs to know
the exact variable name, value range, or dark-mode mapping.

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

### Color Scales (10 stops each)

For each scale, define 50 → 900:

```css
:root {
  /* Neutral */
  --color-neutral-50:  #FAFAFA;
  --color-neutral-100: #F5F5F5;
  --color-neutral-200: #E5E5E5;
  --color-neutral-300: #D4D4D4;
  --color-neutral-400: #A3A3A3;
  --color-neutral-500: #737373;
  --color-neutral-600: #525252;
  --color-neutral-700: #404040;
  --color-neutral-800: #262626;
  --color-neutral-900: #171717;

  /* Primary (override per design) */
  --color-primary-50:  #EEF2FF;
  --color-primary-100: #E0E7FF;
  --color-primary-200: #C7D2FE;
  --color-primary-300: #A5B4FC;
  --color-primary-400: #818CF8;
  --color-primary-500: #6366F1;  /* main */
  --color-primary-600: #4F46E5;
  --color-primary-700: #4338CA;
  --color-primary-800: #3730A3;
  --color-primary-900: #312E81;

  /* Semantic */
  --color-success-500: #10B981;
  --color-warning-500: #F59E0B;
  --color-danger-500:  #EF4444;
  --color-info-500:    #3B82F6;
}
```

### Semantic Colors (use these in components)

```css
:root {
  --color-bg-primary:    var(--color-neutral-50);
  --color-bg-secondary:  var(--color-neutral-100);
  --color-bg-elevated:   #FFFFFF;

  --color-text-primary:   var(--color-neutral-900);
  --color-text-secondary: var(--color-neutral-600);
  --color-text-disabled:  var(--color-neutral-400);
  --color-text-inverse:   #FFFFFF;

  --color-border-default: var(--color-neutral-200);
  --color-border-strong:  var(--color-neutral-300);
}
```

### Dark Mode (mandatory)

```css
[data-theme="dark"] {
  --color-bg-primary:    var(--color-neutral-900);
  --color-bg-secondary:  var(--color-neutral-800);
  --color-bg-elevated:   var(--color-neutral-700);

  --color-text-primary:   var(--color-neutral-50);
  --color-text-secondary: var(--color-neutral-300);
  --color-text-disabled:  var(--color-neutral-600);

  --color-border-default: var(--color-neutral-700);
  --color-border-strong:  var(--color-neutral-600);
}
```

---

## 2. `typography.css`

### Families

```css
:root {
  --font-family-heading: 'Inter', system-ui, -apple-system, sans-serif;
  --font-family-body:    'Inter', system-ui, -apple-system, sans-serif;
  --font-family-mono:    'JetBrains Mono', 'SF Mono', Menlo, monospace;
}
```

### Type Scale (8 steps)

```css
:root {
  --font-size-xs:   12px;  /* caption */
  --font-size-sm:   14px;  /* body-sm */
  --font-size-md:   16px;  /* body (default) */
  --font-size-lg:   18px;  /* body-lg */
  --font-size-xl:   20px;  /* h6 */
  --font-size-2xl:  24px;  /* h5 */
  --font-size-3xl:  30px;  /* h4 */
  --font-size-4xl:  36px;  /* h3 */
  /* Extend as needed: 5xl: 48px, 6xl: 60px */
}
```

### Weights

```css
:root {
  --font-weight-regular:   400;
  --font-weight-medium:    500;
  --font-weight-semibold:  600;
  --font-weight-bold:      700;
}
```

### Line Heights

```css
:root {
  --line-height-tight:    1.2;   /* headings */
  --line-height-normal:   1.5;   /* body */
  --line-height-relaxed:  1.75;  /* long-form */
}
```

### Letter Spacing

```css
:root {
  --letter-spacing-tight:  -0.02em;  /* display sizes */
  --letter-spacing-normal: 0;
  --letter-spacing-wide:   0.05em;   /* labels / uppercase */
}
```

### Component Mapping

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

```css
:root {
  --space-0:   0;
  --space-1:   4px;
  --space-2:   8px;
  --space-3:   12px;
  --space-4:   16px;
  --space-5:   20px;
  --space-6:   24px;
  --space-8:   32px;
  --space-10:  40px;
  --space-12:  48px;
  --space-16:  64px;
  --space-20:  80px;
  --space-24:  96px;
}
```

### Semantic Spacing

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

```css
:root {
  --radius-none: 0;
  --radius-sm:   2px;
  --radius-md:   6px;    /* default for buttons/inputs */
  --radius-lg:   12px;   /* default for cards/modals */
  --radius-xl:   16px;
  --radius-2xl:  24px;
  --radius-full: 9999px; /* pills, avatars */
}
```

---

## 5. `theme.css` (entry point)

```css
/* 1. Import tokens in this exact order */
@import './colors.css';
@import './typography.css';
@import './spacing.css';
@import './radius.css';

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

/* 3. Shadow tokens */
:root {
  --shadow-sm:      0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md:      0 4px 6px rgba(0, 0, 0, 0.07), 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-lg:      0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
  --shadow-overlay: 0 25px 50px rgba(0, 0, 0, 0.15);
}

/* 4. Motion tokens */
:root {
  --motion-duration-fast:   150ms;
  --motion-duration-normal: 200ms;
  --motion-duration-slow:   300ms;
  --motion-easing-standard: cubic-bezier(0.4, 0, 0.2, 1);
  --motion-easing-emphasis: cubic-bezier(0.2, 0, 0, 1);
}
```

---

## Validation Rules

When validating generated tokens, check:

1. **All 5 files exist** and `theme.css` imports the other 4 in order.
2. **No token is defined twice** in different files.
3. **Dark mode variant exists** for every semantic color.
4. **Scale is complete** — every scale has 50 → 900.
5. **No raw values in component files** — only `var(--*)` references.
6. **Type scale is geometric** — each step is ~1.125x–1.25x the previous.
7. **Spacing is 4-based** — every `space-*` is a multiple of 4px.
