# Token Specification Reference

This file is the detailed spec for the 7 token files. Load when the agent needs to know
the exact variable name, value range, or dark-mode mapping.

> **⚠️ BIG CHANGE (v3, 2026-07)**: Tokens are now **component-library-driven**, not
> generic-design-system-driven. We do **NOT** ship a universal `--color-primary-50/100/.../900`
> palette that every project must use. Instead, we ship **3 templates** (Element Plus 2.4,
> Ant Design v5, 中创 fork), each 1:1 matching the real CSS variables of that library.
> See `references/system-prompt.md` for detection and `references/role-glossary.md` for
> natural-language role prompts.

---

## 0. Design principles

### 0.1 Why component-library-driven, not generic design system

A "design system" is not the goal. The goal is: **change the visual layer of an existing
component library without rebuilding it**. If the user has installed Element Plus, they
don't want a generic 10-stop palette named `--color-primary-50..900` mapped onto 85 EP
variables. They want the EP variable names themselves, with new values.

Three supported libraries, three different color structures:

| Library              | Primary color structure                              | Total CSS vars |
|----------------------|------------------------------------------------------|----------------|
| Element Plus 2.4     | 7-stop scale `base / light-3/5/7/8/9 / dark-2`       | ~80            |
| Ant Design v5        | 10-stop scale `1..10` + semantic variants per role   | ~110           |
| 中创 fork            | Inherits EP 2.4 + `--cv-*` extensions                | ~85            |

Each library has its own structure. We **expose each structure as a separate token template**.
Never mix them. Never invent variables that don't exist in the chosen library.

### 0.2 Identify the library (user-driven, primary path)

Ask the user which library they use. The 3 supported options are:
- Element Plus 2.4
- Ant Design v5
- 中创 fork

**Optional detection** (if user is unsure):

```bash
node -p "require('./package.json').dependencies?.['element-plus'] || require('./package.json').devDependencies?.['element-plus']" 2>/dev/null
node -p "require('./package.json').dependencies?.['antd']        || require('./package.json').devDependencies?.['antd']"        2>/dev/null
```

If **none** of the 3 supported libs is selected → STOP. Tell the user:

> "This skill supports Element Plus 2.4, Ant Design v5, and 中创 fork."

See `references/system-prompt.md` for the detection workflow.

### 0.3 Get the real CSS variable list (optional, if lib is installed)

If the user has the library installed, dump the real variables from `node_modules` to verify
the template matches. Otherwise, use the built-in templates directly — they are already
verified against the official versions.

```bash
LIB_DIR="node_modules/element-plus"
[ -d "$LIB_DIR" ] || LIB_DIR="node_modules/antd"

# For Element Plus
grep -oE -- "--el-[a-z0-9-]+" "$LIB_DIR/dist/index.css" 2>/dev/null | sort -u

# For Ant Design v5
grep -oE "colorPrimary[A-Za-z]*" "$LIB_DIR/lib/theme/interface/seeds.ts" \
  "$LIB_DIR/lib/theme/interface/maps.ts" 2>/dev/null | sort -u
```

**Output** of this dump = optional cross-reference against § 1 / § 2 / § 3 of this document.
If the dump contains variables not in the template, add them before generating tokens.

### 0.4 The workflow

1. **Ask user** which lib they use (see 0.2).
2. **Pick the matching template** from § 1 (EP), § 2 (AntD), or § 3 (中创) below.
3. **Optional**: If lib is installed, cross-check variables (see 0.3).
4. **Fill the template** from the design source.

### 0.5 Color derivation algorithm (color-mix) — when the design only gives 1 stop

The lib's real variables come in **N stops** (EP: 7, AntD: 10, 中创: 7). If the design source
only gives **1 anchor color** for a role, derive the other stops with `color-mix()`.

**Element Plus 7-stop scale** (light-3/5/7/8/9 + dark-2 derived from base 500):

