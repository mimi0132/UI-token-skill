# Preview: Comprehensive Component Coverage

## Page Structure & Visual Hierarchy

预览页必须**结构清晰、命名明确**。如果一坨组件堆在一起没有边界、没有标题、没有状态说明,用户无法判断某个 demo 到底在展示什么 → 也就无法验证 token 替换效果。

**每个组件 / 区块必须遵循这个 HTML 结构**:

```html
<!-- 区块外层 -->
<section class="demo-block" id="cat-buttons">
  <!-- 标题 + 简短说明 -->
  <header class="demo-block__header">
    <h2 class="demo-block__title">
      <span class="demo-block__index">01</span>
      Buttons
      <span class="demo-block__tag">8 variants</span>
    </h2>
    <p class="demo-block__desc">
      验证 primary / default / success / warning / danger / info / text / link
      共 8 种语义按钮的颜色、字号、圆角、阴影、hover 状态。
    </p>
  </header>

  <!-- 分割线 -->
  <hr class="demo-block__divider" />

  <!-- demo 内容(只用真实库类名) -->
  <div class="demo-block__body">
    <button class="el-button el-button--primary">Primary</button>
    <button class="el-button el-button--success">Success</button>
    <button class="el-button" disabled>Disabled</button>
    <!-- ... -->
  </div>
</section>
```

**全局规范**:

| 规则 | 要求 |
|------|------|
| 区块标题 | `<h2>`,左侧带 2 位数字编号 (`01` / `02` / ...) + 名称 + 标签徽章 |
| 区块说明 | 标题下 1 句中文,说明这个区块要验证什么视觉变量 |
| 区块分割 | 每个区块之间用 `<hr>` 分割线,视觉上彼此独立 |
| demo 数量 | 1-3 行 HTML,不超过 1 屏高度,50+ 组件合在一起 1 整页可滚动完 |
| 真实类名 | 全部使用目标库的真实类名(`el-button--primary`),不写自定义 `.btn-primary` |
| 状态数 | 每个组件 demo 至少 2 个状态(默认 + 1 个其他) |
| 颜色 / 文字 / 背景 | demo 区域必须有可读的对比度,不能放白底白字或暗色背景被压黑 |
| 侧边导航 | 顶部或左侧加 `<nav class="toc">` 锚点跳转,让用户能秒跳到任意区块 |
| 区块背景 | 奇偶区块交替 `var(--color-bg-primary)` / `var(--color-bg-secondary)`,视觉有节奏 |

**禁止**:

- ❌ 把所有组件一坨平铺,没有 `<section>` / 标题 / 说明
- ❌ demo 区无任何说明文字,只有 1 个无标签的按钮
- ❌ 自定义类名验证库覆盖(`.btn` / `.button` / `.my-btn`)
- ❌ 内联 `<style>` 修组件样式(必须由 override 文件驱动)

---

## Coverage Standard: 8 categories, 50+ components, all required

A preview that only demos 3 components (Button / Tag / Input) is **incomplete by
definition**. The override file is supposed to change the look of the *entire* library,
and the preview is the visual proof. If 47 components aren't demoed, the user has to
take it on faith that the override works — which is exactly what we're trying to avoid.

**Rule**: every preview must cover all categories below (Category 0 Color System +
Categories 1-8 Components). Missing a category means the deliverable is incomplete;
do not declare done until coverage is full.

---

## Category 0: Color System (必须最先展示)

颜色体系是设计稿的根。预览页必须**先**展示完整的颜色 swatch,让用户在浏览组件 demo 前,
就能一眼验证色阶 / 文字色 / 背景色 / 边框色是否对。**主色不能孤立**,neutral / 4 类语义色 /
文字色 / 背景色 / 边框色都要有独立的 swatch 区块,每个区块都有标题 + 简短说明。

**至少 5 个 sub-block**(每个独立 `<section>`,独立标题编号 `00.1` / `00.2` / ...):

### 00.1 Primary Scale — 10 档色阶

展示 `--color-primary-50` 到 `--color-primary-900` 共 10 档,每档一个 swatch 块,块上写色名 + 16 进制色值 + var 变量名。

