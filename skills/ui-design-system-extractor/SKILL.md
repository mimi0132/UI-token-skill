---
name: "ui-design-system-extractor"
description: "Extracts visual design tokens (colors, typography, spacing, radius, shadow, border, motion, grid, container, breakpoint, icon-size, focus-ring, font-family, font-weight, easing) from a design source (Figma / image / hand-written tokens), then generates a single CSS override file that maps those tokens onto an existing component library's CSS variables (Element Plus 2.4 / Ant Design v5 / 中创 fork) to re-skin the existing library without changing any component code. Invoke when user has an existing component library and wants to change only its visual style to match a new design. Supports exactly 3 libraries: Element Plus 2.4, Ant Design v5, 中创 fork (a fork of Element Plus). Other libs (shadcn, Naive UI, MUI, Chakra, Vuetify) are NOT supported — add support in override-patterns.md first."
---

# UI Design System Extractor — Theme Override

> **核心定位**: 给现有组件库做**视觉再设计** —— **保留所有交互行为,只替换视觉层**。
>
> 保留(原组件库自带,不改一行):点击 / 键盘 / 焦点 / 表单校验 / 弹窗定位 / 动画时序 / 无障碍属性 / 滚动行为 / 受控非受控逻辑 / 触屏手势。
>
> 替换或延伸(从新设计稿提取,按需扩展):**颜色 / 字体 / 字号 / 字重 / 行高 / 间距 / 圆角 / 阴影 / 边框 / 栅格 / 容器 / 断点 / 图标尺寸 / 焦点环 / 动效曲线**。
>
> **唯一输出**: `overrides/<lib>-theme-override.css` —— 导入一次,所有现有组件按新设计稿自动变脸,行为零变化。

---

## 安装 (Installation)

### 一键安装到所有 Agent (推荐)

```bash
npx skills add mimi0132/UI-token-skill --all
```

### 自定义安装

```bash
npx skills add mimi0132/UI-token-skill -a claude-code -a cursor -y
npx skills add mimi0132/UI-token-skill -g -y
```

### 不安装直接用

在任意 AI 聊天框粘贴:

```
Read https://raw.githubusercontent.com/mimi0132/UI-token-skill/main/skills/ui-design-system-extractor/SKILL.md
```

完整安装指南见 [references/installation.md](references/installation.md)

---

## 1. What This Skill Does / Does NOT

| ✅ Does | ❌ Does NOT |
|---------|-------------|
| 识别**全部**视觉变量:颜色、字体、字号、字重、行高、间距、圆角、阴影、边框、栅格、容器、断点、图标尺寸、动效曲线 | 生成新组件 |
| **推导**完整变量体系 — 7 类可推导变量(色 / 间距 / 圆角 / 字号 / 行高 / 边框 / 动效时长)用单锚点 + 数学(`color-mix` / `calc` / 几何级数)自动扩展,即使设计稿只给了 1 个值 | 改动组件的 props / API / 行为 |
| 替换 / 延伸目标库的 CSS 变量 | 修改用户已有业务代码 |
| 输出 1 个 override CSS 文件 | 改点击 / 键盘 / 焦点 / 表单 / 弹窗等交互行为 |
| 输出 7 个 token 源文件供独立调参 | 假设用户没有基础组件库 |
| 输出预览页 (用户先看效果) | 强制选某个组件库 |
| 1 行 import 集成 + 1 行暗色模式切换 | 硬编码 hex / 像素值填 40+ 个变量(用推导代替) |

**关键判断**: 用户必须已经有一个正在使用的组件库,且该组件库必须是本 skill 支持的 3 个之一(Element Plus 2.4 / Ant Design v5 / 中创 fork)。如果用户说「我项目里用的 Element Plus」 / 「公司用的是中创」 / 「React 项目用 Ant Design」,这个 skill 适用。如果用户说「我用的是 shadcn / Naive UI / MUI / Chakra / Vuetify」,**不适用**——告知用户并建议换库或扩展 skill。如果用户从零开始,这个 skill 不适用(改用通用 UI 组件生成 skill)。

### 保留 vs 替换的边界