```css
--color-primary-base:     #6366F1;   /* anchor from design, 500 stop */

/* Lighter (mix with white) */
--color-primary-light-3:  color-mix(in srgb, var(--color-primary-base) 75%, white); /* 300 */
--color-primary-light-5:  color-mix(in srgb, var(--color-primary-base) 90%, white); /* 100 */
--color-primary-light-7:  color-mix(in srgb, var(--color-primary-base) 96%, white); /* 50 */
--color-primary-light-8:  color-mix(in srgb, var(--color-primary-base) 98%, white);
--color-primary-light-9:  color-mix(in srgb, var(--color-primary-base) 99%, white);

/* Darker (mix with black) */
--color-primary-dark-2:   color-mix(in srgb, var(--color-primary-base) 84%, black); /* 700 */
```

**Ant Design 10-stop scale** (1..10 from base 6):

```css
--color-primary-6: #6366F1;   /* anchor from design */

/* Lighter (mix with white) */
--color-primary-1: color-mix(in srgb, var(--color-primary-6) 10%, white);
--color-primary-2: color-mix(in srgb, var(--color-primary-6) 20%, white);
--color-primary-3: color-mix(in srgb, var(--color-primary-6) 35%, white);
--color-primary-4: color-mix(in srgb, var(--color-primary-6) 55%, white);
--color-primary-5: color-mix(in srgb, var(--color-primary-6) 80%, white);

/* Darker (mix with black) */
--color-primary-7: color-mix(in srgb, var(--color-primary-6) 80%, black);
--color-primary-8: color-mix(in srgb, var(--color-primary-6) 65%, black);
--color-primary-9: color-mix(in srgb, var(--color-primary-6) 45%, black);
--color-primary-10: color-mix(in srgb, var(--color-primary-6) 25%, black);
```

**When the design gives more than 1 stop** (e.g. primary-500 + primary-700), use the given
stops as anchors and only derive the missing ones. **Never mix hand-picked hex into a derived
scale unless the design explicitly specifies that exact stop.**

### 0.6 Naming convention

For each lib, the token names **MATCH the lib's own variable names**, with a lib-specific
prefix that the override file uses to re-route them:

| Lib              | Token template prefix      | Override target prefix     |
|------------------|----------------------------|----------------------------|
| Element Plus 2.4 | `--color-<role>-<stop>`    | `--el-<role>-<stop>`       |
| Ant Design v5    | `--color-<role>-<stop>`    | `--ant-<role>-<stop>`      |
| 中创 fork        | `--color-<role>-<stop>`    | `--el-<role>-<stop>` + `--cv-<role>` |

Example for EP primary: our token is `--color-primary-base`, override is
`--el-color-primary: var(--color-primary-base)`. This way the project's tokens stay lib-agnostic
in the source file, but the override translates 1:1 to the real CSS variable.

**For other token categories (typography, spacing, radius, etc.)** the same rule applies:
use the lib's own variable names, do not invent new categories.

---

## 1. Element Plus 2.4 token template

> **Dump the real variables first** (§ 0.3). The list below is the standard set for 2.4.x.
> If the dump contains extra variables not listed here, add them before generating tokens.

### 1.1 Color roles (natural language → EP variable)

When asking the user which design color maps to which role, use **natural language** (see
`role-glossary.md`). Do NOT ask the user to fill in `primary` / `success` etc.