```html
<section class="demo-block" id="cat-color-primary">
  <header class="demo-block__header">
    <h2 class="demo-block__title">
      <span class="demo-block__index">00.1</span>
      Primary Scale
      <span class="demo-block__tag">10 stops</span>
    </h2>
    <p class="demo-block__desc">主色 10 档色阶 (50-900),改 `--color-primary-500` 一处,9 档全部自动重新推导。</p>
  </header>
  <hr class="demo-block__divider" />
  <div class="demo-block__body swatch-grid">
    <div class="swatch" style="background: var(--color-primary-50)">
      <span class="swatch__name">primary-50</span>
      <span class="swatch__var">--color-primary-50</span>
    </div>
    <div class="swatch" style="background: var(--color-primary-100)">
      <span class="swatch__name">primary-100</span>
      <span class="swatch__var">--color-primary-100</span>
    </div>
    <!-- ... 100/200/300/400/500/600/700/800/900 ... -->
  </div>
</section>
```

### 00.2 Neutral Scale — 10 档中性色

展示 `--color-neutral-50` 到 `--color-neutral-900`,用于文字、边框、背景次级表面。

### 00.3 Semantic Scales — 4 类语义色,每类 10 档

`--color-success-*` / `--color-warning-*` / `--color-danger-*` / `--color-info-*`,
每类展示 10 档 swatch。可以用 4 个 sub-section 分开展示,也可以用 4 行横向铺开。

### 00.4 Text Colors — 5 档文字色

| 名称 | 变量 | 用途 |
|------|------|------|
| Primary | `--color-text-primary` | 标题、正文主文 |
| Regular | `--color-text-regular` | 常规正文 |
| Secondary | `--color-text-secondary` | 辅助文字 |
| Placeholder | `--color-text-placeholder` | 输入框 placeholder |
| Disabled | `--color-text-disabled` | 禁用状态文字 |

展示方式:每行用该文字色写一段"这是一段示例文字 The quick brown fox jumps over the lazy dog 0123456789",
让用户能直接对比 5 档色阶的层次感。

### 00.5 Background Colors — 4 档背景色

| 名称 | 变量 | 用途 |
|------|------|------|
| Page | `--color-bg-primary` | 页面主背景 |
| Surface | `--color-bg-secondary` | 卡片 / 区块次级背景 |
| Elevated | `--color-bg-elevated` | 弹窗 / 下拉悬浮层 |
| Hover | `--color-bg-hover` | hover 态背景 |

### 00.6 Border Colors — 3 档边框色

| 名称 | 变量 | 用途 |
|------|------|------|
| Default | `--color-border-default` | 常规边框 |
| Strong | `--color-border-strong` | 强调边框 |
| Focus | `--color-border-focus` | 焦点环(配合 `box-shadow: 0 0 0 3px var(--color-border-focus-soft)`) |

展示方式:每个色值画一个 1px / 2px 边框的方块,块内标注变量名。

**Color System 是预览页的"地基",后续 Category 1-8 全部组件的颜色都依赖这里的 6 个 sub-block**。
如果只展示主色,后续组件 demo 的 hover / disabled / placeholder 状态颜色无法被验证。

---

## Category 1: Buttons (8+ variants)

- Primary / Default / Success / Warning / Danger / Info / Text — all in one row
- Disabled (Default)
- Sizes: default / small / large — Primary only
- With icon (e.g., `<el-icon><Search /></el-icon>`)
- Loading state (use `:loading="true"`)
- Plain button (e.g., `el-button--primary is-plain`)
- Round button (e.g., `el-button--primary is-round`)
- Group: `<el-button-group>` with 2-3 buttons
- Link: `<el-button type="primary" link>`

## Category 2: Form (15+ components)

- **Input**: text, password, with prefix icon, with suffix icon, with clearable,
  textarea, disabled
- **InputNumber**: default, with step controls, disabled, sizes
- **Select**: single, multiple, clearable, filterable, with placeholder, disabled
- **Cascader**: default, multiple, filterable, disabled
- **DatePicker**: date, week, month, year, datetime, daterange, disabled
- **TimePicker**: default, fixed time range, disabled
- **TimeSelect**: default
- **Switch**: default, disabled, with text, sizes, with custom icons
- **Checkbox**: default, disabled, indeterminate, with border, group
- **Radio**: default, disabled, group, button group, border
- **Slider**: default, range, with marks, vertical, disabled
- **Form**: with rules, with validation states (valid / invalid / validating),
  disabled, with FormItem labels
