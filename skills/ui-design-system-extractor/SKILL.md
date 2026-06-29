---
name: "ui-design-system-extractor"
description: "Extracts visual design tokens (colors, typography, spacing, radius, shadow) from a design source (Figma / image / hand-written tokens), then generates a single CSS override file that maps those tokens onto an existing component library's CSS variables (Element Plus / Ant Design / Naive UI / shadcn / MUI / Chakra) to re-skin the existing library without changing any component code. Invoke when user has an existing component library and wants to change only its visual style to match a new design."
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
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor --all
```

### 自定义安装

```bash
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor -a claude-code -a cursor -y
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor -g -y
```

### 不安装直接用

在任意 AI 聊天框粘贴:

```
Read https://raw.githubusercontent.com/YOUR_GITHUB_USER/ui-design-system-extractor/main/skills/ui-design-system-extractor/SKILL.md
```

完整安装指南见 [references/installation.md](references/installation.md)

---

## 1. What This Skill Does / Does NOT

| ✅ Does | ❌ Does NOT |
|---------|-------------|
| 识别**全部**视觉变量:颜色、字体、字号、字重、行高、间距、圆角、阴影、边框、栅格、容器、断点、图标尺寸、动效曲线 | 生成新组件 |
| **推导**完整变量体系 — 6 类可推导变量(色 / 间距 / 圆角 / 字号 / 行高 / 边框 / 动效时长)用单锚点 + 数学(`color-mix` / `calc` / 几何级数)自动扩展,即使设计稿只给了 1 个值 | 改动组件的 props / API / 行为 |
| 替换 / 延伸目标库的 CSS 变量 | 修改用户已有业务代码 |
| 输出 1 个 override CSS 文件 | 改点击 / 键盘 / 焦点 / 表单 / 弹窗等交互行为 |
| 输出 7 个 token 源文件供独立调参 | 假设用户没有基础组件库 |
| 输出预览页 (用户先看效果) | 强制选某个组件库 |
| 1 行 import 集成 + 1 行暗色模式切换 | 硬编码 hex / 像素值填 40+ 个变量(用推导代替) |

**关键判断**: 用户必须已经有一个正在使用的组件库。如果用户说「我项目里用的 Element Plus」 / 「我装的是 shadcn」 / 「基于 Ant Design」,这个 skill 适用。如果用户从零开始,这个 skill 不适用(改用通用 UI 组件生成 skill)。

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
2. **Override 文件必须 `@import` 5 个 token 文件**,不能有 raw color / raw px。
3. **必须映射目标库 ALL relevant CSS variables**。只映射 `--el-color-primary` 但不映射 `--el-color-primary-light-3` 会导致 hover/active 状态颜色不一致 —— 这是最常见的失败模式。
4. **必须生成预览页** (`preview.html`),用 iframe 或 inline HTML 展示现有组件套上
   override 后的样子。用户必须能先看效果再决定要不要集成。
5. **集成说明必须是一行 import**,不能多于 3 步。
6. **暗色模式必须支持**: 输出 `[data-theme="dark"]` 变体,且告诉用户怎么切换。
7. **不能生成任何新组件文件** (Button.vue / Button.tsx / 等等) —— 永远不写。

---

## 4. Workflow

### Step 0 — Clarify (CHECKPOINT, 必问)

问用户 **3 件事** (用 AskUserQuestion 一次问完):

1. **现有组件库**? (Element Plus / Ant Design / shadcn / Naive UI / MUI / Chakra / 其他)
2. **设计源**? (Figma URL / 本地图片 / 手写 tokens 路径)
3. **覆盖范围**? 
   - 完整换肤 (颜色 + 字体 + 间距 + 圆角 + 阴影)
   - 只换颜色
   - 自定义 (用户指定)

如果用户没说现有库,**必须先问**,不要假设。

### Step 1 — Extract Design Tokens

从设计源提取 5 类 token,输出到 `tokens/` 目录的 5 个独立文件。

完整 token 规范见 [references/token-spec.md](references/token-spec.md)。

**5 个文件固定顺序与职责**:

| # | 文件 | 包含 |
|---|------|------|
| 1 | `tokens/theme.css` | 入口,`@import` 其他 4 个,定义组件层 token (`--button-bg` 等) |
| 2 | `tokens/colors.css` | primary/neutral/semantic 色阶 (50-900),语义色,暗色映射 |
| 3 | `tokens/typography.css` | 字体家族、字号 (xs-4xl)、字重、行高、字间距 |
| 4 | `tokens/spacing.css` | 4px 基准间距 (0-24),语义间距 |
| 5 | `tokens/radius.css` | 圆角 (none/sm/md/lg/xl/2xl/full) |

**输入 → 输出映射**:

| 设计源 | 怎么提取 |
|--------|---------|
| Figma URL | `mcp_Figma_AI_Bridge.get_figma_data` → 见 [references/figma-extraction.md](references/figma-extraction.md) |
| 本地图片 | Agent 视觉分析,识别主色/字体/圆角/阴影,询问用户确认 |
| 手写 tokens | `Read` 文件,直接解析为 CSS 变量 |

**提取规则**:
- **颜色**: 主色 (primary) 必须有完整 50-900 色阶。可以用 `color-mix(in srgb, var(--color-primary-500) X%, white/black)` 推导其他档。
- **字体**: 字号梯度按 1.125x-1.25x 几何增长。识别 body / heading / mono 三族。
- **圆角**: 至少识别 sm / md / lg / full 四档。
- **阴影**: 至少 sm / md / lg / overlay 四档。
- **间距**: 4px 基准。如果设计稿不规则,按 4 对齐。

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

输出 `preview/preview.html`,**用 CDN 引入目标组件库**,然后加载 override 文件,
展示所有常见组件在新主题下的样子:

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
  <h1>Element Plus · 新主题预览</h1>

  <section>
    <h2>颜色</h2>
    <div class="swatch primary-500">Primary 500</div>
    <div class="swatch primary-100">Primary 100</div>
    <!-- ... -->
  </section>

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

  <!-- ... 至少展示 5 个常见组件, 覆盖 default / hover / active / disabled 状态 -->
</body>
</html>
```