| # | Natural language role                              | EP variable group       | Stops                              |
|---|-----------------------------------------------------|-------------------------|------------------------------------|
| 1 | 品牌主色 (按钮、链接、品牌元素)                     | `--el-color-primary`    | base/light-3/5/7/8/9/dark-2        |
| 2 | 成功色 (对勾、确认、完成)                           | `--el-color-success`    | base/light-3/5/7/8/9/dark-2        |
| 3 | 警告色 (提示、需注意)                               | `--el-color-warning`    | base/light-3/5/7/8/9/dark-2        |
| 4 | 错误/危险色 (错误提示、删除)                        | `--el-color-danger`     | base/light-3/5/7/8/9/dark-2        |
| 5 | 信息色 (普通通知、可点击)                           | `--el-color-info`       | base/light-3/5/7/8/9/dark-2        |
| 6 | 页面背景色 (整页底色)                               | `--el-bg-color-page`    | 1                                  |
| 7 | 卡片/弹窗背景 (默认白色)                            | `--el-bg-color`         | 1                                  |
| 8 | 浮层背景 (下拉、tooltip、弹窗)                      | `--el-bg-color-overlay` | 1                                  |
| 9 | 主要文字 (标题)                                     | `--el-text-color-primary`     | 1                            |
| 10| 正文文字 (段落、表单)                               | `--el-text-color-regular`     | 1                            |
| 11| 次要文字 (辅助说明)                                 | `--el-text-color-secondary`   | 1                            |
| 12| 占位文字 (input placeholder)                       | `--el-text-color-placeholder` | 1                            |
| 13| 禁用文字                                           | `--el-text-color-disabled`    | 1                            |
| 14| 边框 (默认卡片边框)                                | `--el-border-color`      | 1                                 |
| 15| 浅边框 (表格内分隔)                                | `--el-border-color-light`     | 1                            |
| 16| 更浅边框 (装饰)                                    | `--el-border-color-lighter`   | 1                            |
| 17| 最浅边框 (虚线装饰)                                | `--el-border-color-extra-light` | 1                            |
| 18| 深边框 (输入框默认)                                | `--el-border-color-dark`     | 1                            |
| 19| 边框 hover                                        | `--el-border-color-hover`    | 1                            |
| 20| 填充 (按钮次级背景)                                | `--el-fill-color`         | 1                                 |
| 21| 浅填充 (下拉 hover)                                | `--el-fill-color-light`    | 1                                 |
| 22| 禁用背景                                           | `--el-disabled-bg-color`   | 1                                 |
| 23| 遮罩 (弹窗蒙层)                                    | `--el-mask-color`         | 1                                 |

For the full list (60+ variables) including light/lighter/extra-light fill colors, overlay-color
variants, box-shadow variants, and component-specific overrides, see `override-patterns.md`.

### 1.2 Element Plus token file shape

```css
/* theme.css (light) */
:root {
  /* 1. 品牌主色 — 设计稿给 base, 其他档用 color-mix 推导 */
  --color-primary-base:      #6366F1;
  --color-primary-light-3:   color-mix(in srgb, var(--color-primary-base) 75%, white);
  --color-primary-light-5:   color-mix(in srgb, var(--color-primary-base) 90%, white);
  --color-primary-light-7:   color-mix(in srgb, var(--color-primary-base) 96%, white);
  --color-primary-light-8:   color-mix(in srgb, var(--color-primary-base) 98%, white);
  --color-primary-light-9:   color-mix(in srgb, var(--color-primary-base) 99%, white);
  --color-primary-dark-2:    color-mix(in srgb, var(--color-primary-base) 84%, black);

  /* 2-5. 成功/警告/危险/信息色 — 同样 7 档 (省略, 同上结构) */
  --color-success-base:      #67C23A;
  /* ... color-mix 推导 light-3/5/7/8/9 + dark-2 */

  /* 6-8. 背景色 */
  --color-bg-page:           #F2F3F5;
  --color-bg-surface:        #FFFFFF;
  --color-bg-overlay:        #FFFFFF;

  /* 9-13. 文字色 5 档 */
  --color-text-primary:      #303133;
  --color-text-regular:      #606266;
  --color-text-secondary:    #909399;
  --color-text-placeholder:  #A8ABB2;
  --color-text-disabled:     #C0C4CC;

  /* 14-19. 边框色 6 档 + hover */
  --color-border-default:    #DCDFE6;
  --color-border-light:      #E4E7ED;
  --color-border-lighter:    #EBEEF5;
  --color-border-extra-light:#F2F6FC;
  --color-border-dark:       #D4D7DE;
  --color-border-hover:      #C0C4CC;

  /* 20-22. 填充色 + 禁用 */
  --color-fill-default:      #F0F2F5;
  --color-fill-light:        #F5F7FA;
  --color-disabled-bg:       var(--color-fill-light);

  /* 23. 遮罩 */
  --color-mask:              rgba(255, 255, 255, 0.9);
  --color-mask-extra-light:  rgba(255, 255, 255, 0.3);
}
```

