# UI Design System Extractor

> **从设计稿给现有组件库换皮肤 —— 保留交互,只换视觉。**

项目用的是 **Element Plus / Ant Design v5 / 中创 fork (Element Plus 内部 fork)**,拿到一份新设计稿,你想让所有组件**看起来像新设计**(颜色、字体、间距、栅格、圆角、阴影、边框、动效全部跟着走),但**点击 / 键盘 / 焦点 / 表单 / 弹窗 / 无障碍 / 滚动 / 动画时序一行不动**?用这个 skill。

**只 import 一个 CSS 文件**,搞定。

---

## ✨ 核心特性

- 🎨 **15 维度提取** — 颜色、字体、字号、字重、行高、间距、圆角、阴影、边框、栅格、容器、断点、图标尺寸、动效曲线、焦点环
- 🔄 **变量自动推导** — 单锚点 + `color-mix` / `calc` / 几何级数,设计稿只给 1 个值也能生成完整色阶
- 🧩 **3 个组件库** — Element Plus 2.4 · Ant Design v5 · 中创 fork
-  **交互零侵入** — 组件库原有的点击 / 键盘 / 焦点 / 表单 / 弹窗 / a11y / API 一行代码不动
- 📦 **零依赖** — 不需要 Tailwind / 预处理器

---

## 🚀 安装

```bash
npx skills add mimi0132/UI-token-skill --all
```

`--all` 让 CLI 自动检测并装到所有 AI Agent(Cursor / Claude Code / Trae / Codex / Copilot / Cline / Roo),**不弹 TUI 选择框**。

---

## 📖 用法

把设计稿(Figma 链接 / 本地图片 / tokens JSON)和在用的组件库告诉 AI Agent,例如:

> 我的项目是 Vue 3 + Element Plus,这是设计稿:https://figma.com/.../MyApp,按这个设计稿覆盖所有视觉,保留所有交互

Agent 会:
1. 自动检测项目里的组件库
2. 提取 15 维度变量 → 7 个 token 文件 (`tokens/*.css`)
3. 映射到目标库 CSS 变量 → 1 个 override (`overrides/<lib>-theme-override.css`) ⭐
4. 生成 1 个预览页让你先看效果 (`preview/comprehensive-preview.html`)

集成后,你项目里的 `<el-button>` 等组件代码**一行不动**,刷新就变脸。详细集成见下一节。

---

## 🔄 集成示例

### 1. 引入主题文件

> **约定**: 所有主题文件放在项目根 `ui-theme/` 目录下,包含 `tokens/` (设计令牌) 和 `overrides/` (覆盖文件) 两个子目录。
> ```
> project-root/
> ├── ui-theme/
> │   ├── tokens/
> │   │   ├── theme.css          # 入口(@import 其余 6 个)
> │   │   ├── colors.css
> │   │   ├── typography.css
> │   │   ├── spacing.css
> │   │   ├── radius.css
> │   │   ├── border.css
> │   │   └── motion.css
> │   └── overrides/
> │       ├── element-plus-theme-override.css    # 或 antd / zhongchuang
> │       └── zhongchuang-theme-override.css
> ├── src/
> │   └── styles/zhongchuang/     # 中创 scss 布局补丁 (仅中创需要,见 resources/zhongchuang/)
> └── main.js
> ```

#### 1.1 Vue 3 + Element Plus

```js
// main.js
import 'element-plus/dist/index.css'                       // 1. 库 CSS
import './ui-theme/tokens/theme.css'                       // 2. ⭐ token 入口(自动 @import 其余 6 个)
import './ui-theme/overrides/element-plus-theme-override.css'  // 3. ⭐ 覆盖(把 --el-* 映射到 --color-*)
```

#### 1.2 Vue 3 + 中创组件库

```js
// main.js
import 'element-plus/dist/index.css'                          // 1. 库 CSS
import './ui-theme/tokens/theme.css'                          // 2. ⭐ token 入口
import './ui-theme/overrides/zhongchuang-theme-override.css'  // 3. ⭐ override (--el-* + --cv-*)
import '@/styles/zhongchuang/_element.scss'                  // 4. 中创 scss 布局补丁
                                                            //    (从 skill 的 resources/zhongchuang/ 复制)
```