**重要**:
- 预览页就是用户验收的地方。**必须**能用浏览器打开并看到效果。
- 至少展示: Button / Input / Tag / Card / Dialog (Modal) 5 个组件。
  - **真实标准**: 8 大类 **50+ 组件必须全覆盖**,完整清单见
    [references/preview-comprehensive.md](references/preview-comprehensive.md)。
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
# 1. 5 个 token 文件存在
ls tokens/{theme,colors,typography,spacing,radius}.css

# 2. override 文件存在且 @import token
grep -q "@import" overrides/<lib>-theme-override.css

# 3. override 文件里没有 raw hex
! grep -E "#[0-9a-fA-F]{3,8}" overrides/<lib>-theme-override.css

# 4. override 文件里没有 raw px
! grep -E "[0-9]+px" overrides/<lib>-theme-override.css

# 5. 预览页存在
test -f preview/preview.html

# 6. README 含集成 snippet
grep -q "import" README.md
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
│   └── radius.css                      # 圆角
├── overrides/
│   └── <lib>-theme-override.css        # ⭐ 唯一关键文件
├── preview/
│   └── preview.html                    # 验收用
└── README.md                           # 集成说明
```

**8 个文件。完事。**

不生成组件源码、不生成 `index.ts`、不生成 `package.json`、不生成 `dist/`。

---

## 6. Examples

### Example 1 — Element Plus 换肤

User: "我用 Element Plus, 这是 Figma https://www.figma.com/design/xxx/MyApp, 帮我换套风格"

Action:
1. **Clarify**: 库 = Element Plus, 源 = Figma, 范围 = 完整换肤
2. **Extract**: `mcp_Figma_AI_Bridge.get_figma_data` → 5 个 token 文件
3. **Override**: 读 [references/override-patterns.md#element-plus](references/override-patterns.md#element-plus) → 生成 `element-plus-theme-override.css`,**完整映射所有 `--el-color-*` / `--el-border-*` / `--el-text-*`**
4. **Preview**: `preview.html` 用 CDN 引 `element-plus`,展示 Button/Input/Tag/Card/Dialog
5. **Document**: README 给 3 行 import

### Example 2 — shadcn/ui 换肤

User: "项目里是 shadcn/ui + Tailwind, 给了个设计稿, 改一下风格, 不要重写组件"

Action:
1. **Clarify**: 库 = shadcn/ui, 源 = 图片
2. **Extract**: 视觉分析 → 5 个 token
3. **Override**: shadcn 用 CSS 变量 (`--background` / `--foreground` / `--primary` 等),生成
   `shadcn-theme-override.css` + `tailwind.config.js` 调整片段
4. **Preview**: shadcn 没有 CDN,但可以展示基础 HTML 元素 (button/input/card) 套上 shadcn tokens 的样子
5. **Document**: README 解释 shadcn 的 `tailwind.config.js` 集成方式

### Example 3 — 只换颜色

User: "我用的是 Naive UI, 只想把颜色换成设计稿这套, 字体圆角保持原样"

Action:
1. **Clarify**: 库 = Naive UI, 范围 = **只换颜色**
2. **Extract**: 只从设计稿提取颜色,其他 token 沿用 Naive UI 默认
3. **Override**: 只生成颜色相关的 CSS 变量映射,跳过字体/圆角
4. **Preview**: 同样只展示颜色效果
5. **Document**: README 注明"只改了颜色,其他保持原样"

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
- [references/token-spec.md](references/token-spec.md) — 5 个 token 文件的详细规范
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