**Override file** (`overrides/element-plus-theme-override.css`):
```css
/* 仅做 1:1 路由, 不做语义映射 */
:root {
  --el-color-primary:           var(--color-primary-base);
  --el-color-primary-light-3:   var(--color-primary-light-3);
  --el-color-primary-light-5:   var(--color-primary-light-5);
  --el-color-primary-light-7:   var(--color-primary-light-7);
  --el-color-primary-light-8:   var(--color-primary-light-8);
  --el-color-primary-light-9:   var(--color-primary-light-9);
  --el-color-primary-dark-2:    var(--color-primary-dark-2);

  --el-color-success:           var(--color-success-base);
  /* ... 同样 7 档 */
  --el-color-warning:           var(--color-warning-base);
  --el-color-danger:            var(--color-danger-base);
  --el-color-info:              var(--color-info-base);

  --el-bg-color-page:           var(--color-bg-page);
  --el-bg-color:                var(--color-bg-surface);
  --el-bg-color-overlay:        var(--color-bg-overlay);

  --el-text-color-primary:      var(--color-text-primary);
  --el-text-color-regular:      var(--color-text-regular);
  --el-text-color-secondary:    var(--color-text-secondary);
  --el-text-color-placeholder:  var(--color-text-placeholder);
  --el-text-color-disabled:     var(--color-text-disabled);

  --el-border-color:            var(--color-border-default);
  --el-border-color-light:      var(--color-border-light);
  /* ... */
  --el-border-color-hover:      var(--color-border-hover);

  --el-fill-color:              var(--color-fill-default);
  --el-fill-color-light:        var(--color-fill-light);
  --el-disabled-bg-color:       var(--color-disabled-bg);
  --el-mask-color:              var(--color-mask);
  --el-mask-color-extra-light:  var(--color-mask-extra-light);
}
```

### 1.3 Element Plus non-color tokens

```css
:root {
  /* 字体 */
  --font-family-base:    'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', '微软雅黑', Arial, sans-serif;
  --font-size-extra-large: 20px;
  --font-size-large:     18px;
  --font-size-medium:    16px;
  --font-size-base:      14px;
  --font-size-small:     13px;
  --font-size-extra-small: 12px;
  --font-weight-primary: 500;
  --font-line-height-primary: 24px;

  /* 圆角 */
  --border-radius-base:  4px;
  --border-radius-small: 2px;
  --border-radius-round: 20px;
  --border-radius-circle: 100%;

  /* 过渡 */
  --transition-duration: 0.3s;
  --transition-duration-fast: 0.2s;

  /* 组件尺寸 */
  --component-size-large: 40px;
  --component-size:       32px;
  --component-size-small: 24px;

  /* 边框 */
  --border-width: 1px;
  --border-style: solid;
}
```

Override target: `--el-font-*` / `--el-border-radius-*` / `--el-transition-*` /
`--el-component-size-*` / `--el-border-width`. 1:1 mapping.

---

## 2. Ant Design v5 token template

> **AntD v5 uses CSS-in-JS**, not raw CSS variables. Token names appear inside
> `ConfigProvider({ theme: { token: {...}, components: {...} } })`. To override at runtime
> from CSS, AntD v5.12+ exposes them as `--ant-*` CSS variables (via `cssVar: true`).
> The dump (§ 0.3) confirms which CSS-variable names are available in the installed version.

### 2.1 Color roles (natural language → AntD SeedToken)