#### 1.3 React + Ant Design v5

Ant Design v5 的 token 是通过 `<ConfigProvider>` 在 JS 里注入(不靠覆盖 CSS),所以用法稍有不同:

```tsx
// App.tsx
import { ConfigProvider, theme } from 'antd'
import tokenImport from './ui-theme/tokens/theme.generated.json'  // 我们生成的 token JSON
                                                          // (或直接写死从 .css 转的 token 对象)

// 方式 A:用 ConfigProvider(运行时切换)
// 方式 B:用 static 方法覆盖(antd v5.4+):
//   import { theme } from 'antd'
//   import { ThemeConfig } from 'antd/es/config-provider/context'
//   自行处理

const customTheme = {
  token: {
    colorPrimary: 'var(--color-primary-500)',
    colorSuccess: 'var(--color-success-500)',
    // ... 映射 antd token 到我们 token
  },
}

export default function App() {
  return (
    <ConfigProvider theme={customTheme}>
      <YourApp />
    </ConfigProvider>
  )
}
```

> **重要**: AntD v5 的 token 系统在 JS 侧,不靠 CSS 变量覆盖。SKILL.md Step 2 会输出 `token.generated.json` 给 `ConfigProvider` 用。

### 2. 改 token(全组件跟着变)

业务**永远不直接写 hex / px**,全部用 `var(--xxx)`。改设计只动 `tokens/` 目录:

| 想改什么 | 改哪个文件 | 改完影响范围 |
|----------|------------|---------------|
| 改主色 / 辅色 / 状态色 / 文字 / 背景 | `tokens/colors.css` | 所有用该色的组件 + 派生色阶自动重算 |
| 改字体 / 字号 / 字重 / 行高 | `tokens/typography.css` | 全局文字 |
| 改间距步进(4px → 8px) | `tokens/spacing.css` | 全局间距 / padding / margin |
| 改圆角档位 | `tokens/radius.css` | 按钮 / 输入框 / 卡片 / 弹窗 |
| 改边框粗细 / 样式 | `tokens/border.css` | 边框 / 分割线 |
| 改动效曲线 / 时长 | `tokens/motion.css` | 过渡 / 弹窗 / tooltip |
| 改组件层语义映射(`--button-bg` 用哪个色) | `tokens/theme.css` | 组件层 token |

```css
/* tokens/colors.css — 改主色 */
:root {
  --color-primary-500: #FF6B35;  /* 改这里,所有用 primary 的组件全部跟着变 */
  /* light-3 / light-5 / light-7 / dark-2 由 color-mix() 自动推导,不用手改 */
}
```

### 3. 业务代码里怎么写

```css
/* 推荐:用 token,不用 hex */
.my-card {
  background: var(--color-bg-container);
  color: var(--color-text-primary);
  padding: var(--space-4);
  border-radius: var(--radius-md);
  border: 1px solid var(--color-border-base);
  transition: all var(--motion-duration-normal) var(--motion-easing-standard);
}

/* 禁止:硬编码 hex / px */
.my-card {
  background: #ffffff;     /* ❌ */
  color: #333333;          /* ❌ */
  padding: 16px;           /* ❌ */
  border-radius: 8px;      /* ❌ */
  transition: all 200ms ease;  /* ❌ */
}
```

```tsx
// React 组件里
<Button style={{
  background: 'var(--color-primary-500)',
  padding: 'var(--space-2) var(--space-4)',
  borderRadius: 'var(--radius-md)',
}}>按钮</Button>
```

### 4. 常见问题

**Q: 改一个 token 没生效?**
A: 检查 import 顺序 — 库 CSS → `tokens/theme.css` → `overrides/<lib>-theme-override.css`。override **必须在最后**(覆盖 CSS 变量需要后置声明)。

**Q: hover / active 状态颜色不对?**
A: 大概率是只覆盖了 `--el-color-primary` 没覆盖 `light-3/5/7/8/9 / dark-2`。这几个都映射,hover / active 才正常。

**Q: 某个组件完全没变?**
A: 该组件可能用了硬编码颜色(在 JS 里 `style={{ color: '#fff' }}`),不在 CSS 变量体系里。要么改组件源码(不推荐),要么用更高优先级 selector 覆盖。