- **Upload**: default, drag area, picture card, disabled
- **Transfer**
- **ColorPicker**: default

## Category 3: Data Display (15+ components)

- **Table**: with header row, striped (`stripe`), bordered (`border`), with sizes
  (`small` / `default` / `large`), with selection, with fixed column, with sort
  indicator, with status row
- **Tag**: all 6 types (primary / success / warning / danger / info / default),
  closable, sizes, with dot
- **Card**: default, with header, with body, with footer, with shadow, simple
- **Badge**: with value, with dot, with max=99, types
- **Avatar**: text, image (placeholder), icon, with badge, sizes, shape (circle/square)
- **Descriptions**: default, with border, vertical, with sizes
- **Empty**: default, with image, with bottom content
- **Pagination**: default, with background, small, with total, with jumper
- **Tree**: default, with checkbox, with default-expanded-keys, disabled
- **TreeSelect**: default, multiple
- **Skeleton**: paragraph, with avatar, with image, with active
- **Timeline**: default, with timestamps, with colors
- **Statistic**: with prefix, with suffix, with title, with value-style
- **Progress**: line (with status), circle, dashboard, with custom color
- **Image**: default, with preview, with fit modes, lazy load

## Category 4: Navigation (8+ components)

- **Tabs**: default (top), card style, border-card, with add button, with position
  (left / right / bottom)
- **Menu**: horizontal, vertical, vertical with collapse, with submenu, dark theme
- **Breadcrumb**: default, with separator prop, with icon, with redirect
- **Dropdown**: default (single), with trigger (hover / click / contextmenu), split
  button, sizes
- **Steps**: default (horizontal), vertical, with description, with status
  (wait / process / finish / error / success)
- **Affix**: default offset
- **Anchor**: default with links
- **Backtop**: default
- **PageHeader**: default with breadcrumb and content

## Category 5: Feedback (10+ components)

- **Dialog**: default with header / body / footer, sizes (small / large / full),
  with custom content, with `center`
- **Drawer**: default, with size, with direction (rtl / ltr / ttb / btt), multi-layer
- **Popover**: default, with trigger, with placement (top / right / bottom / left)
- **Tooltip**: default, with placement, controlled, light vs dark theme
- **Alert**: all 4 types (success / warning / info / error), with closable, with
  description, with icon
- **Notification**: triggered (or static) for all 4 types
- **Message**: triggered (or static) for all 4 types, with center, with HTML
- **Loading**: default (visible, no text), with text, with custom spinner, fullscreen
- **Popconfirm**: default
- **MessageBox**: triggered (or static) for alert / confirm / prompt

## Category 6: Layout (6+ components)

- **Container / Header / Aside / Main / Footer**: a complete layout skeleton
- **Row / Col**: with span, with offset, with gutter, with alignment (top / middle /
  bottom / stretch), with responsive
- **Divider**: default, vertical, with text
- **Space**: default, sizes (small / default / large / huge), with alignment, with wrap
- **Scrollbar**: default, with size, with wrap
- **Responsive grid**: 24 columns

## Category 7: Data Entry (Advanced)

- **Autocomplete**: default, with fetch-suggestions
- **Mention**: default
- **Cascader-Panel**: standalone
- **DatePicker-Panel**: standalone

## Category 8: Misc

- **Result**: success, warning, info, error, 404, 403, 500 (each)
- **Carousel**: default, with indicator, with arrow, vertical, autoplay
- **Collapse**: default, accordion, with custom title, with extra
- **Link**: default, primary, success, warning, danger, info, disabled, with icon
- **Text**: default, with type, with size, with tag
- **Icon**: a row of 8-12 common icons
- **Segmented**: default with 3 options
- **Tour**: default
- **Watermark**: default with text and image
- **Rate**: default, with half, with custom icon count
- **ConfigProvider**: wrap the page once to verify locale / size / zIndex all work
- **Scrollbar** (already in Layout)
- **VirtualizedTable**: at least one example with 100 rows

---

## Verification Checklist (Agent runs this before declaring done)

