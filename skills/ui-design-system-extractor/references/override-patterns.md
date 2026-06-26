# Override Patterns Cookbook

This file is the **core reference** for Step 2 of the workflow. It gives the complete
CSS variable mapping for each major UI library. When the user picks a library, the
agent should generate the override file by copying the relevant section verbatim and
substituting our design tokens.

**Rule**: Always copy the entire section, not just the variables that "seem" relevant.
The libs have ~50–200 CSS variables for a reason — every one affects some visual state.

---

## Index

1. [Element Plus (Vue 3)](#element-plus-vue-3)
2. [Ant Design (React)](#ant-design-react)
3. [Ant Design Vue (Vue 3)](#ant-design-vue-vue-3)
4. [Naive UI (Vue 3)](#naive-ui-vue-3)
5. [shadcn/ui (React + Tailwind)](#shadcnui-react--tailwind)
6. [MUI (React)](#mui-react)
7. [Chakra UI (React)](#chakra-ui-react)
8. [Vuetify (Vue 3)](#vuetify-vue-3)

---

## Element Plus (Vue 3)

### Required import

```ts
// main.ts
import 'element-plus/dist/index.css'
import './ui-theme/tokens/theme.css'
import './ui-theme/overrides/element-plus-theme-override.css'
```

### Override file

```css
/* overrides/element-plus-theme-override.css */
@import '../tokens/theme.css';

:root {
  /* ===== Primary palette (ALL stops required) ===== */
  --el-color-primary:           var(--color-primary-500);
  --el-color-primary-light-3:   var(--color-primary-300);
  --el-color-primary-light-5:   var(--color-primary-100);
  --el-color-primary-light-7:   var(--color-primary-50);
  --el-color-primary-light-8:   var(--color-primary-50);
  --el-color-primary-light-9:   var(--color-primary-50);
  --el-color-primary-dark-2:    var(--color-primary-700);

  /* ===== Semantic palette ===== */
  --el-color-success:           var(--color-success-500);
  --el-color-success-light-3:   var(--color-success-300);
  --el-color-success-light-5:   var(--color-success-100);
  --el-color-success-light-7:   var(--color-success-50);
  --el-color-success-light-8:   var(--color-success-50);
  --el-color-success-light-9:   var(--color-success-50);
  --el-color-success-dark-2:    var(--color-success-700);

  --el-color-warning:           var(--color-warning-500);
  --el-color-warning-light-3:   var(--color-warning-300);
  --el-color-warning-light-5:   var(--color-warning-100);
  --el-color-warning-light-7:   var(--color-warning-50);
  --el-color-warning-light-8:   var(--color-warning-50);
  --el-color-warning-light-9:   var(--color-warning-50);
  --el-color-warning-dark-2:    var(--color-warning-700);

  --el-color-danger:            var(--color-danger-500);
  --el-color-danger-light-3:    var(--color-danger-300);
  --el-color-danger-light-5:    var(--color-danger-100);
  --el-color-danger-light-7:    var(--color-danger-50);
  --el-color-danger-light-8:    var(--color-danger-50);
  --el-color-danger-light-9:    var(--color-danger-50);
  --el-color-danger-dark-2:     var(--color-danger-700);

  --el-color-error:             var(--color-danger-500);
  --el-color-error-light-3:     var(--color-danger-300);
  --el-color-error-light-5:     var(--color-danger-100);
  --el-color-error-light-7:     var(--color-danger-50);
  --el-color-error-light-8:     var(--color-danger-50);
  --el-color-error-light-9:     var(--color-danger-50);
  --el-color-error-dark-2:      var(--color-danger-700);

  --el-color-info:              var(--color-info-500);
  --el-color-info-light-3:      var(--color-info-300);
  --el-color-info-light-5:      var(--color-info-100);
  --el-color-info-light-7:      var(--color-info-50);
  --el-color-info-light-8:      var(--color-info-50);
  --el-color-info-light-9:      var(--color-info-50);
  --el-color-info-dark-2:       var(--color-info-700);

  /* ===== Border radius ===== */
  --el-border-radius-base:      var(--radius-md);
  --el-border-radius-small:     var(--radius-sm);
  --el-border-radius-round:     var(--radius-full);
  --el-border-radius-circle:    var(--radius-full);

  /* ===== Border colors ===== */
  --el-border-color:                  var(--color-border-default);
  --el-border-color-light:            var(--color-border-subtle);
  --el-border-color-lighter:          var(--color-border-subtle);
  --el-border-color-extra-light:      var(--color-border-subtle);
  --el-border-color-dark:             var(--color-border-strong);
  --el-border-color-darker:           var(--color-border-strong);
  --el-border-color-hover:            var(--color-primary-500);

  /* ===== Fill (background) ===== */
  --el-fill-color:               var(--color-bg-elevated);
  --el-fill-color-light:         var(--color-bg-secondary);
  --el-fill-color-lighter:       var(--color-bg-secondary);
  --el-fill-color-extra-light:   var(--color-bg-primary);
  --el-fill-color-dark:          var(--color-bg-secondary);
  --el-fill-color-darker:        var(--color-bg-primary);
  --el-fill-color-blank:         var(--color-bg-elevated);

  /* ===== Background ===== */
  --el-bg-color:                 var(--color-bg-primary);
  --el-bg-color-page:            var(--color-bg-secondary);
  --el-bg-overlay:               var(--color-bg-elevated);

  /* ===== Text ===== */
  --el-text-color-primary:       var(--color-text-primary);
  --el-text-color-regular:       var(--color-text-primary);
  --el-text-color-secondary:     var(--color-text-secondary);
  --el-text-color-placeholder:   var(--color-text-disabled);
  --el-text-color-disabled:      var(--color-text-disabled);

  /* ===== Typography ===== */
  --el-font-family:              var(--font-family-body);
  --el-font-size-base:           var(--font-size-md);

  /* ===== Shadow ===== */
  --el-box-shadow:               var(--shadow-sm);
  --el-box-shadow-light:         var(--shadow-sm);
  --el-box-shadow-lighter:       var(--shadow-sm);
  --el-box-shadow-dark:          var(--shadow-md);
  --el-disabled-bg-color:        var(--color-bg-secondary);
  --el-disabled-text-color:      var(--color-text-disabled);
  --el-disabled-border-color:    var(--color-border-subtle);
}

[data-theme="dark"] {
  --el-bg-color:                 var(--color-bg-primary);
  --el-bg-color-page:            var(--color-bg-primary);
  --el-bg-overlay:               var(--color-bg-elevated);
  --el-text-color-primary:       var(--color-text-primary);
  --el-text-color-regular:       var(--color-text-primary);
  --el-text-color-secondary:     var(--color-text-secondary);
  --el-fill-color-blank:         var(--color-bg-elevated);
  --el-border-color:             var(--color-border-default);
  --el-disabled-bg-color:        var(--color-bg-elevated);
}
```

---

## Ant Design (React)

### Required import

```tsx
// main.tsx
import 'antd/dist/reset.css'
import './ui-theme/tokens/theme.css'
import './ui-theme/overrides/antd-theme-override.css'
```

### Override file

```css
/* overrides/antd-theme-override.css */
@import '../tokens/theme.css';

:root {
  /* ===== Primary palette ===== */
  --ant-color-primary:           var(--color-primary-500);
  --ant-color-primary-hover:     var(--color-primary-400);
  --ant-color-primary-active:    var(--color-primary-600);
  --ant-color-primary-bg:        var(--color-primary-50);
  --ant-color-primary-bg-hover:  var(--color-primary-100);
  --ant-color-primary-border:    var(--color-primary-300);
  --ant-color-primary-border-hover: var(--color-primary-400);
  --ant-color-primary-text:      var(--color-primary-700);
  --ant-color-primary-text-hover: var(--color-primary-600);
  --ant-color-primary-text-active: var(--color-primary-700);

  /* ===== Semantic ===== */
  --ant-color-success:           var(--color-success-500);
  --ant-color-success-hover:     var(--color-success-400);
  --ant-color-success-active:    var(--color-success-600);
  --ant-color-success-bg:        var(--color-success-50);
  --ant-color-success-bg-hover:  var(--color-success-100);
  --ant-color-success-border:    var(--color-success-300);
  --ant-color-success-text:      var(--color-success-700);

  --ant-color-warning:           var(--color-warning-500);
  --ant-color-warning-hover:     var(--color-warning-400);
  --ant-color-warning-active:    var(--color-warning-600);
  --ant-color-warning-bg:        var(--color-warning-50);
  --ant-color-warning-bg-hover:  var(--color-warning-100);
  --ant-color-warning-border:    var(--color-warning-300);
  --ant-color-warning-text:      var(--color-warning-700);

  --ant-color-error:             var(--color-danger-500);
  --ant-color-error-hover:       var(--color-danger-400);
  --ant-color-error-active:      var(--color-danger-600);
  --ant-color-error-bg:          var(--color-danger-50);
  --ant-color-error-bg-hover:    var(--color-danger-100);
  --ant-color-error-border:      var(--color-danger-300);
  --ant-color-error-text:        var(--color-danger-700);

  --ant-color-info:              var(--color-info-500);
  --ant-color-info-hover:        var(--color-info-400);
  --ant-color-info-active:       var(--color-info-600);
  --ant-color-info-bg:           var(--color-info-50);
  --ant-color-info-bg-hover:     var(--color-info-100);
  --ant-color-info-border:       var(--color-info-300);
  --ant-color-info-text:         var(--color-info-700);

  /* ===== Text ===== */
  --ant-color-text:              var(--color-text-primary);
  --ant-color-text-secondary:    var(--color-text-secondary);
  --ant-color-text-tertiary:     var(--color-text-secondary);
  --ant-color-text-quaternary:   var(--color-text-disabled);
  --ant-color-text-placeholder:  var(--color-text-disabled);
  --ant-color-text-disabled:     var(--color-text-disabled);
  --ant-color-text-heading:      var(--color-text-primary);
  --ant-color-text-description:  var(--color-text-secondary);
  --ant-color-text-light-solid:  var(--color-text-inverse);

  /* ===== Background ===== */
  --ant-color-bg-base:           var(--color-bg-primary);
  --ant-color-bg-container:      var(--color-bg-elevated);
  --ant-color-bg-elevated:       var(--color-bg-elevated);
  --ant-color-bg-layout:         var(--color-bg-secondary);
  --ant-color-bg-spotlight:      var(--color-bg-elevated);
  --ant-color-bg-mask:           var(--color-bg-primary);

  /* ===== Border ===== */
  --ant-color-border:            var(--color-border-default);
  --ant-color-border-secondary:  var(--color-border-subtle);

  /* ===== Radius ===== */
  --ant-border-radius:           var(--radius-md);
  --ant-border-radius-sm:        var(--radius-sm);
  --ant-border-radius-lg:        var(--radius-lg);
  --ant-border-radius-xs:        var(--radius-sm);
  --ant-border-radius-outer:     var(--radius-md);

  /* ===== Typography ===== */
  --ant-font-family:             var(--font-family-body);
  --ant-font-size:               var(--font-size-md);
  --ant-font-size-sm:            var(--font-size-sm);
  --ant-font-size-lg:            var(--font-size-lg);
  --ant-line-height:             var(--line-height-normal);

  /* ===== Shadow ===== */
  --ant-box-shadow:              var(--shadow-sm);
  --ant-box-shadow-secondary:    var(--shadow-md);
  --ant-box-shadow-popover:      var(--shadow-lg);
  --ant-box-shadow-card:         var(--shadow-sm);
  --ant-box-shadow-drawer:       var(--shadow-overlay);

  /* ===== Control ===== */
  --ant-control-height:          32px;
  --ant-control-height-sm:       24px;
  --ant-control-height-lg:       40px;
}

[data-theme="dark"] {
  --ant-color-bg-base:           var(--color-bg-primary);
  --ant-color-bg-container:      var(--color-bg-elevated);
  --ant-color-bg-layout:         var(--color-bg-primary);
  --ant-color-text:              var(--color-text-primary);
  --ant-color-text-secondary:    var(--color-text-secondary);
  --ant-color-border:            var(--color-border-default);
}
```

> **Note**: Ant Design 5 also accepts a runtime `theme` prop on `ConfigProvider` for
> JS-level theming. CSS variables are the lighter integration path; use them when
> possible. If user needs JS theming (e.g. dynamic primary color), see Ant Design docs
> for the `theme.token` API.

---

## Ant Design Vue (Vue 3)

```css
/* overrides/ant-design-vue-theme-override.css */
@import '../tokens/theme.css';

:root {
  --ant-primary-color:           var(--color-primary-500);
  --ant-primary-color-hover:     var(--color-primary-400);
  --ant-primary-color-active:    var(--color-primary-600);
  --ant-primary-1:               var(--color-primary-50);
  --ant-primary-2:               var(--color-primary-100);
  --ant-primary-3:               var(--color-primary-200);
  --ant-primary-4:               var(--color-primary-300);
  --ant-primary-5:               var(--color-primary-400);
  --ant-primary-6:               var(--color-primary-500);
  --ant-primary-7:               var(--color-primary-600);
  --ant-success-color:           var(--color-success-500);
  --ant-warning-color:           var(--color-warning-500);
  --ant-error-color:             var(--color-danger-500);
  --ant-info-color:              var(--color-info-500);
  --ant-text-color:              var(--color-text-primary);
  --ant-text-color-secondary:    var(--color-text-secondary);
  --ant-text-color-disabled:     var(--color-text-disabled);
  --ant-bg-color:                var(--color-bg-elevated);
  --ant-bg-color-hover:          var(--color-bg-secondary);
  --ant-bg-color-active:         var(--color-bg-secondary);
  --ant-border-color:            var(--color-border-default);
  --ant-border-radius-base:      var(--radius-md);
  --ant-font-family:             var(--font-family-body);
}
```

---

## Naive UI (Vue 3)

Naive UI mostly uses JS theme overrides (`themeOverrides` prop on `NConfigProvider`).
The CSS variable approach only covers a subset. **Recommend the JS path for Naive UI.**

### Option A: JS theme overrides (recommended)

```ts
// naive-theme.ts
export const themeOverrides = {
  common: {
    primaryColor:           'var(--color-primary-500)',
    primaryColorHover:      'var(--color-primary-400)',
    primaryColorPressed:    'var(--color-primary-600)',
    primaryColorSuppl:      'var(--color-primary-600)',

    infoColor:              'var(--color-info-500)',
    infoColorHover:         'var(--color-info-400)',
    infoColorPressed:       'var(--color-info-600)',

    successColor:           'var(--color-success-500)',
    successColorHover:      'var(--color-success-400)',
    successColorPressed:    'var(--color-success-600)',

    warningColor:           'var(--color-warning-500)',
    warningColorHover:      'var(--color-warning-400)',
    warningColorPressed:    'var(--color-warning-600)',

    errorColor:             'var(--color-danger-500)',
    errorColorHover:        'var(--color-danger-400)',
    errorColorPressed:      'var(--color-danger-600)',

    textColorBase:          'var(--color-text-primary)',
    textColor1:             'var(--color-text-primary)',
    textColor2:             'var(--color-text-primary)',
    textColor3:             'var(--color-text-secondary)',
    placeholderColor:       'var(--color-text-disabled)',
    placeholderColorHover:  'var(--color-text-disabled)',
    placeholderColorPressed:'var(--color-text-disabled)',
    iconColor:              'var(--color-text-secondary)',
    iconColorHover:         'var(--color-text-primary)',
    iconColorPressed:       'var(--color-text-primary)',

    bodyColor:              'var(--color-bg-primary)',
    cardColor:              'var(--color-bg-elevated)',
    modalColor:             'var(--color-bg-elevated)',
    popoverColor:           'var(--color-bg-elevated)',
    tableColor:             'var(--color-bg-elevated)',
    inputColor:             'var(--color-bg-elevated)',
    actionColor:            'var(--color-bg-secondary)',
    tagColor:               'var(--color-neutral-100)',

    borderColor:            'var(--color-border-default)',
    dividerColor:           'var(--color-border-subtle)',

    borderRadius:           'var(--radius-md)',
    borderRadiusSmall:      'var(--radius-sm)',

    fontFamily:             'var(--font-family-body)',
    fontFamilyMono:         'var(--font-family-mono)',

    heightSmall:            '24px',
    heightMedium:           '32px',
    heightLarge:            '40px',
  },
}
```

```vue
<!-- App.vue -->
<script setup lang="ts">
import { NConfigProvider } from 'naive-ui'
import { themeOverrides } from './naive-theme'
import './ui-theme/tokens/theme.css'
</script>

<template>
  <NConfigProvider :theme-overrides="themeOverrides">
    <router-view />
  </NConfigProvider>
</template>
```

### Option B: CSS variables (limited coverage)

```css
/* overrides/naive-ui-theme-override.css */
@import '../tokens/theme.css';

:root {
  --primary-color:           var(--color-primary-500);
  --primary-color-hover:     var(--color-primary-400);
  --primary-color-pressed:   var(--color-primary-600);
  --info-color:              var(--color-info-500);
  --success-color:           var(--color-success-500);
  --warning-color:           var(--color-warning-500);
  --error-color:             var(--color-danger-500);
  --text-color-base:         var(--color-text-primary);
  --text-color-1:            var(--color-text-primary);
  --text-color-2:            var(--color-text-primary);
  --text-color-3:            var(--color-text-secondary);
  --placeholder-color:       var(--color-text-disabled);
  --border-color:            var(--color-border-default);
  --border-radius:           var(--radius-md);
  --border-radius-small:     var(--radius-sm);
}
```

---

## shadcn/ui (React + Tailwind)

shadcn uses CSS variables in HSL format (e.g. `222.2 47.4% 11.2%`). We need to convert.

### Required import

```tsx
// main.tsx
import './ui-theme/tokens/theme.css'
import './ui-theme/overrides/shadcn-theme-override.css'
import './globals.css'  // shadcn's own CSS
```

### Override file

```css
/* overrides/shadcn-theme-override.css */
@import '../tokens/theme.css';

:root {
  /* shadcn requires HSL components, not full color values.
     Convert: hsl(var(--color-primary-hsl)) */

  --background:                       var(--color-bg-primary);
  --foreground:                       var(--color-text-primary);

  --card:                             var(--color-bg-elevated);
  --card-foreground:                  var(--color-text-primary);

  --popover:                          var(--color-bg-elevated);
  --popover-foreground:               var(--color-text-primary);

  --primary:                          var(--color-primary-500);
  --primary-foreground:               var(--color-text-inverse);

  --secondary:                        var(--color-neutral-100);
  --secondary-foreground:             var(--color-text-primary);

  --muted:                            var(--color-neutral-100);
  --muted-foreground:                 var(--color-text-secondary);

  --accent:                           var(--color-neutral-100);
  --accent-foreground:                var(--color-text-primary);

  --destructive:                      var(--color-danger-500);
  --destructive-foreground:           var(--color-text-inverse);

  --success:                          var(--color-success-500);
  --success-foreground:               var(--color-text-inverse);

  --warning:                          var(--color-warning-500);
  --warning-foreground:               var(--color-text-primary);

  --info:                             var(--color-info-500);
  --info-foreground:                  var(--color-text-inverse);

  --border:                           var(--color-border-default);
  --input:                            var(--color-border-default);
  --ring:                             var(--color-primary-500);

  --radius:                           var(--radius-md);
}

[data-theme="dark"] {
  --background:                       var(--color-bg-primary);
  --foreground:                       var(--color-text-primary);
  --card:                             var(--color-bg-elevated);
  --card-foreground:                  var(--color-text-primary);
  --popover:                          var(--color-bg-elevated);
  --popover-foreground:               var(--color-text-primary);
  --primary:                          var(--color-primary-500);
  --primary-foreground:               var(--color-text-inverse);
  --secondary:                        var(--color-neutral-800);
  --secondary-foreground:             var(--color-text-primary);
  --muted:                            var(--color-neutral-800);
  --muted-foreground:                 var(--color-text-secondary);
  --accent:                           var(--color-neutral-800);
  --accent-foreground:                var(--color-text-primary);
  --destructive:                      var(--color-danger-500);
  --destructive-foreground:           var(--color-text-inverse);
  --border:                           var(--color-border-default);
  --input:                            var(--color-border-default);
  --ring:                             var(--color-primary-500);
}
```

> **Heads up**: shadcn's variables expect HSL components. If your `colors.css` defines
> HEX values, the above may not work — Tailwind's `hsl(var(--primary))` won't parse
> `#6366F1`. You have two options:
>
> 1. **Add HSL-format tokens to `colors.css`** (recommended):
>    ```css
>    --color-primary-500-hsl: 239 84% 67%;  /* #6366F1 in HSL */
>    --color-primary-500: hsl(var(--color-primary-500-hsl));
>    ```
> 2. **Drop the HSL wrapper** in the consumer:
>    ```js
>    // tailwind.config.js
>    colors: { primary: { 500: 'var(--color-primary-500)' } }
>    // then use bg-primary-500 directly, not bg-primary/hsl(var(--primary))
>    ```

### Tailwind config adjustment

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
}
```

---

## MUI (React)

MUI v5's `createTheme` doesn't support `var()` in `shape.borderRadius`. **For full CSS
variable support, recommend MUI v6 (Joy UI)** or use static values.

### Theme file (MUI v5)

```ts
// theme.ts
import { createTheme } from '@mui/material/styles'

export const theme = createTheme({
  palette: {
    mode: 'light',  // or 'dark' — set by user toggle
    primary: {
      main:         'var(--color-primary-500)',
      light:        'var(--color-primary-300)',
      dark:         'var(--color-primary-700)',
      contrastText: 'var(--color-text-inverse)',
    },
    secondary: {
      main:         'var(--color-neutral-500)',
      light:        'var(--color-neutral-300)',
      dark:         'var(--color-neutral-700)',
      contrastText: 'var(--color-text-inverse)',
    },
    error: {
      main:         'var(--color-danger-500)',
      light:        'var(--color-danger-300)',
      dark:         'var(--color-danger-700)',
    },
    warning: {
      main:         'var(--color-warning-500)',
      light:        'var(--color-warning-300)',
      dark:         'var(--color-warning-700)',
    },
    info: {
      main:         'var(--color-info-500)',
      light:        'var(--color-info-300)',
      dark:         'var(--color-info-700)',
    },
    success: {
      main:         'var(--color-success-500)',
      light:        'var(--color-success-300)',
      dark:         'var(--color-success-700)',
    },
    background: {
      default: 'var(--color-bg-primary)',
      paper:   'var(--color-bg-elevated)',
    },
    text: {
      primary:   'var(--color-text-primary)',
      secondary: 'var(--color-text-secondary)',
      disabled:  'var(--color-text-disabled)',
    },
    divider: 'var(--color-border-default)',
  },
  shape: {
    // ⚠️ MUI v5 doesn't support var() here — use static values
    borderRadius: 8,  // matches --radius-md
  },
  typography: {
    fontFamily: 'var(--font-family-body)',
    fontSize: 14,
    h1: { fontFamily: 'var(--font-family-heading)' },
    h2: { fontFamily: 'var(--font-family-heading)' },
    h3: { fontFamily: 'var(--font-family-heading)' },
    h4: { fontFamily: 'var(--font-family-heading)' },
    h5: { fontFamily: 'var(--font-family-heading)' },
    h6: { fontFamily: 'var(--font-family-heading)' },
  },
})
```

### Theme file (MUI v6 / Joy UI — full CSS variable support)

```ts
import { extendTheme } from '@mui/joy/styles'

export const theme = extendTheme({
  colorSchemes: {
    light: {
      palette: {
        primary: {
          solidBg:    'var(--color-primary-500)',
          solidHoverBg: 'var(--color-primary-600)',
          solidActiveBg: 'var(--color-primary-700)',
        },
        // ... map all
      },
    },
  },
  radius: {
    sm: 'var(--radius-sm)',
    md: 'var(--radius-md)',
    lg: 'var(--radius-lg)',
  },
  fontFamily: {
    body: 'var(--font-family-body)',
    display: 'var(--font-family-heading)',
  },
})
```

---

## Chakra UI (React)

```ts
// theme.ts
import { extendTheme } from '@chakra-ui/react'

export const theme = extendTheme({
  semanticTokens: {
    colors: {
      'chakra-body-bg':         { default: 'var(--color-bg-primary)',   _dark: 'var(--color-bg-primary)' },
      'chakra-body-text':       { default: 'var(--color-text-primary)',  _dark: 'var(--color-text-primary)' },
      'chakra-border-color':    { default: 'var(--color-border-default)', _dark: 'var(--color-border-default)' },
      'chakra-subtle-bg':       { default: 'var(--color-bg-secondary)',  _dark: 'var(--color-bg-elevated)' },
      'chakra-subtle-text':     { default: 'var(--color-text-secondary)', _dark: 'var(--color-text-secondary)' },
      'chakra-placeholder':     { default: 'var(--color-text-disabled)',  _dark: 'var(--color-text-disabled)' },
    },
  },
  colors: {
    brand: {
      50:  'var(--color-primary-50)',
      100: 'var(--color-primary-100)',
      200: 'var(--color-primary-200)',
      300: 'var(--color-primary-300)',
      400: 'var(--color-primary-400)',
      500: 'var(--color-primary-500)',
      600: 'var(--color-primary-600)',
      700: 'var(--color-primary-700)',
      800: 'var(--color-primary-800)',
      900: 'var(--color-primary-900)',
    },
  },
  radii: {
    sm: 'var(--radius-sm)',
    md: 'var(--radius-md)',
    lg: 'var(--radius-lg)',
  },
  fonts: {
    body: 'var(--font-family-body)',
    heading: 'var(--font-family-heading)',
    mono: 'var(--font-family-mono)',
  },
})
```

---

## Vuetify (Vue 3)

```ts
// vuetify-theme.ts
import { createVuetify } from 'vuetify'
import 'vuetify/styles'
import './ui-theme/tokens/theme.css'
import './ui-theme/overrides/vuetify-theme-override.css'

export const vuetify = createVuetify({
  theme: {
    defaultTheme: 'custom',
    themes: {
      custom: {
        dark: false,
        colors: {
          background:  'var(--color-bg-primary)',
          surface:     'var(--color-bg-elevated)',
          primary:     'var(--color-primary-500)',
          secondary:   'var(--color-neutral-500)',
          success:     'var(--color-success-500)',
          warning:     'var(--color-warning-500)',
          error:       'var(--color-danger-500)',
          info:        'var(--color-info-500)',
          'on-primary':    'var(--color-text-inverse)',
          'on-surface':    'var(--color-text-primary)',
          'on-background': 'var(--color-text-primary)',
        },
      },
    },
  },
})
```

---

## How to Choose a Library

If the user is unsure, ask these questions and recommend based on answers:

| User says                                                | Recommend                            |
|----------------------------------------------------------|--------------------------------------|
| "我项目是 Vue 3" + "公司强制" / "老项目"                  | Element Plus                         |
| "我项目是 Vue 3" + "想用 TS 优先"                        | Naive UI                             |
| "我项目是 React" + "国内项目" / "后台"                    | Ant Design                           |
| "我项目是 React" + "想用 Tailwind"                        | shadcn/ui                            |
| "我项目是 React" + "Material Design 风格"                 | MUI                                  |
| "我项目是 React" + "想快速拼"                             | Chakra UI                            |
| "我项目是 Vue 3" + "Material Design 风格"                | Vuetify                              |
| "老项目是 Bootstrap / Foundation"                          | 不支持(这两个没有 CSS 变量体系)      |

If the user has no preference, **default to Element Plus** for Vue or **Ant Design** for
React — both have the most complete CSS variable coverage.

---

## Layout / Grid / Breakpoint Overrides

Grid systems in most UI libraries use a mix of CSS variables and JS props. The CSS
variable part (gutter / columns at the CSS layer) is overridable; the JS part (Row span
on Ant Design `<Col span={n}>`, Element Plus `<el-col :span="n">`) is **not** — those
props still take numbers but the visual gap is controlled by CSS variables.

### Element Plus

```css
/* Append to overrides/element-plus-theme-override.css */
:root {
  --el-grid-columns:           var(--grid-columns);
  --el-grid-gutter:            var(--grid-gutter);
  --el-grid-row-gap:           var(--grid-gutter);
  --el-grid-column-gap:        var(--grid-gutter);

  /* Breakpoints — Element Plus uses these for responsive props */
  --el-breakpoint-xs:          480px;
  --el-breakpoint-sm:          var(--breakpoint-sm);
  --el-breakpoint-md:          var(--breakpoint-md);
  --el-breakpoint-lg:          var(--breakpoint-lg);
  --el-breakpoint-xl:          var(--breakpoint-xl);

  /* Container */
  --el-container-padding:      var(--container-padding);
  --el-container-max-width:    var(--container-max-width);
}
```

### Ant Design

```css
/* Append to overrides/antd-theme-override.css */
:root {
  --ant-grid-columns:          var(--grid-columns);
  --ant-grid-gutter-width:     var(--grid-gutter);
  --ant-grid-row-gutter-width: var(--grid-gutter);

  --ant-screen-xs:             480px;
  --ant-screen-sm:             var(--breakpoint-sm);
  --ant-screen-md:             var(--breakpoint-md);
  --ant-screen-lg:             var(--breakpoint-lg);
  --ant-screen-xl:             var(--breakpoint-xl);
  --ant-screen-xxl:            var(--breakpoint-2xl);

  --ant-container-max-width:   var(--container-max-width);
}
```

### shadcn / Tailwind

Tailwind handles grid via `tailwind.config.js` — override in JS, not CSS:

```js
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      sm: '640px',   // --breakpoint-sm
      md: '768px',
      lg: '1024px',
      xl: '1280px',
      '2xl': '1536px',
    },
    container: {
      center: true,
      padding: '1rem',  // --container-padding
      screens: {  // override container max-width per breakpoint
        sm: '640px',
        md: '768px',
        lg: '1024px',
        xl: '1280px',
        '2xl': '1536px',
      },
    },
  },
}
```

### Vuetify

```ts
// In vuetify-theme.ts
export const vuetify = createVuetify({
  theme: { /* ... colors ... */ },
  defaults: {
    VContainer: { maxWidth: 'var(--container-max-width)' },
    VRow: { noGutters: false },
  },
  display: {
    thresholds: {
      xs: 0, sm: 640, md: 768, lg: 1024, xl: 1280, xxl: 1536,
    },
  },
})
```

### Naive UI

```ts
// naive-theme.ts — append to themeOverrides.common
export const themeOverrides = {
  common: {
    /* ... colors / fonts ... */
    heightSmall:  '24px',
    heightMedium: '32px',
    heightLarge:  '40px',
    heightHuge:   '48px',
  },
}
// Naive UI grid is NGrid / NGi — gutter is a prop, not a CSS var.
// Use a wrapper CSS rule for responsive grid.
```

### Quick Library Support Matrix (Layout)

| Library        | CSS var override for gutter | CSS var override for breakpoints | Grid column count change |
|----------------|----------------------------|----------------------------------|--------------------------|
| Element Plus   | ✅ `--el-grid-gutter`       | ✅ `--el-breakpoint-*`            | ⚠️ partial (still uses 24 internal) |
| Ant Design     | ✅ `--ant-grid-gutter-width`| ✅ `--ant-screen-*`               | ⚠️ partial |
| shadcn / TW    | ❌ use tailwind.config      | ❌ use tailwind.config           | ✅ fully configurable    |
| Naive UI       | ❌ gutter is prop           | ❌ use theme breakpoints          | ✅ fully configurable    |
| MUI v6 / Joy   | ✅ CSS vars                 | ✅ CSS vars                       | ✅ fully configurable    |
| Chakra UI      | ✅ CSS vars                 | ✅ theme tokens                   | ✅ fully configurable    |
| Vuetify        | ⚠️ mixed                   | ✅ `display.thresholds`           | ✅ fully configurable    |

If the new design demands a grid column count change (12 → 24), use direct CSS for the
breakpoint layer. The `<Col span={n}>` prop continues to work; only the gutter and
breakpoint behavior shifts.

---

## Extension Patterns

When the new design has visual patterns the lib doesn't expose via CSS variables, use
**direct CSS** (not just `--var` overrides). Examples:

```css
/* New design uses 5xl / 6xl heading sizes — Element Plus only has h1-h6 */
:root {
  --el-font-size-extra-large: 4rem;     /* 64px */
  --el-font-size-display:      5.5rem;  /* 88px */
}

/* New design uses more shadow levels than the lib exposes */
.el-card.is-elevated {
  box-shadow: var(--shadow-xl);
}

/* New design has a brand-tinted glow */
.el-button.is-primary:hover {
  box-shadow: var(--shadow-primary-glow);
}

/* New design uses unusual control heights */
.el-button.is-jumbo {
  height: 56px;
  padding: 0 32px;
  font-size: var(--font-size-lg);
}
```

**Rule**: When you can't find the right CSS variable, use the lib's CSS class
(`.el-button`, `.ant-btn`, etc.) + `:where()` to keep specificity low so user code can
still override.

```css
/* ✅ good — low specificity */
:where(.el-button).is-primary {
  background: var(--color-primary-500);
}

/* ❌ avoid — high specificity prevents user override */
body .el-button.el-button--primary {
  background: var(--color-primary-500);
}
```

---

## Working with Forked / Internal Component Libraries

> **Reality check**: 80% of company-internal "组件库" are forks of Element Plus /
> Ant Design / Naive UI with a different prefix, some variables renamed, others
> extended. **The mappings below are starting templates, not final truth.** Always
> run [Phase 0](system-prompt.md#phase-0-target-library-discovery-do-this-first-for-any-lib) first to discover the actual variables your lib exposes.

### Common fork patterns

| Upstream     | Forked prefix examples              | Typical renames                           |
|--------------|--------------------------------------|-------------------------------------------|
| Element Plus | `--el-` → `--my-`, `--dc-`, `--company-`, `--xxx-brand-` | color-primary → color-brand               |
| Ant Design   | `--ant-` → same as above             | colorPrimary → brandColor, colorText → textColor |
| Naive UI     | `--n-` → same as above               | primaryColor → brandColor                 |
| shadcn/ui    | `--` (no prefix)                     | foreground → text-primary                 |
| Vuetify      | `--v-` → `--my-`                     | primary → brand                           |

### Step 1: Run the discovery script (Phase 0 in system-prompt.md)

Get the actual list of variables the lib uses. Save the output.

### Step 2: Apply prefix transform to the template mappings

Use a small script to generate the forked mapping from any of the templates in
this file:

```js
// Adjust these
const UPSTREAM_PREFIX = '--el-'        // the prefix in the template
const FORK_PREFIX = '--my-'             // the prefix in the user's fork
const VARIABLE_RENAMES = {
  // Optional: variables that were renamed (not just prefixed)
  // '--el-color-primary': '--my-color-brand',
  // '--el-color-success': '--my-color-positive',
}

const template = {
  // Paste the mapping from above (Element Plus, Ant Design, etc.)
  '--el-color-primary':           'var(--color-primary-500)',
  '--el-color-primary-light-3':   'var(--color-primary-400)',
  // ... 100 more
}

const forkMapping = Object.fromEntries(
  Object.entries(template).map(([k, v]) => {
    let newKey = k.replace(UPSTREAM_PREFIX, FORK_PREFIX)
    if (VARIABLE_RENAMES[k]) newKey = VARIABLE_RENAMES[k]
    return [newKey, v]
  })
)

// Filter to only variables that actually exist in the lib
const ACTUAL_VARS = new Set([
  // Paste the output from the Phase 0 dump
  // '--my-color-brand',
  // '--my-color-positive',
  // ...
])

const finalMapping = Object.fromEntries(
  Object.entries(forkMapping).filter(([k]) => ACTUAL_VARS.has(k))
)

console.log(`Mapped ${Object.keys(finalMapping).length} of ${Object.keys(forkMapping).length} template entries`)
console.log(`Unused template entries:`, Object.keys(forkMapping).filter(k => !ACTUAL_VARS.has(k)))
console.log(`Lib variables not covered:`, [...ACTUAL_VARS].filter(k => !forkMapping[k]))
```

**This gives you three lists**:
1. **Mapped** (most of them) — use in the override file
2. **Template entries the lib doesn't have** — remove from template (lib was simplified)
3. **Lib variables not in the template** — add manually (lib was extended)

### Step 3: Add the missing variables (lib extensions)

Common extensions in forked libs:

```css
/* Company-specific brand tokens */
--my-color-brand:        var(--color-primary-500);    /* their renamed primary */
--my-color-brand-hover:  var(--color-primary-600);
--my-color-positive:     var(--color-success-500);    /* their renamed success */

/* Company-specific surfaces */
--my-bg-page:            var(--color-bg-primary);
--my-bg-card:            var(--color-bg-elevated);
--my-bg-hover:           var(--color-neutral-100);

/* Company-specific typography */
--my-text-display:       var(--font-size-4xl);
--my-text-caption:       var(--font-size-xs);
```

The user will need to provide these — they're not in any upstream template.

### Step 4: Handle the "removed variable" case

Sometimes a fork **removes** a variable. If the lib doesn't have
`--my-color-warning-light-5` anymore, the override mapping for it is a no-op —
**don't include it**. The agent should not write CSS for variables that don't
exist; it just bloats the file and may cause confusion.

### Step 5: Handle the "SCSS-only" case

Element Plus 1.x and Vuetify don't expose CSS variables directly — they use
SCSS preprocessor variables compiled at build time. In this case:

**Option A: Upgrade the lib to v2 (recommended)**
- Element Plus 2.x exposes `--el-*` variables; 1.x doesn't
- Ant Design 5.x exposes `--ant-*` variables; 4.x doesn't

**Option B: Write CSS variable wrappers**
```scss
// In your own styles, before the lib's CSS
:root {
  --my-color-primary: #6366F1;  // your design token
}

// Re-define the lib's SCSS vars via CSS
$--color-primary: var(--my-color-primary);  // SCSS-level, not CSS
```

This requires SCSS build config. Use only if upgrading isn't an option.

**Option C: Use direct CSS instead of variable override**
```css
/* No variable to override, so we override the property directly */
:where(.el-button.el-button--primary) {
  background-color: var(--color-primary-500);
  border-color:     var(--color-primary-500);
}
```

This is the most fragile but works without any lib changes. Use sparingly.

### Validation: cross-check vs the actual rendered page

After generating the override, verify on a real page:

```js
// For each variable in your override file, check it's actually defined
const overrideVars = [
  '--my-color-primary', '--my-color-brand', // ...
]
const actualVars = new Set([...document.styleSheets].flatMap(sheet => {
  try { return [...sheet.cssRules].filter(r => r.selectorText === ':root')
    .flatMap(r => [...r.style].filter(p => p.startsWith('--'))) } catch (e) { return [] }
}))

const missing = overrideVars.filter(v => !actualVars.has(v))
if (missing.length) console.warn('Override targets non-existent variables:', missing)
```

If a target variable doesn't exist, the override silently does nothing. Always
check the rendered output, not just the file content.

### Template maintenance

The mappings in this file are maintained against **stable, major-version
branches** of upstream libs:

- Element Plus: 2.4+
- Ant Design: 5.0+
- Naive UI: 2.36+
- shadcn/ui: latest (continuously updated)
- MUI: 5.0+
- Chakra UI: 2.0+
- Vuetify: 3.0+ (v2 has no CSS variables)

If the user's lib is older, the variable names may differ. Run Phase 0 discovery
to confirm.