| 类别 | 处理 | 理由 |
|------|------|------|
| 颜色 / 字体 / 圆角 / 阴影 / 边框 / 间距 | **替换** | 纯视觉,新设计稿说了算 |
| 字号 / 字重 / 行高 | **替换** | 视觉决策,影响排版节奏 |
| 栅格列数 / 容器宽度 / 断点 | **替换** | 视觉决策,影响布局结构 |
| 动效曲线 / 时长 | **替换** | 视觉风格,影响"性格" |
| 焦点环颜色 / 粗细 | **替换** | 视觉,但要保留 a11y 对比度 |
| 按钮点击 → 弹窗 → 表单校验流程 | **保留** | 交互逻辑,组件库已经验证过 |
| 键盘 Tab 顺序 / Esc 关闭 / 滚轮锁定 | **保留** | 交互逻辑,组件库已经验证过 |
| 组件 props / 受控非受控 | **保留** | API 契约,改了就破坏代码 |
| 错误提示文案 / 国际化 | **保留** | 内容层,不是视觉层 |

如果新设计稿有**原组件库不支持的视觉模式**(比如 5xl 字号、更复杂的阴影层级),Agent 应该**扩展** token(新增变量)并通过 direct CSS selector 覆盖,而不是要求用户改组件代码。

---

## 2. When to Invoke

Invoke when the user message contains ANY of:

**直接表达**:
- "我想换一套风格" / "换肤" / "改主题" / "样式切换"
- "把现在组件库的样式改成..." / "用这个设计稿改一下我现有项目的样式"
- "re-skin" / "rebrand" / "retheme" / "reskin" / "theme this to match..."

**间接表达** (需要追问现有组件库):
- 给了设计稿 + "改改颜色" / "圆角调整一下" / "换成这种风格"
- "我用的是 X 库,想套这套设计"

**输入形式**:
- Figma URL + 现有库名
- 设计稿图片 + 现有库名
- 手写 tokens + 现有库名

Do NOT invoke when:

- 用户没有现有组件库 (从零开始的 → 走别的 skill)
- 用户想新增/重构组件 (用组件生成 skill)
- 用户想"按设计稿写一个新页面" (用代码生成 skill,不是主题替换)

---

## 3. Hard Constraints (MUST)

1. **唯一核心输出**: `overrides/<lib>-theme-override.css`。
   其他文件都是辅助,**生成这 1 个文件就完成了 80% 的价值**。
2. **Override 文件必须 `@import` 7 个 token 文件**(theme/colors/typography/spacing/radius/border/motion),不能有 raw color / raw px。
3. **必须映射目标库 ALL relevant CSS variables**。只映射 `--el-color-primary` 但不映射 `--el-color-primary-light-3` 会导致 hover/active 状态颜色不一致 —— 这是最常见的失败模式。
4. **必须生成预览页** (`preview/comprehensive-preview.html` 唯一 1 个),用 iframe 或 inline HTML 展示现有组件套上
   override 后的样子。用户必须能先看效果再决定要不要集成。**禁止**生成 dashboard / user-list / preview 等其他文件。
5. **集成说明必须是一行 import**,不能多于 3 步。
6. **暗色模式必须支持**: 输出 `[data-theme="dark"]` 变体,且告诉用户怎么切换。
7. **不能生成任何新组件文件** (Button.vue / Button.tsx / 等等) —— 永远不写。

---

## 4. Workflow

### Step 0 — Auto-Detect Target Library (mandatory, no asking)

> **The user may not know the version. The user may not know the precise lib name.
> That's fine. This step auto-detects from `package.json` / `node_modules` and refuses
> to start if the project uses a lib this skill does not support.**

**Supported libraries** (3 only):

| Library              | CSS variable prefix | Color structure                              |
|----------------------|---------------------|----------------------------------------------|
| **Element Plus 2.4** | `--el-`             | 7-stop scale `base / light-3/5/7/8/9 / dark-2` |
| **Ant Design v5**    | `--ant-` + token    | 10-stop scale `1..10` (CSS-in-JS, ConfigProvider) |
| **中创 fork**         | `--el-` + `--cv-`   | Inherits EP 2.4 + `--cv-*` extensions         |

**Detection script** (run in project root, before any other work):

