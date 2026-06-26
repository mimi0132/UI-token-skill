# Override Patterns Reference

This file shows how to map design tokens onto each major base component library's
CSS variables. Load when the user picks `--base-lib` so the agent generates the right
`<lib>-theme-override.css`.

---

## Element Plus (Vue 3)

```css
/* overrides/element-plus-theme-override.css */
@import './tokens/theme.css';

:root {
  /* Primary palette */
  --el-color-primary: var(--color-primary-500);
  --el-color-primary-light-3: var(--color-primary-300);
  --el-color-primary-light-5: var(--color-primary-100);
  --el-color-primary-light-7: var(--color-primary-50);
  --el-color-primary-light-8: var(--color-primary-50);
  --el-color-primary-light-9: var(--color-primary-50);
  --el-color-primary-dark-2: var(--color-primary-700);

  /* Semantic */
  --el-color-success: var(--color-success-500);
  --el-color-warning: var(--color-warning-500);
  --el-color-danger:  var(--color-danger-500);
  --el-color-info:    var(--color-info-500);

  /* Border */
  --el-border-radius-base:  var(--radius-md);
  --el-border-radius-small: var(--radius-sm);
  --el-border-radius-round: var(--radius-full);
  --el-border-color:        var(--color-border-default);
  --el-border-color-light:  var(--color-border-default);
  --el-border-color-lighter:var(--color-border-default);
  --el-border-color-extra-light: var(--color-border-default);

  /* Typography */
  --el-font-family: var(--font-family-body);
  --el-font-size-base: var(--font-size-md);

  /* Background */
  --el-bg-color:    var(--color-bg-primary);
  --el-bg-color-page: var(--color-bg-secondary);
  --el-text-color-primary:   var(--color-text-primary);
  --el-text-color-regular:   var(--color-text-primary);
  --el-text-color-secondary: var(--color-text-secondary);
  --el-text-color-placeholder: var(--color-text-disabled);
}
```

```ts
// main.ts
import 'element-plus/dist/index.css'
import 'ui-design-system-extractor/dist/overrides/element-plus-theme-override.css'
```

---

## Ant Design (React)

```css
/* overrides/antd-theme-override.css */
@import './tokens/theme.css';

:root {
  --ant-color-primary:    var(--color-primary-500);
  --ant-color-success:    var(--color-success-500);
  --ant-color-warning:    var(--color-warning-500);
  --ant-color-error:      var(--color-danger-500);
  --ant-color-info:       var(--color-info-500);
  --ant-color-text:       var(--color-text-primary);
  --ant-color-text-secondary: var(--color-text-secondary);
  --ant-color-bg-base:    var(--color-bg-primary);
  --ant-color-bg-container: var(--color-bg-elevated);
  --ant-color-bg-elevated:  var(--color-bg-elevated);
  --ant-color-bg-layout:    var(--color-bg-secondary);
  --ant-color-border:     var(--color-border-default);

  --ant-border-radius:    var(--radius-md);
  --ant-border-radius-sm: var(--radius-sm);
  --ant-border-radius-lg: var(--radius-lg);

  --ant-font-family:      var(--font-family-body);
  --ant-font-size:        var(--font-size-md);
}
```

For runtime theming, use `ConfigProvider`:

```tsx
import { ConfigProvider, theme } from 'antd'
import 'ui-design-system-extractor/dist/overrides/antd-theme-override.css'

export default function App() {
  return (
    <ConfigProvider theme={{ algorithm: theme.defaultAlgorithm }}>
      {/* ... */}
    </ConfigProvider>
  )
}
```

---

## shadcn/ui (React + Tailwind)

shadcn doesn't use CSS variables in the same way — it reads from `tailwind.config.js`.
Generate a Tailwind config that maps to your tokens:

```js
// tailwind.config.js
const tokens = require('./dist/tokens/colors.css') // or import the .css as a string

module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
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
        // ... semantic
      },
      borderRadius: {
        sm: 'var(--radius-sm)',
        md: 'var(--radius-md)',
        lg: 'var(--radius-lg)',
        full: 'var(--radius-full)',
      },
      fontFamily: {
        sans: ['var(--font-family-body)'],
        mono: ['var(--font-family-mono)'],
      },
    },
  },
}
```

For shadcn's CSS variables (`--background`, `--foreground`, etc.):