| # | Natural language role                              | AntD SeedToken         | MapToken derivatives (10-stop)                                          |
|---|-----------------------------------------------------|------------------------|--------------------------------------------------------------------------|
| 1 | 品牌主色 (按钮、链接)                               | `colorPrimary`         | `colorPrimaryBg/BgHover/Border/BorderHover/Hover/Text/TextHover/TextActive` |
| 2 | 成功色 (对勾、确认)                                | `colorSuccess`         | `colorSuccessBg/BgHover/Border/BorderHover/Text/TextHover/TextActive` + `colorSuccess-1..10` |
| 3 | 警告色                                             | `colorWarning`         | same pattern                                                            |
| 4 | 错误/危险色                                        | `colorError`           | same pattern                                                            |
| 5 | 信息色                                             | `colorInfo`            | same pattern                                                            |
| 6 | 超链接颜色                                         | `colorLink`            | `colorLinkHover/Active`                                                 |
| 7 | 基础文字色 (派生 5 档)                              | `colorTextBase`        | `colorText/TextSecondary/TextTertiary/TextQuaternary/Heading`            |
| 8 | 基础背景色 (派生 5 档)                              | `colorBgBase`          | `colorBgContainer/ContainerDisabled/Elevated/Layout/Spotlight/Mask`     |
| 9 | 边框 (默认)                                        | (derived)              | `colorBorder/BorderSecondary`                                           |
| 10| 填充 (下拉 hover)                                  | (derived)              | `colorFill/FillSecondary/FillTertiary/Quaternary`                       |
| 11| 文字禁用                                           | (derived)              | `colorTextDisabled`                                                     |
| 12| 占位文字 (input)                                   | (derived)              | `colorTextPlaceholder`                                                  |

For the full AntD v5 token list, see `override-patterns.md § Ant Design v5` and the
official docs at https://ant.design/docs/react/customize-theme. For the full SeedToken
schema, see `node_modules/antd/lib/theme/interface/seeds.ts`.

### 2.2 AntD token file shape

```css
/* theme.css (light) */
:root {
  /* 1. 品牌主色 — 1 个 anchor, 10 档 + 8 个语义变体 */
  --color-primary:               #6366F1;  /* anchor, 6 档 */
  --color-primary-1:             color-mix(in srgb, var(--color-primary) 10%, white);
  --color-primary-2:             color-mix(in srgb, var(--color-primary) 20%, white);
  --color-primary-3:             color-mix(in srgb, var(--color-primary) 35%, white);
  --color-primary-4:             color-mix(in srgb, var(--color-primary) 55%, white);
  --color-primary-5:             color-mix(in srgb, var(--color-primary) 80%, white);
  --color-primary-7:             color-mix(in srgb, var(--color-primary) 80%, black);
  --color-primary-8:             color-mix(in srgb, var(--color-primary) 65%, black);
  --color-primary-9:             color-mix(in srgb, var(--color-primary) 45%, black);
  --color-primary-10:            color-mix(in srgb, var(--color-primary) 25%, black);
  --color-primary-bg:            var(--color-primary-1);
  --color-primary-bg-hover:      var(--color-primary-2);
  --color-primary-border:        var(--color-primary-3);
  --color-primary-border-hover:  var(--color-primary-4);
  --color-primary-hover:         var(--color-primary-5);
  --color-primary-text:          var(--color-primary-7);
  --color-primary-text-hover:    var(--color-primary-8);
  --color-primary-text-active:   var(--color-primary-9);

  /* 2-5. 成功/警告/错误/信息色 — 同样结构 (省略) */
  --color-success:               #52C41A;
  --color-warning:               #FAAD14;
  --color-error:                 #FF4D4F;
  --color-info:                  #1677FF;

  /* 6. 超链接 */
  --color-link:                  #1677FF;
  --color-link-hover:            #4096FF;
  --color-link-active:           #0958D9;

  /* 7. 基础文字色 — 派生 5 档 */
  --color-text-base:             #000000;
  --color-text:                  rgba(0, 0, 0, 0.88);
  --color-text-secondary:        rgba(0, 0, 0, 0.65);
  --color-text-tertiary:         rgba(0, 0, 0, 0.45);
  --color-text-quaternary:       rgba(0, 0, 0, 0.25);
  --color-text-disabled:         rgba(0, 0, 0, 0.25);
  --color-text-placeholder:      rgba(0, 0, 0, 0.25);
  --color-text-heading:          rgba(0, 0, 0, 0.88);

  /* 8. 基础背景色 — 派生 5 档 */
  --color-bg-base:               #FFFFFF;
  --color-bg-container:          #FFFFFF;
  --color-bg-container-disabled: rgba(0, 0, 0, 0.04);
  --color-bg-elevated:           #FFFFFF;
  --color-bg-layout:             #F5F5F5;
  --color-bg-spotlight:          rgba(0, 0, 0, 0.85);
  --color-bg-mask:               rgba(0, 0, 0, 0.45);

  /* 9. 边框 */
  --color-border:                #D9D9D9;
  --color-border-secondary:      #F0F0F0;

  /* 10. 填充 */
  --color-fill:                  rgba(0, 0, 0, 0.15);
  --color-fill-secondary:        rgba(0, 0, 0, 0.06);
  --color-fill-tertiary:         rgba(0, 0, 0, 0.04);
  --color-fill-quaternary:       rgba(0, 0, 0, 0.02);
}
```