```bash
node -e "
const p = require('./package.json');
const all = { ...p.dependencies, ...p.devDependencies };
const out = {
  'element-plus': all['element-plus'],
  'antd':         all['antd'],
  'naive-ui':     all['naive-ui'],
  '@shadcn/ui':   all['@shadcn/ui'],
  'tailwindcss':  all['tailwindcss'],
  'vuetify':      all['vuetify'],
  '@mui/material': all['@mui/material'],
};
console.log(JSON.stringify(out, null, 2));
"
# Then check for --cv- markers to detect 中创 fork:
grep -rh -- '--cv-' node_modules/element-plus/lib/theme-chalk/*.scss 2>/dev/null | head -3
```

**Classification rules**:

| Detected                                  | Action                              |
|-------------------------------------------|-------------------------------------|
| `element-plus` + NO `cv-` markers         | Vanilla Element Plus → use § 1 template |
| `element-plus` + `cv-` markers in scss    | 中创 fork → use § 3 template         |
| `antd` ≥ 5.0                              | Ant Design v5 → use § 2 template     |
| `antd` < 5.0                              | **STOP**, ask user to upgrade to v5  |
| Multiple libs                             | **STOP**, ask user which to target   |
| None of the 3 supported                   | **STOP**, tell user what's supported |

**Then ask the user 2 things** (use AskUserQuestion):

1. **Design source**? (Figma URL / local image / hand-written tokens path)
2. **Coverage scope**?
   - Complete re-skin (colors + font + spacing + radius + shadow)
   - Colors only
   - Custom (user specifies)

**Do NOT ask which lib** — that's already auto-detected. **Do NOT ask the version** —
read it from package.json. The user is presumed to be a non-designer; technical
details are the agent's job.

### Step 1 — Extract Design Tokens

**Per-lib token structure** (use the lib template from `references/token-spec.md`):

- **EP / 中创**: 7-stop scale `base / light-3/5/7/8/9 / dark-2` per color role (primary, success, warning, danger, info, neutral).
- **AntD**: 10-stop scale `1..10` per color role, fed to `<ConfigProvider theme={{ token }}>`.

**7 token files** (same files for all libs, contents differ per lib):

| # | File | Contains |
|---|------|----------|
| 1 | `tokens/theme.css` | Entry, `@import` other 6, define component-layer tokens (`--button-bg` etc.) |
| 2 | `tokens/colors.css` | Per-role color scale (EP: 7 stops, AntD: 10 stops), semantic text/bg/border, dark mode mapping |
| 3 | `tokens/typography.css` | Font family, size (xs-4xl), weight, line height |
| 4 | `tokens/spacing.css` | 4px base (0-24) |
| 5 | `tokens/radius.css` | radius (none/sm/md/lg/xl/2xl/full) |
| 6 | `tokens/border.css` | Border width (none/default/strong/heavy), style |
| 7 | `tokens/motion.css` | Duration (instant/fast/normal/slow/slower), easing (standard/decelerate/accelerate/emphasis/spring) |

> **Note**: Grid columns/gutter, container max-width/padding, breakpoint, icon size, focus ring are **NOT** extracted to token files. They're hard-coded per-lib in `overrides/<lib>-theme-override.css` (see [references/override-patterns.md](references/override-patterns.md)), because the names differ across libs.

**Input → output mapping**:

| Design source | How to extract |
|---------------|----------------|
| Figma URL | `mcp_Figma_AI_Bridge.get_figma_data` → see [references/figma-extraction.md](references/figma-extraction.md) |
| Local image | Agent visual analysis, identify main color / font / radius / shadow, ask user to confirm |
| Hand-written tokens | `Read` file, parse directly into CSS variables |

**Extraction rules**:
- **Colors**: anchor = `base` (EP) or `6` (AntD), derive the rest with `color-mix(in srgb, var(--anchor) X%, white/black)`. Do not hand-pick 7 or 10 hex values per role.
- **Font**: size scale grows by 1.125x-1.25x. Identify body / heading / mono 3 families.
- **Radius**: identify sm / md / lg / full at minimum.
- **Shadow**: sm / md / lg / overlay at minimum.
- **Spacing**: 4px base. Round design values to multiples of 4.

### Step 2 — Generate Override File (核心)

**这是这个 skill 唯一真正"做事"的一步。**

读取 [references/override-patterns.md](references/override-patterns.md),根据用户选的库,
**完整**生成 `overrides/<lib>-theme-override.css`。

模板:

```css
/* overrides/<lib>-theme-override.css */
@import '../tokens/theme.css';

:root {
  /* ===== 颜色: 必须映射所有 primary light/dark 档 ===== */
  --<lib>-color-primary:        var(--color-primary-500);
  --<lib>-color-primary-light-3: var(--color-primary-300);
  --<lib>-color-primary-light-5: var(--color-primary-100);
  --<lib>-color-primary-light-7: var(--color-primary-50);
  --<lib>-color-primary-light-8: var(--color-primary-50);
  --<lib>-color-primary-light-9: var(--color-primary-50);
  --<lib>-color-primary-dark-2:  var(--color-primary-700);

  /* 语义色 */
  --<lib>-color-success: var(--color-success-500);
  --<lib>-color-warning: var(--color-warning-500);
  --<lib>-color-danger:  var(--color-danger-500);
  --<lib>-color-info:    var(--color-info-500);

  /* 圆角 */
  --<lib>-border-radius-base:  var(--radius-md);
  --<lib>-border-radius-small: var(--radius-sm);

  /* 字体 */
  --<lib>-font-family: var(--font-family-body);

  /* 背景 / 文字 */
  --<lib>-bg-color: var(--color-bg-primary);
  --<lib>-text-color-primary: var(--color-text-primary);
  --<lib>-text-color-regular: var(--color-text-primary);
  --<lib>-text-color-secondary: var(--color-text-secondary);
  --<lib>-text-color-placeholder: var(--color-text-disabled);
  --<lib>-border-color: var(--color-border-default);
}

[data-theme="dark"] {
  /* 暗色模式映射 - 见 references/override-patterns.md */
  --<lib>-bg-color: var(--color-bg-primary);
  --<lib>-text-color-primary: var(--color-text-primary);
  /* ... */
}
```

**反例 (禁止)**:
```css
/* ❌ 只映射了 primary, 没映射 light 档 → hover 状态颜色错乱 */
:root {
  --el-color-primary: var(--color-primary-500);
  /* 缺 --el-color-primary-light-3/5/7/8/9 和 --el-color-primary-dark-2 */
}
```

### Step 3 — Generate Preview Page

输出 **`preview/comprehensive-preview.html`(只此一个文件)**,**用 CDN 引入目标组件库**,
然后加载 override 文件,展示所有常见组件在新主题下的样子。

**严禁生成多个预览页**(不要 `dashboard-xxx.html` / `user-list-xxx.html` / `preview.html` /
`overview.html` 等)。一个项目只需要 1 个总览页,其他都是噪音。

```html
<!DOCTYPE html>
<html>
<head>
  <!-- 1. 引入目标组件库的默认样式 (CDN) -->
  <link rel="stylesheet" href="https://unpkg.com/element-plus/dist/index.css">

  <!-- 2. 引入我们的 override -->
  <link rel="stylesheet" href="../overrides/element-plus-theme-override.css">
</head>
<body>
  <!-- Element Plus 是 Vue 3 组件库, 必须有 Vue + Element Plus JS 才能渲染 -->
  <div id="app">
    <h1>Element Plus · 新主题预览</h1>

    <section>
      <h2>Button</h2>
      <el-button type="primary">Primary</el-button>
      <el-button type="success">Success</el-button>
      <el-button type="warning">Warning</el-button>
      <el-button type="danger">Danger</el-button>
      <el-button>Default</el-button>
      <el-button disabled>Disabled</el-button>
    </section>

    <section>
      <h2>Input</h2>
      <el-input placeholder="Default"></el-input>
      <el-input placeholder="Disabled" disabled></el-input>
    </section>

    <!-- 完整版本 50+ 组件 + 6 sub-block color system,
         见 references/preview-comprehensive.md -->
  </div>

  <!-- Vue 3 全局变量 (UMD 版, 用于纯 HTML demo) -->
  <script src="https://unpkg.com/vue@3"></script>
  <script src="https://unpkg.com/element-plus"></script>
  <script src="https://unpkg.com/@element-plus/icons-vue"></script>
  <script>
    const App = {
      template: document.querySelector('#app').innerHTML
    }
    const app = Vue.createApp(App)
    app.use(ElementPlus)
    for (const [name, comp] of Object.entries(ElementPlusIconsVue)) {
      app.component(name, comp)
    }
    app.mount('#app')
  </script>
</body>
</html>
```