```css
/* overrides/shadcn-theme-override.css */
@import './tokens/theme.css';

:root {
  --background:           var(--color-bg-primary);
  --foreground:           var(--color-text-primary);
  --card:                 var(--color-bg-elevated);
  --card-foreground:      var(--color-text-primary);
  --popover:              var(--color-bg-elevated);
  --popover-foreground:   var(--color-text-primary);
  --primary:              var(--color-primary-500);
  --primary-foreground:   var(--color-text-inverse);
  --secondary:            var(--color-neutral-100);
  --secondary-foreground: var(--color-text-primary);
  --muted:                var(--color-neutral-100);
  --muted-foreground:     var(--color-text-secondary);
  --accent:               var(--color-neutral-100);
  --accent-foreground:    var(--color-text-primary);
  --destructive:          var(--color-danger-500);
  --destructive-foreground: var(--color-text-inverse);
  --border:               var(--color-border-default);
  --input:                var(--color-border-default);
  --ring:                 var(--color-primary-500);
}
```

---

## Naive UI (Vue 3)

```css
/* overrides/naive-ui-theme-override.css */
@import './tokens/theme.css';

:root {
  --primary-color:           var(--color-primary-500);
  --primary-color-hover:     var(--color-primary-400);
  --primary-color-pressed:   var(--color-primary-600);
  --primary-color-suppl:     var(--color-primary-100);

  --info-color:              var(--color-info-500);
  --success-color:           var(--color-success-500);
  --warning-color:           var(--color-warning-500);
  --error-color:             var(--color-danger-500);

  --text-color-base:         var(--color-text-primary);
  --text-color-1:            var(--color-text-primary);
  --text-color-2:            var(--color-text-secondary);
  --text-color-3:            var(--color-text-secondary);
  --placeholder-color:       var(--color-text-disabled);

  --border-color:            var(--color-border-default);

  --border-radius:           var(--radius-md);
  --border-radius-small:     var(--radius-sm);
}
```

For Naive UI, the override goes through `NConfigProvider` + `themeOverrides` prop in JS,
not just CSS. Generate the JS override object too:

```ts
import { NConfigProvider, theme } from 'naive-ui'
import 'ui-design-system-extractor/dist/overrides/naive-ui-theme-override.css'

const themeOverrides = {
  common: {
    primaryColor: 'var(--color-primary-500)',
    primaryColorHover: 'var(--color-primary-400)',
    // ...
  },
}
```

---

## MUI (React)

MUI uses a JS theme object, not pure CSS variables. Generate both:

```ts
// theme.ts
import { createTheme } from '@mui/material/styles'

export const theme = createTheme({
  palette: {
    primary:   { main: 'var(--color-primary-500)' },
    secondary: { main: 'var(--color-neutral-500)' },
    error:     { main: 'var(--color-danger-500)' },
    warning:   { main: 'var(--color-warning-500)' },
    info:      { main: 'var(--color-info-500)' },
    success:   { main: 'var(--color-success-500)' },
    background: {
      default: 'var(--color-bg-primary)',
      paper:   'var(--color-bg-elevated)',
    },
    text: {
      primary:   'var(--color-text-primary)',
      secondary: 'var(--color-text-secondary)',
      disabled:  'var(--color-text-disabled)',
    },
  },
  shape: {
    borderRadius: 6, // matches --radius-md, MUI doesn't accept var() here in v5
  },
  typography: {
    fontFamily: 'var(--font-family-body)',
    fontSize: 14,
  },
})
```

> **Note**: MUI v5 doesn't support `var()` in `shape.borderRadius`. Either use a static
> value or upgrade to MUI v6 / Joy UI which has CSS variables support.

---

## Chakra UI (React)

```ts
// theme.ts
import { extendTheme } from '@chakra-ui/react'

export const theme = extendTheme({
  semanticTokens: {
    colors: {
      'chakra-body-bg':      { _light: 'var(--color-bg-primary)',  _dark: 'var(--color-bg-primary)' },
      'chakra-body-text':    { _light: 'var(--color-text-primary)', _dark: 'var(--color-text-primary)' },
      'chakra-border-color': { _light: 'var(--color-border-default)', _dark: 'var(--color-border-default)' },
      'brand': {
        50:  'var(--color-primary-50)',
        500: 'var(--color-primary-500)',
        900: 'var(--color-primary-900)',
      },
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

## Choosing a Base Library

If the user is unsure, recommend in this order:

1. **Vue + Element Plus** — most common in Chinese Vue ecosystem, mature, well-documented CSS vars
2. **React + Ant Design** — most common in Chinese React ecosystem, comprehensive
3. **React + shadcn/ui** — fastest-growing, modern, Tailwind-based
4. **Vue + Naive UI** — TypeScript-first, beautiful default theme
5. **React + MUI** — if user is migrating from Material Design
6. **React + Chakra UI** — if user wants composable primitives

If the user has no preference for "override vs standalone", default to **override** — it
lets the user keep their existing component code and just swap styles.