**Override via ConfigProvider** (AntD v5 idiomatic way):
```jsx
import { ConfigProvider, theme } from 'antd';
import { useToken } from './ui-theme/tokens/theme.css';  /* our CSS tokens */

<ConfigProvider theme={{
  token: {
    colorPrimary:   'var(--color-primary)',
    colorSuccess:   'var(--color-success)',
    colorError:     'var(--color-error)',
    colorWarning:   'var(--color-warning)',
    colorInfo:      'var(--color-info)',
    colorLink:      'var(--color-link)',
    colorTextBase:  'var(--color-text-base)',
    colorBgBase:    'var(--color-bg-base)',
    colorBorder:    'var(--color-border)',
    borderRadius:   6,            /* design value */
    fontSize:       14,            /* design value */
  },
  algorithm: theme.defaultAlgorithm,
}}>
  <App />
</ConfigProvider>
```

**Override via CSS variables** (AntD v5.12+ with `cssVar: true`):
```jsx
<ConfigProvider theme={{ cssVar: true, hashed: false, token: { ... } }}>
```
Then in CSS:
```css
:root {
  --ant-color-primary: var(--color-primary);
  --ant-color-text:   var(--color-text);
  /* ... */
}
```

### 2.3 AntD non-color tokens

```css
:root {
  /* 字体 */
  --font-family:        -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  --font-family-code:   'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace;
  --font-size:          14px;

  /* 圆角 (AntD 用 borderRadius, 单一值) */
  --border-radius:     6px;

  /* 线 */
  --line-width:         1px;
  --line-type:          solid;

  /* 尺寸 */
  --size-unit:          4;
  --size-step:          4;
  --size-popup-arrow:   16px;
  --control-height:     32px;

  /* Z-index */
  --z-index-base:       0;
  --z-index-popup-base: 1000;

  /* 透明度 */
  --opacity-image:      1;

  /* 动效 */
  --motion-unit:        0.1s;     /* 100ms */
  --motion-base:        0;        /* base offset */
}
```

---

## 3. 中创 fork token template

> **中创 fork = Element Plus 2.4 + extensions**. The base token structure is identical
> to § 1 (Element Plus). The fork adds 4 SCSS files (`_avatar.scss`, `_message.scss`,
> `_popconfirm.scss`, `_tooltip.scss`) that introduce `--cv-*` variables and override
> some `--el-*` variables.

### 3.1 Inherited from Element Plus 2.4

All § 1 variables apply unchanged. Token template file `theme.css` is **the same** as EP.

### 3.2 中创-specific extensions (`--cv-*`)

The 中创 SCSS patches introduce these additional variables. **All of them are filled by
the same `--color-*` tokens from § 1**, no new tokens are needed:

| SCSS file              | Variable                       | Maps to (from § 1)                |
|------------------------|--------------------------------|------------------------------------|
| `_avatar.scss`         | `--cv-avatar-text-color`       | `--color-text-regular`             |
| `_message.scss`        | `--cv-message-bg-color`        | `--el-message-bg-color` (default)  |
| `_popconfirm.scss`     | `--cv-popconfirm-bg-color`     | `--el-popconfirm-bg-color` (default) |
| `_tooltip.scss`        | `--cv-tooltip-text-color`      | `--el-text-color-primary` (inverted)|