**重要**:
- 预览页就是用户验收的地方。**必须**能用浏览器打开并看到效果。
- 结构必须清晰: 每个区块独立 `<section class="demo-block">`,带 2 位编号 + 标题 + 1 句说明 + 分割线,1 整页可滚动完。
- **真实标准**:
  - **Category 0** Color System **6 个 sub-block 必全覆盖**: Primary 10 档 + Neutral 10 档 + Semantic 4 类 10 档 + Text 5 档 + Background 4 档 + Border 3 档
  - **Categories 1-8** **50+ 组件必全覆盖**,完整清单见
    [references/preview-comprehensive.md](references/preview-comprehensive.md)
- **少一个 = 不算交付**。组件库覆盖是 preview 的核心目的,3-5 个远远不够(组件库一般 50+ 个,5 个覆盖率 < 10%)。
- 暗色模式预览可以放 toggle 按钮,默认 light。

### Step 4 — Document (README.md)

README 必须包含:

1. **集成步骤** (3 行 import, 不多于 3 步)
2. **怎么切换暗色模式** (`document.documentElement.setAttribute('data-theme', 'dark')`)
3. **怎么调整某个 token** (改 `tokens/colors.css` 里的一行,自动应用到所有组件)
4. **常见问题**:
   - "为什么我改了一个 token 没生效?" → 检查 import 顺序
   - "暗色模式不切换?" → 检查 `[data-theme]` 选择器
   - "某个组件没变化?" → 该组件可能用了硬编码颜色,不在 CSS 变量体系

### Step 5 — Verify

```bash
# 1. 7 个 token 文件存在
ls tokens/{theme,colors,typography,spacing,radius,border,motion}.css

# 2. override 文件存在且 @import token
grep -q "@import" overrides/<lib>-theme-override.css

# 3. override 文件里没有 raw hex
! grep -E "#[0-9a-fA-F]{3,8}" overrides/<lib>-theme-override.css

# 4. override 文件里没有 raw px
#    允许例外: 静态结构值 (control-height / breakpoint / container-max-width / icon-size / focus-ring-width)
#    这些必须用 raw px,不能用 var() 推导 —— 因为它们是组件库 API 硬契约
! grep -E "[0-9]+px" overrides/<lib>-theme-override.css | grep -vE "(control-height|breakpoint|container-max-width|icon-size|focus-ring-width)" || true

# 5. 预览页存在
test -f preview/comprehensive-preview.html

# 6. README 含集成 snippet
grep -q "import" README.md

# 7. 预览目录严格只有 1 个文件
PREVIEW_COUNT=$(ls preview/*.html 2>/dev/null | wc -l | tr -d ' ')
[ "$PREVIEW_COUNT" = "1" ] || { echo "preview/ 应只有 1 个文件,实际有 $PREVIEW_COUNT 个"; ls preview/; exit 1; }
```

---

## 5. Output Structure

```
<output-dir>/
├── tokens/
│   ├── theme.css                       # 入口
│   ├── colors.css                      # 颜色库
│   ├── typography.css                  # 字体
│   ├── spacing.css                     # 间距
│   ├── radius.css                      # 圆角
│   ├── border.css                      # 边框宽度 / 样式
│   └── motion.css                      # 动效时长 / 缓动曲线
├── overrides/
│   └── <lib>-theme-override.css        # ⭐ 唯一关键文件 (含 grid/breakpoint/container/icon-size/focus-ring 库内映射)
├── preview/
│   └── comprehensive-preview.html       # 验收用 (唯一 1 个预览文件)
└── README.md                           # 集成说明
```

**8 个文件。完事。**

不生成组件源码、不生成 `index.ts`、不生成 `package.json`、不生成 `dist/`、**不生成多个预览页**(只 1 个 comprehensive-preview.html)。

---

## 6. Examples

### Example 1 — Element Plus 换肤

User: "我用 Element Plus, 这是 Figma https://www.figma.com/design/xxx/MyApp, 帮我换套风格"

