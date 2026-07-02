# Override Patterns Cookbook

This file is the **core reference** for Step 2 of the workflow. It gives the complete
CSS variable mapping for the **3 supported libraries**: Element Plus 2.4, Ant Design v5,
and the 中创 fork. When the auto-detect (system-prompt.md Phase 0) picks a library,
the agent should generate the override file by copying the relevant section verbatim
and substituting the design tokens from `token-spec.md`.

**Rule**: Always copy the entire section, not just the variables that "seem" relevant.
The libs have ~50–200 CSS variables for a reason — every one affects some visual state.

**Why only 3**: every library has its own variable structure (EP = 7-stop, AntD = 10-stop,
中创 = EP + `--cv-*` extensions). Supporting more libraries means maintaining 3+
parallel templates, more bugs, and inconsistent results. If your project uses a 4th
library, **STOP and add it to this file first**, or refuse the task.

---

## Index

1. [Element Plus 2.4 (Vue 3)](#element-plus-24-vue-3) — `--el-` prefix, 7-stop scale
2. [Ant Design v5 (React)](#ant-design-v5-react) — `<ConfigProvider theme={{ token }}>`, 10-stop scale
3. [中创组件库 (Vue 3, Element Plus fork)](#中创组件库-vue-3-element-plus-fork) — `--el-` + `--cv-` extensions

---

## Element Plus 2.4 (Vue 3)

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
  --el-bg-color-overlay:         var(--color-bg-elevated);
  --el-mask-color:               rgba(0, 0, 0, 0.5);
  --el-mask-color-extra-light:   rgba(0, 0, 0, 0.1);

  /* ===== Text ===== */
  --el-text-color-primary:       var(--color-text-primary);
  --el-text-color-regular:       var(--color-text-regular);
  --el-text-color-secondary:     var(--color-text-secondary);
  --el-text-color-placeholder:   var(--color-text-placeholder);
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
  --el-bg-color-overlay:         var(--color-bg-elevated);
  --el-text-color-primary:       var(--color-text-primary);
  --el-text-color-regular:       var(--color-text-regular);
  --el-text-color-secondary:     var(--color-text-secondary);
  --el-fill-color-blank:         var(--color-bg-elevated);
  --el-border-color:             var(--color-border-default);
  --el-disabled-bg-color:        var(--color-bg-elevated);
}
```

---

## Ant Design v5 (React)

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
  --ant-color-text-placeholder:  var(--color-text-placeholder);
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

## 中创组件库 (Vue 3, Element Plus fork)

> **背景**: 中创组件库 = 公司内部基于 **Element Plus 1.x/2.x fork** 的组件库,沿用 `--el-`
> 前缀,在此基础上**扩展**了若干 `--cv-` 前缀的自定义变量,并配套了 `_element.scss` /
> `_tooltip.scss` / `_avatar.scss` / `_message.scss` / `_popconfirm.scss` 等补丁,
> 覆盖了 el-alert / el-table / el-drawer / el-dialog / el-input / el-select /
> el-textarea / el-input-number / el-pagination / el-popper / el-avatar / el-message
> / el-popconfirm 等组件的尺寸 / 边框 / 阴影 / 文字色细节。
>
> 我们的 skill 在中创上 = **变量覆盖 + scss 补丁并行**,两者互补不冲突。

### 怎么识别是不是中创

| 特征 | 含义 |
|------|------|
| `package.json` 依赖里有 `element-plus` | 上游是 EP |
| 项目里有 `cv-*.scss` 或 `_element.scss` 风格文件 | 中创扩展 |
| CSS 变量前缀是 `--el-` (没改) | 中创沿用 EP 前缀 |
| 但额外有 `--cv-message-success-border-color` 之类的 `--cv-` 变量 | 中创自定义扩展 |
| 使用了 `rem` 而非 EP 默认的 `px` 单位 | 中创的单位体系 |

### Required import

中创需要 SCSS 编译(因为它有自己的 `_element.scss` 补丁),所以比普通 Element Plus 多一个步骤。

中创的 5 个 SCSS 补丁已随 skill 一起提供在 [`resources/zhongchuang/`](../resources/zhongchuang/),
**集成时需先把它们复制到你的项目里**,再按下面的方式 import。

```ts
// main.ts
// 1) 第一次集成时,从中创资源目录复制到项目 (一次性):
//    cp -r <skill>/resources/zhongchuang/* ./src/styles/zhongchuang/

import 'element-plus/dist/index.css'                                    // 1. 库 CSS
import './ui-theme/tokens/theme.css'                                     // 2. token
import './ui-theme/overrides/zhongchuang-theme-override.css'             // 3. override (CSS 变量映射)
import '@/styles/zhongchuang/_element.scss'                              // 4. 中创 scss 布局补丁(放最后)
```

或者用 Vite:

```ts
// main.ts
import 'element-plus/dist/index.css'
import './ui-theme/tokens/theme.css'
import './ui-theme/overrides/zhongchuang-theme-override.css'
import '@/styles/zhongchuang/_element.scss'  // 中创的 scss 入口(会自动 @use 其他 4 个)
```

> **顺序很关键**: 中创 scss 必须在 override 之后加载,因为 scss 里的 `!important` /
> 精确 padding 等细节会"压住"我们 CSS 变量映射。但我们的颜色 / 字号 / 圆角已经在
> override 阶段设好了,scss 不会动这些。

### 中创 SCSS 资源位置

中创的 5 个 scss 补丁**随 skill 一起分发**,不依赖用户项目本地路径:

```
skills/ui-design-system-extractor/
└── resources/
    └── zhongchuang/                  # 中创补丁目录
        ├── README.md                 # 集成说明
        ├── _element.scss             # 入口
        ├── _avatar.scss
        ├── _message.scss
        ├── _popconfirm.scss
        └── _tooltip.scss
```

集成时执行一次复制命令:

```bash
# 从 skill 安装目录(因 Agent 不同路径会变,常见位置)
# Cursor / Continue:   ~/.cursor/skills/ui-design-system-extractor/resources/zhongchuang/
# Trae:               ~/.trae-cn/mcps/.../ui-design-system-extractor/resources/zhongchuang/
# Cline:              ~/.cline/skills/ui-design-system-extractor/resources/zhongchuang/

cp -r <skill-install-path>/resources/zhongchuang ./src/styles/
```

详见 [`resources/zhongchuang/README.md`](../resources/zhongchuang/README.md)。

### Override file 模板

```css
/* overrides/zhongchuang-theme-override.css */
@import '../tokens/theme.css';

:root {
  /* ===== Element Plus 基础色(中创沿用 --el- 前缀) ===== */
  --el-color-primary:           var(--color-primary-500);
  --el-color-primary-light-3:   var(--color-primary-300);
  --el-color-primary-light-5:   var(--color-primary-100);
  --el-color-primary-light-7:   var(--color-primary-50);
  --el-color-primary-light-8:   var(--color-primary-50);
  --el-color-primary-light-9:   var(--color-primary-50);
  --el-color-primary-dark-2:    var(--color-primary-700);

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

  --el-color-white:             #ffffff;
  --el-color-black:             #000000;

  /* ===== 文字色(中创重点用 --el-text-color-*) ===== */
  --el-text-color-primary:      var(--color-text-primary);
  --el-text-color-regular:      var(--color-text-regular);
  --el-text-color-secondary:    var(--color-text-secondary);
  --el-text-color-placeholder:  var(--color-text-placeholder);
  --el-text-color-disabled:     var(--color-text-disabled);

  /* ===== 背景色 ===== */
  --el-bg-color:                var(--color-bg-primary);
  --el-bg-color-page:           var(--color-bg-primary);
  --el-bg-color-overlay:        var(--color-bg-elevated);
  --el-mask-color:              rgba(0, 0, 0, 0.5);
  --el-mask-color-extra-light:  rgba(0, 0, 0, 0.1);

  /* ===== 填充色(中创的 el-pagination / el-input 大量用) ===== */
  --el-fill-color:              var(--color-neutral-100);
  --el-fill-color-light:        var(--color-neutral-50);
  --el-fill-color-lighter:      var(--color-neutral-50);
  --el-fill-color-extra-light:  var(--color-neutral-50);
  --el-fill-color-blank:        #ffffff;

  /* ===== 边框色 ===== */
  --el-border-color:            var(--color-border-default);
  --el-border-color-light:      var(--color-border-default);
  --el-border-color-lighter:    var(--color-border-default);
  --el-border-color-extra-light: var(--color-border-default);
  --el-border-color-hover:      var(--color-border-strong);

  /* ===== 圆角 ===== */
  --el-border-radius-base:      var(--radius-md);
  --el-border-radius-small:     var(--radius-sm);
  --el-border-radius-round:     var(--radius-full);
  --el-border-radius-circle:    var(--radius-full);

  /* ===== 阴影 ===== */
  --el-box-shadow:              var(--shadow-md);
  --el-box-shadow-light:        var(--shadow-sm);
  --el-box-shadow-lighter:      var(--shadow-sm);
  --el-box-shadow-dark:         var(--shadow-lg);

  /* ===== 禁用遮罩 ===== */
  --el-disabled-bg-color:       var(--color-neutral-100);
  --el-disabled-text-color:     var(--color-text-disabled);
  --el-disabled-border-color:   var(--color-border-default);

  /* ===== 文字字号(中创用 rem 单位,映射到 token) ===== */
  --el-font-size-extra-small:   var(--font-size-xs);
  --el-font-size-small:         var(--font-size-sm);
  --el-font-size-base:          var(--font-size-base);
  --el-font-size-medium:        var(--font-size-md);
  --el-font-size-large:         var(--font-size-lg);
  --el-font-size-extra-large:   var(--font-size-xl);

  /* ===== Pagination(中创的 el-pagination 有 hover-color 自定义) ===== */
  --el-pagination-hover-color:  var(--color-primary-500);

  /* ===== Input placeholder (中创 _element.scss 显式将其重写为 --el-text-color-secondary) ===== */
  --el-input-placeholder-color: var(--color-text-secondary);

  /* ===== Message 文字色 ===== */
  --el-message-text-color:      var(--color-text-primary);

  /* ===== 中创扩展的 --cv- 自定义变量(从 _message.scss 反推) ===== */
  --cv-message-success-border-color: var(--color-success-500);
  --cv-message-warning-border-color: var(--color-warning-500);
  --cv-message-error-border-color:   var(--color-danger-500);
  --cv-message-info-border-color:    var(--color-info-500);
}

[data-theme="dark"] {
  /* 暗色模式:中创跟随 EP 默认的暗色映射,我们把基础色和文字色都反转 */
  --el-text-color-primary:      var(--color-text-primary);
  --el-text-color-regular:      var(--color-text-regular);
  --el-text-color-secondary:    var(--color-text-secondary);
  --el-text-color-placeholder:  var(--color-text-placeholder);
  --el-text-color-disabled:     var(--color-text-disabled);

  --el-bg-color:                var(--color-bg-primary);
  --el-bg-color-page:           var(--color-bg-primary);
  --el-bg-color-overlay:        var(--color-bg-elevated);

  --el-fill-color:              var(--color-neutral-800);
  --el-fill-color-light:        var(--color-neutral-800);
  --el-fill-color-lighter:      var(--color-neutral-700);
  --el-fill-color-extra-light:  var(--color-neutral-700);
  --el-fill-color-blank:        var(--color-neutral-800);

  --el-border-color:            var(--color-neutral-700);
  --el-border-color-light:      var(--color-neutral-700);
  --el-border-color-lighter:    var(--color-neutral-800);
  --el-border-color-extra-light: var(--color-neutral-800);

  --el-disabled-bg-color:       var(--color-neutral-800);
  --el-disabled-text-color:     var(--color-neutral-600);
  --el-disabled-border-color:   var(--color-neutral-700);
}
```

### 与中创 `_element.scss` 补丁的协作

中创的 `_element.scss` 写的是布局细节(padding / font-size / `!important` 重写),
这些**不在**我们 skill 的覆盖范围。集成方式有两种:

**方式 A: 简单叠加(推荐)**

先按上面"中创 SCSS 资源位置"小节把 5 个 scss 文件复制到项目,然后:

```ts
// main.ts
import 'element-plus/dist/index.css'                                    // 1
import './ui-theme/tokens/theme.css'                                     // 2
import './ui-theme/overrides/zhongchuang-theme-override.css'             // 3 (CSS 变量)
import '@/styles/zhongchuang/_element.scss'                              // 4 (scss 布局补丁,放最后)
```

**方式 B: 我们的 override 注入到中创 scss 里(高级)**

如果想保证中创 scss 一定在 override 之后,可以反过来 — 把中创 scss `@use` 进我们
的 override 文件,这样编译产物里 override 一定在前:

```scss
/* overrides/zhongchuang-theme-override.scss */
@use '@/styles/zhongchuang/element';   // 中创的 _element.scss 等 4 个
@use '../tokens/theme.css';             // 我们的 token

:root {
  // 上面模板里的所有 CSS 变量映射
  --el-color-primary: var(--color-primary-500);
  // ...
}
```

**注意**: 方式 B 需要 sass 编译,产物仍是 .css。方式 A 不需要 sass 直接 import 即可。

### 验证: 中创组件全过 CSS 变量

中创的 css 变量名空间 = `--el-*` + `--cv-*` 共存。验证时**两个前缀都要查**:

```js
const elVars = [...document.styleSheets].flatMap(s => {
  try { return [...s.cssRules].filter(r => r.selectorText === ':root')
    .flatMap(r => [...r.style].filter(p => p.startsWith('--el-'))) } catch (e) { return [] }
})
const cvVars = [...document.styleSheets].flatMap(s => {
  try { return [...s.cssRules].filter(r => r.selectorText === ':root')
    .flatMap(r => [...r.style].filter(p => p.startsWith('--cv-'))) } catch (e) { return [] }
})
console.log('--el- count:', elVars.length, '— should be 50+')
console.log('--cv- count:', cvVars.length, '— should be 4+ (中创扩展)')

// 检查主色没被 EP 默认 #409EFF 卡住
const primaryBtn = document.querySelector('.el-button--primary')
console.log('primary bg:', primaryBtn ? getComputedStyle(primaryBtn).backgroundColor : 'N/A')
```

**Pass 标准**:
- `--el-` 变量 ≥ 50 个
- `--cv-` 变量 ≥ 4 个(message 4 类)
- `primary` 按钮 bg ≠ `#409EFF`

### 已知约束

- 中创的 `_element.scss` / `_tooltip.scss` 等**布局补丁**不在我们 skill 覆盖范围,
  改设计稿后这些补丁**不需要动**,它们只是尺寸/字体的细节调整。
- 中创若升级到 Element Plus 2.x+,CSS 变量体系会重构,override 模板要重新跑一遍
  Phase 0 discovery。
- 中创若改了 `--el-` 前缀为 `--zc-`(假设),则需用 [Working with Forked / Internal
  Component Libraries](#working-with-forked--internal-component-libraries) 章节的
  prefix transform 脚本批量重命名。

---

## How to Choose a Library

**This skill supports 3 libraries only**: Element Plus 2.4, Ant Design v5, and the
中创 fork. Detection is automatic (see system-prompt.md Phase 0) — the agent reads
`package.json` to identify which lib the project uses, then picks the matching
template. **Do not ask the user to pick**.

| Detected in `package.json` + scss                | Template used       | Section in this file                          |
|--------------------------------------------------|---------------------|-----------------------------------------------|
| `element-plus` + NO `--cv-` markers in scss     | Element Plus 2.4    | [§ Element Plus 2.4](#element-plus-24-vue-3)  |
| `element-plus` + `--cv-` markers in scss        | 中创 fork           | [§ 中创组件库](#中创组件库-vue-3-element-plus-fork) |
| `antd` ≥ 5.0                                     | Ant Design v5       | [§ Ant Design v5](#ant-design-v5-react)       |
| `antd` < 5.0                                     | **STOP** — ask to upgrade to v5 |
| Multiple of the above                            | **STOP** — ask user which to target |
| None of the 3 supported                          | **STOP** — message user, do not guess |

If a 4th library appears (e.g., shadcn, Naive UI, MUI, Chakra, Vuetify), **add a
new section to this file first** with the full CSS variable mapping for that lib,
then proceed. Do not silently fall back to a generic 10-stop scale.

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

### Quick Library Support Matrix (Layout)

| Library              | CSS var override for gutter   | CSS var override for breakpoints | Grid column count change |
|----------------------|-------------------------------|----------------------------------|--------------------------|
| Element Plus 2.4     | ✅ `--el-grid-gutter`          | ✅ `--el-breakpoint-*`            | ⚠️ partial (uses 24 internal) |
| 中创 fork            | ✅ inherits EP (`--el-*`)     | ✅ inherits EP (`--el-*`)         | ⚠️ partial               |
| Ant Design v5        | ✅ `--ant-grid-gutter-width`   | ✅ `--ant-screen-*`               | ⚠️ partial               |

> If the new design demands a grid column count change (12 → 24), use direct CSS
> for the breakpoint layer. The `<Col span={n}>` prop continues to work; only
> the gutter and breakpoint behavior shifts. **No other library is supported in
> v1** — see [§ Out of Scope](#out-of-scope-not-supported-in-v1).

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

## Out of Scope (NOT supported in v1)

The following are explicitly **NOT** supported in this skill's first version:

| Topic | Why excluded | Workaround |
|-------|--------------|------------|
| Renamed-prefix forks (e.g. `--el-` → `--my-`, `--dc-`, `--company-`) | Adds a 2nd mapping layer (prefix transform) per fork. Only 中创 is a known EP fork in our user base. | Add a 4th section above with the full mapping for that fork. Do not silently apply prefix transforms. |
| Naive UI, shadcn/ui, MUI, Chakra UI, Vuetify, Ant Design Vue, Bootstrap | Each has a different color structure (Naive 7-stop + JS themeOverrides, MUI 8-stop Joy, shadcn HSL, etc.). Supporting all 8 = 8x template maintenance. | If a project uses one, tell the user to either switch to a supported lib or extend this skill. |
| Element Plus 1.x (no CSS variables — SCSS-only) | EP 1.x compiles variables at SCSS time, no `--el-*` to override. | Recommend upgrading to EP 2.4+ (we test against 2.4). |
| Ant Design 4.x and earlier (no CSS variables) | AntD 4 uses `less` variables compiled at build time, no `--ant-*` to override. | Recommend upgrading to AntD v5 (we test against v5). |
| Bootstrap 4/5, Foundation, Bulma | No CSS variable system at all. | Not applicable. |

**The principle**: 3 libs done excellently > 8 libs done poorly. If a 4th lib
appears in the wild, add a new section to this file with its full mapping, then
proceed. Do not apply a "best guess" mapping to an unsupported lib.