**These are not new colors** — they are the same colors as their `--el-*` counterparts,
just named with the `cv-` prefix for component-specific override. The override file
re-maps them to the same `--color-*` tokens.

### 3.3 中创 override file shape

```css
/* overrides/zhongchuang-theme-override.css */

/* 1. Inherit ALL Element Plus overrides from § 1.2 */
@import './element-plus-theme-override.css';   /* or copy the rules here */

/* 2. + 中创-specific --cv-* routing */
:root {
  --cv-avatar-text-color:     var(--color-text-regular);
  --cv-message-bg-color:      var(--el-message-bg-color);
  --cv-popconfirm-bg-color:   var(--el-popconfirm-bg-color);
  --cv-tooltip-text-color:    var(--el-text-color-primary);
}
```

> **Important**: The 中创 SCSS patches (`resources/zhongchuang/_*.scss`) are the
> authoritative source for the cv- extension variable names. **Do not modify these files.**
> If a new release adds new --cv- variables, extend the table above before generating tokens.

---

## 4. Color derivation recipes (recap)

When the design source gives you **fewer stops than the lib needs**, derive the rest.
See § 0.5 for the mixing math.

| Lib           | Anchor (from design) | Derive                       |
|---------------|----------------------|------------------------------|
| Element Plus  | `base` (500)         | `light-3/5/7/8/9` + `dark-2` |
| Ant Design    | `color-X` (6 stop)   | `color-X-1/2/3/4/5/7/8/9/10` |
| 中创 fork     | (same as EP)         | (same as EP)                 |

When the design gives you **all stops explicitly** (rare), use them as-is — no derivation.
When the design gives you **only a "hover" variant** but no base, treat hover as 600 / dark-2
and derive the rest.

### 4.1 Lightness/contrast principles

- Lighter stops (light-3/5/7/8/9) = mix with white → backgrounds, hover washes, disabled text.
- Darker stops (dark-2) = mix with black → hover/active states, high-contrast text.
- The ratio between adjacent stops should look perceptually uniform — see the percentages
  in § 0.5. If two adjacent stops look identical, the percentage is too small; if the
  jump looks too big, the percentage is too large.
- The 500 stop (base) is the **brand-defining** color. It should be the most saturated,
  most distinctive stop in the scale. Don't make 500 too close to neutral.

---

## 5. `typography.css` (lib-agnostic)

Typography is more standard across libs. We define a single token file, then route per lib.

```css
:root {
  /* 字体家族 (3 套, 不可推导 — 必须在设计稿里有) */
  --font-family-base:    'Helvetica Neue', 'PingFang SC', 'Microsoft YaHei', Arial, sans-serif;
  --font-family-heading: var(--font-family-base);
  --font-family-mono:    'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace;

  /* 字号梯度 (设计稿给 base, 比例 1.250 派生) */
  --font-size-3xs: 11px;   /* base / 1.25^2 */
  --font-size-2xs: 12px;
  --font-size-xs:  13px;
  --font-size-sm:  14px;   /* base */
  --font-size-md:  16px;   /* base * 1.143 */
  --font-size-lg:  18px;
  --font-size-xl:  20px;
  --font-size-2xl: 24px;
  --font-size-3xl: 30px;
  --font-size-4xl: 36px;

  /* 字重 (4 档, 行业固定) */
  --font-weight-regular:  400;
  --font-weight-medium:   500;
  --font-weight-semibold: 600;
  --font-weight-bold:     700;

  /* 行高 */
  --line-height-tight:   1.25;
  --line-height-base:    1.5;
  --line-height-loose:   1.75;

  /* 字间距 */
  --letter-spacing-tight: -0.01em;
  --letter-spacing-base:  0;
  --letter-spacing-loose: 0.05em;
}
```