**Q: 我用的是 Element Plus 的暗黑 / Fluid / 其他主题?**
A: 都支持,override 文件在它们之后加载就能覆盖。

**Q: 业务代码里可以混用 Element Plus 组件和原生 HTML 吗?**
A: 可以。原生 HTML 用 `var(--space-4)` 等 token,跟组件库走同一套变量,视觉一致。

---

## 📂 项目结构

- `SKILL.md` — Agent 加载的入口(完整工作流 + 边界 + FAQ)
- `references/` — 6 个细节文档:安装、token 规范、各库 CSS 变量映射、预览页规范、提取 checklist、Figma MCP 用法
- `resources/zhongchuang/` — 中创组件库的 5 个 scss 布局补丁(集成时复制到项目里)

---

## ✅ 支持的组件库

| 组件库 | 前缀 | 颜色结构 | 适用项目 |
|--------|------|----------|----------|
| **Element Plus 2.4** | `--el-*` | 7 档 (`base / light-3/5/7/8/9 / dark-2`) | Vue 3 |
| **中创 fork** | `--el-*` + `--cv-*` | 继承 EP 2.4 + `--cv-*` 扩展 | Vue 3 (公司内部项目) |
| **Ant Design v5** | `--ant-*` + `ConfigProvider` | 10 档 (`1..10`) | React |

> 其他组件库 (shadcn / Naive UI / MUI / Chakra / Vuetify / Ant Design Vue / Bootstrap / EP 1.x / AntD 4.x) **不在支持范围**。如需扩展,在 `references/override-patterns.md` 新增对应章节即可。

**中创组件库**: 中创是基于 **Element Plus 的内部 fork**。
- 沿用 EP 的 `--el-*` CSS 变量前缀(没改)
- 扩展了 `--cv-message-success/warning/error/info-border-color` 等 `--cv-*` 自定义变量
- 配套 5 个 scss 布局补丁(`_element.scss` / `_avatar.scss` / `_message.scss` /
  `_popconfirm.scss` / `_tooltip.scss`),做 EP 没做的 padding / font-size 细节重写

集成方式 = 4 步 import: 库 CSS → 我们 token → 我们的 override (改 `--el-*` + `--cv-*`) → 中创 scss 补丁。

---

## ⚠️ 不做什么

为了保持 skill 单一职责,这个 skill **明确不做**:

- ❌ 不生成新组件文件(Button.vue / Button.tsx 等)
- ❌ 不改组件的 props / API / 行为
- ❌ 不从零开始做一套组件库(改用别的组件生成 skill)
- ❌ 不改用户已有业务代码
- ❌ 不假设你用某个组件库(必须先问)

---

## 🤔 FAQ

集成相关的常见问题见 [🔄 集成示例 §5](#5-常见问题)。完整 FAQ 在 [SKILL.md](skills/ui-design-system-extractor/SKILL.md) 里。

不在支持的库(Naive UI / shadcn / MUI / Chakra / Vuetify / Ant Design Vue / Bootstrap / EP 1.x / AntD 4.x)需要在 [references/override-patterns.md](skills/ui-design-system-extractor/references/override-patterns.md) 新增对应章节,提交 PR。

---

## 📚 详细文档

- [SKILL.md](skills/ui-design-system-extractor/SKILL.md) — Agent 加载的完整指令、替换/保留边界、调参指南、FAQ
- [references/installation.md](skills/ui-design-system-extractor/references/installation.md) — 按 Agent 分别安装
- [references/token-spec.md](skills/ui-design-system-extractor/references/token-spec.md) — 7 个 token 文件的规范与推导算法
- [references/override-patterns.md](skills/ui-design-system-extractor/references/override-patterns.md) — 各组件库的 CSS 变量映射表
- [references/preview-comprehensive.md](skills/ui-design-system-extractor/references/preview-comprehensive.md) — 预览页结构规范 + Color System 6 子块 + 8 大类 50+ 组件清单

---

## 🤝 贡献

欢迎 PR 和 issue:
- 新的组件库 CSS 变量映射表 → `references/override-patterns.md`
- 新的推导算法(比如其他色彩空间) → `references/token-spec.md`
- Bug 报告或功能建议 → issue

---

## 📜 许可

MIT