Action:
1. **Clarify**: 库 = Element Plus, 源 = Figma, 范围 = 完整换肤
2. **Extract**: `mcp_Figma_AI_Bridge.get_figma_data` → 7 个 token 文件
3. **Override**: 读 [references/override-patterns.md#element-plus](references/override-patterns.md#element-plus) → 生成 `element-plus-theme-override.css`,**完整映射所有 `--el-color-*` / `--el-border-*` / `--el-text-*`**
4. **Preview**: `preview/comprehensive-preview.html` 用 CDN 引 `element-plus`,展示 Category 0 色系 6 子块 + 50+ 组件
5. **Document**: README 给 3 行 import

### Example 2 — 中创组件库换肤

User: "公司用的是中创组件库(EP fork), 给了个设计稿, 改一下风格, 不要重写组件"

Action:
1. **Detect**: 读 `package.json` 发现 `element-plus` 存在; grep 项目内 scss 发现 `--cv-*` markers → 判定为 **中创 fork**
2. **Extract**: 视觉分析 → 7 个 token
3. **Override**: 中创 = EP 2.4 7-stop scale + `--cv-*` 扩展,生成 `zhongchuang-theme-override.css` (中创 section in override-patterns.md)
4. **Preview**: `preview/comprehensive-preview.html` 展示基础 EP 元素 (el-button / el-input / el-card) 套上中创 tokens 的样子
5. **Document**: README 给 3 行 import + 中创 `--cv-*` 扩展变量的处理说明

### Example 3 — 只换颜色(Ant Design v5)

User: "我项目是 React + Ant Design v5, 只想把颜色换成设计稿这套, 字体圆角保持原样"

Action:
1. **Detect**: 读 `package.json` 发现 `antd` ≥ 5.0 → 判定为 **Ant Design v5**
2. **Extract**: 只从设计稿提取颜色 token (10-stop scale 1..10),其他 token (font / radius) 沿用 AntD 默认
3. **Override**: 只生成颜色相关的 token 映射,生成 `antd-theme-override.css` + `<ConfigProvider theme={{ token }}>` 片段
4. **Preview**: 同样只展示颜色效果
5. **Document**: README 注明"只改了颜色,其他保持原样,使用 ConfigProvider 集成"

### Example 4 — 没有现有库 → 不要用这个 skill

User: "从零开始, 帮我做一套组件库"

Action: **不要用这个 skill**。告诉用户这个 skill 只做"换肤",从零开始请用别的组件生成 skill。

---

## 7. Anti-Patterns (DO NOT)

- ❌ **不生成任何新组件文件** (Button.vue, Button.tsx, etc.) —— 永远不写
- ❌ 不修改用户现有代码
- ❌ 不假设用户的组件库 —— 必须先问
- ❌ 不在 override 文件里写 raw color / raw px
- ❌ 不"只映射一部分" CSS 变量 —— 视觉会不一致
- ❌ 不跳过预览页
- ❌ 不让集成步骤超过 3 行 import
- ❌ 不做"半套" (例如只覆盖 primary 颜色不覆盖 neutral)
- ❌ 不假设有暗色模式 —— 显式询问用户是否需要

---

## 8. References

按需加载:

- [references/installation.md](references/installation.md) — 按 Agent 详细安装
- [references/system-prompt.md](references/system-prompt.md) — 提取 checklist + 质量门
- [references/token-spec.md](references/token-spec.md) — 7 个 token 文件的详细规范
- [references/override-patterns.md](references/override-patterns.md) — **各 UI 库的 CSS 变量映射表 (核心)**
- [references/figma-extraction.md](references/figma-extraction.md) — Figma MCP 用法

---

## 9. Supported Agents

| Agent       | Install Location            | Notes                              |
|-------------|-----------------------------|------------------------------------|
| Trae AI     | `.trae/skills/<name>/`      | Native Trae skills directory       |
| Cursor      | `.cursor/skills/<name>/`    | Detected by `npx skills`           |
| Claude Code | `.claude/skills/<name>/`    | Detected by `npx skills`           |
| Codex       | `.codex/skills/<name>/`     | Detected by `npx skills`           |
| GitHub Copilot | `.copilot/skills/<name>/` | Detected by `npx skills`           |
| Cline       | `.cline/skills/<name>/`     | Detected by `npx skills`           |
| Roo Code    | `.roo/skills/<name>/`       | Detected by `npx skills`           |

任何能读文件、写文件、分析图片的 Agent 都能用,即便是用粘贴 SKILL.md 方式。