**Override target per lib**:
- Element Plus: `--el-font-family` / `--el-font-size-{xs,sm,base,md,lg,xl}` / `--el-font-weight-primary` / `--el-font-line-height-primary`
- Ant Design: `fontFamily` / `fontFamilyCode` / `fontSize` (inside ConfigProvider `theme.token`)
- 中创 fork: same as Element Plus

---

## 6. `spacing.css` (lib-agnostic)

```css
:root {
  /* 基础单位 (设计稿给一个值, 派生 7 档) */
  --space-3xs: 2px;     /* base / 4 */
  --space-2xs: 4px;     /* base / 2 */
  --space-xs:  8px;     /* base */
  --space-sm:  12px;
  --space-md:  16px;    /* base * 2 */
  --space-lg:  24px;    /* base * 3 */
  --space-xl:  32px;    /* base * 4 */
  --space-2xl: 48px;    /* base * 6 */
  --space-3xl: 64px;    /* base * 8 */

  /* 组件内边距 (基于 size-md) */
  --space-component-padding-sm:  var(--space-xs);
  --space-component-padding-md:  var(--space-sm);
  --space-component-padding-lg:  var(--space-md);

  /* 组件间距 (基于 size-md) */
  --space-component-gap-sm:  var(--space-xs);
  --space-component-gap-md:  var(--space-sm);
  --space-component-gap-lg:  var(--space-md);
}
```

EP / 中创 / AntD each have their own component-size variables, but the underlying spacing
scale is shared. Override routes are minimal (most components use the scale directly).

---

## 7. `radius.css` / `border.css` / `motion.css` / `theme.css`

### 7.1 `radius.css`

```css
:root {
  --radius-none: 0;
  --radius-sm:   2px;
  --radius-md:   4px;     /* anchor */
  --radius-lg:   8px;
  --radius-xl:   12px;
  --radius-2xl:  16px;
  --radius-full: 9999px;
}
```

Override: EP uses `--el-border-radius-{base,small,round,circle}`. AntD uses `borderRadius`
(single value, applied as the default for all components via `theme.algorithm` or
`theme.components.Button.borderRadius`).

### 7.2 `border.css`

```css
:root {
  --border-width-thin:  1px;
  --border-width-base:  1px;
  --border-width-thick: 2px;
  --border-style:       solid;
  --border-color:       var(--color-border-default);
}
```

### 7.3 `motion.css`

```css
:root {
  --motion-duration-instant: 0.1s;
  --motion-duration-fast:    0.2s;   /* EP default */
  --motion-duration-base:    0.3s;   /* EP default */
  --motion-duration-slow:    0.5s;
  --motion-ease-standard:    cubic-bezier(0.4, 0, 0.2, 1);
  --motion-ease-decelerate:  cubic-bezier(0, 0, 0.2, 1);
  --motion-ease-accelerate:  cubic-bezier(0.4, 0, 1, 1);
}
```

EP override: `--el-transition-duration` / `--el-transition-duration-fast` /
`--el-transition-function-{ease-in-out-bezier,fast-bezier}`.

### 7.4 `theme.css` (entry point)

```css
/* Import order matters: primitives first, then semantic */
@import './colors.css';
@import './typography.css';
@import './spacing.css';
@import './radius.css';
@import './border.css';
@import './motion.css';
```

---

## 8. Validation rules

A generated token set passes the validation check only if:

1. **Every variable in the lib's CSS dump is either**:
   - Defined in the project's token file, OR
   - Mapped to a `--color-*` token in the override file, OR
   - Left as the lib's default (only acceptable for variables not part of the design).

2. **No variable in the token file references a name that doesn't exist in the lib's dump**.
   E.g. don't define `--color-primary-foo` if EP doesn't have a `--el-color-primary-foo`.

3. **The lib name in the override file matches the lib actually installed** in the project.
   Run `node -p "require('./package.json').dependencies?.['element-plus']"` to confirm (optional).

4. **The workflow (§ 0.4) was followed**: ask user → pick template → fill from design source.

See `references/system-prompt.md` for the audit table that the agent must fill out
before claiming a token set is complete.