For each of the 50+ components listed above, the preview must satisfy:

- [ ] The component HTML tag is present in the page (`el-button`, `el-input`, etc.)
- [ ] At least 2 states shown (default + 1 other: hover / disabled / focus / active /
      loading)
- [ ] Real library class names are used — `el-button el-button--primary`, **not**
      `.btn-primary` (the latter doesn't prove the override works on the real lib)
- [ ] Component library is loaded via CDN / local bundle, **not** simulated
- [ ] User's `theme.css` is loaded *after* the library CSS so cascade order is right
- [ ] User's override file is loaded *after* `theme.css`
- [ ] Background, text, border, and shadow colors are all driven by the override
      (verify in DevTools that `var(--*)` resolves to the user's value)

## Self-test script (run in browser console)

```js
// 1. Confirm the chain of stylesheets loaded correctly
const links = [...document.styleSheets].map(s => s.href).filter(Boolean)
console.log('stylesheets:', links)

// 2. Confirm component library is actually loaded
console.log('ElementPlus:', !!window.ElementPlus)
// For Ant Design: console.log('antd:', !!window.antd)
// For Naive UI: console.log('naive:', !!window.naive)
// For shadcn: there's no global; check for Radix data attrs

// 3. Verify all 50+ component classes are present
const required = [
  'el-button', 'el-input', 'el-select', 'el-cascader', 'el-date-editor',
  'el-switch', 'el-checkbox', 'el-radio', 'el-slider', 'el-upload', 'el-form',
  'el-form-item', 'el-table', 'el-tag', 'el-card', 'el-badge', 'el-avatar',
  'el-descriptions', 'el-empty', 'el-pagination', 'el-tree', 'el-tree-select',
  'el-skeleton', 'el-timeline', 'el-statistic', 'el-progress', 'el-image',
  'el-tabs', 'el-menu', 'el-breadcrumb', 'el-dropdown', 'el-dropdown-menu',
  'el-steps', 'el-affix', 'el-anchor', 'el-page-header', 'el-dialog',
  'el-drawer', 'el-popper', 'el-tooltip', 'el-popover', 'el-popconfirm',
  'el-alert', 'el-notification', 'el-message', 'el-message-box', 'el-loading',
  'el-container', 'el-header', 'el-aside', 'el-main', 'el-footer',
  'el-row', 'el-col', 'el-divider', 'el-space', 'el-scrollbar',
  'el-result', 'el-carousel', 'el-collapse', 'el-collapse-item', 'el-link',
  'el-text', 'el-icon', 'el-segmented', 'el-tour', 'el-watermark',
  'el-rate', 'el-color-picker', 'el-transfer', 'el-mention', 'el-autocomplete',
  'el-input-number', 'el-time-picker', 'el-time-select'
]
const missing = required.filter(cls => !document.querySelector('.' + cls))
console.log('component coverage:', required.length - missing.length, '/', required.length)
if (missing.length) console.warn('missing:', missing)

// 4. Verify the primary color actually changed
const primaryBtn = document.querySelector('.el-button--primary')
const primaryBg = primaryBtn ? getComputedStyle(primaryBtn).backgroundColor : null
console.log('primary button bg:', primaryBg, '— should NOT be Element Plus default #409EFF')

// 5. Verify the body text font actually changed
const body = getComputedStyle(document.body)
console.log('body font-family:', body.fontFamily, '— should be your design font')

// 6. Verify the body background is your page color
console.log('body bg:', body.backgroundColor)

// 7. Verify card / table / dialog corners are your radius
const card = document.querySelector('.el-card')
console.log('card border-radius:', card ? getComputedStyle(card).borderRadius : 'N/A')

// 8. Verify Color System is fully demoed (Category 0)
//    — at least 6 sub-blocks: primary / neutral / semantic / text / bg / border
const colorSections = ['cat-color-primary', 'cat-color-neutral', 'cat-color-semantic',
                       'cat-color-text', 'cat-color-bg', 'cat-color-border']
  .filter(id => document.getElementById(id))
console.log('color system coverage:', colorSections.length, '/ 6 — must be 6')
if (colorSections.length < 6) console.warn('missing color sub-blocks:',
  ['cat-color-primary','cat-color-neutral','cat-color-semantic',
   'cat-color-text','cat-color-bg','cat-color-border']
   .filter(id => !document.getElementById(id)))

// 9. Verify text colors actually changed
const textPrimary = getComputedStyle(document.documentElement)
  .getPropertyValue('--color-text-primary').trim()
console.log('--color-text-primary:', textPrimary, '— should NOT be #303133 (EP default)')
```

**Pass criteria**:
- `component coverage` ≥ 50
- `color system coverage` = 6/6 (primary + neutral + semantic + text + bg + border)
- `primary button bg` is NOT `#409EFF` (Element Plus default blue)
- `--color-text-primary` is NOT `#303133` (Element Plus default)
- `body font-family` is the user's design font
- `card border-radius` is the user's design radius

**Fail → fix**:
- Coverage < 50 → re-generate the preview; you missed categories
- Color system coverage < 6 → you forgot to demo text / bg / border / neutral / semantic; re-generate
- Primary bg still `#409EFF` → cascade order wrong; check `theme.css` is loaded after
  the library CSS
- Text primary still `#303133` → override file didn't load `--color-text-primary`
- Font / radius still default → the override file didn't load, or used wrong variable
  names

---

## Generation rules for the Agent

1. **Load component library first**, then theme, then override. The order matters.

   ```html
   <head>
     <!-- 1. Library CSS (lowest priority) -->
     <link rel="stylesheet" href="https://unpkg.com/element-plus/dist/index.css" />

     <!-- 2. User's token sources (mid priority) -->
     <link rel="stylesheet" href="../tokens/theme.css" />
     <link rel="stylesheet" href="../tokens/colors.css" />
     <link rel="stylesheet" href="../tokens/typography.css" />
     <link rel="stylesheet" href="../tokens/spacing.css" />
     <link rel="stylesheet" href="../tokens/radius.css" />
     <link rel="stylesheet" href="../tokens/border.css" />
     <link rel="stylesheet" href="../tokens/motion.css" />

     <!-- 3. User's override (highest priority for variable cascade) -->
     <link rel="stylesheet" href="../overrides/el-theme-override.css" />
   </head>
   ```

2. **Use real library class names**, not custom ones. The whole point is to verify
   the override works on the actual library.

   ```html
   <!-- ✅ right -->
   <button class="el-button el-button--primary">Primary</button>

   <!-- ❌ wrong — proves nothing about the library -->
   <button class="btn btn-primary">Primary</button>
   ```

3. **No custom CSS for components**. If a component needs styling, it must be
   styled by the override file, not by inline `<style>` in the preview.

4. **For each component, demo it in 1-3 lines of HTML**. A 50+ component gallery
   should fit in a single page (or one screen of vertical scroll). Don't write
   paragraphs of demo content.

5. **Order by category**, with a `<h2>` heading for each.

6. **Top of page: design tokens** (color swatches / typography sample / shadow
   sample / radius sample / spacing sample). These verify the variables look
   right standalone.

7. **Below: component gallery**. These verify the variables look right when
   applied to real components.

8. **Bottom: self-test script** in a `<script>` tag that runs on page load and
   logs the coverage + cascade verification. The user can open DevTools → Console
   to see the result.

---

## Adapting to other libraries

Same structure, change class names:

| Library | Sample classes |
|---|---|
| Element Plus | `el-button el-button--primary`, `el-input`, `el-table`, `el-select` |
| Ant Design | `ant-btn ant-btn-primary`, `ant-input`, `ant-table`, `ant-select` |
| Naive UI | `n-button n-button--primary-type`, `n-input`, `n-data-table`, `n-select` |
| Vuetify | `v-btn v-btn--variant-elevated`, `v-text-field`, `v-data-table` |
| MUI | `MuiButton-contained`, `MuiTextField-root`, `MuiDataGrid-root` |
| shadcn/ui | Tailwind utilities: `bg-primary`, `text-primary-foreground`, `rounded-md` |
| Arco Design | `arco-btn arco-btn-primary`, `arco-input`, `arco-table` |
| TDesign | `t-button t-button--theme-primary`, `t-input`, `t-table` |

The 8 categories map to similar components in all major libraries. When the user's
library is unknown, ask (Phase 0), then look up the equivalent in that lib's docs.
