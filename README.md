# UI Design System Extractor

> **从设计稿给现有组件库换皮肤 —— 保留交互,只换视觉。**

项目用的是 Element Plus / Ant Design / Naive UI / shadcn / MUI / Chakra / Vuetify / 中创 fork 等,拿到一份新设计稿,你想让所有组件**看起来像新设计**(颜色、字体、间距、栅格、圆角、阴影、边框、动效全部跟着走),但**点击 / 键盘 / 焦点 / 表单 / 弹窗 / 无障碍 / 滚动 / 动画时序一行不动**?用这个 skill。

**只 import 一个 CSS 文件**,搞定。

---

## ✨ 核心特性

- 🎨 **15 维度提取** — 颜色、字体、字号、字重、行高、间距、圆角、阴影、边框、栅格、容器、断点、图标尺寸、动效曲线、焦点环
- 🔄 **变量自动推导** — 单锚点 + `color-mix` / `calc` / 几何级数,即使设计稿只给 1 个值也能生成完整 10 档色阶 + 4 档间距 + 4 档圆角
- 🧩 **8 大组件库** — Element Plus · Ant Design · Ant Design Vue · Naive UI · shadcn · MUI · Chakra · Vuetify
- 🏢 **公司内部 fork 也支持** — 中创组件库 (Element Plus fork) 等,沿用 EP 前缀并扩展 `--cv-*` 自定义变量
- 🌓 **暗色模式** — 一行 `data-theme` 切换,色阶自动反转
- 🎯 **交互零侵入** — 组件库原有的点击 / 键盘 / 焦点 / 表单 / 弹窗 / 滚动 / a11y / API / 受控逻辑 一行代码不动
- 📦 **零依赖** — 不需要 Tailwind / 预处理器 / 构建工具
- 🖼 **预览页 50+ 组件全覆盖** — Color System 6 子块 (Primary 10 档 / Neutral 10 档 / Semantic 4 类 / Text 5 档 / Bg 4 档 / Border 3 档) + 8 大类组件 (Buttons / Form / Data Display / Navigation / Feedback / Layout / Advanced / Misc) 全验证,改一个 token 立即看到所有组件变化

---

## 🚀 安装

```bash
npx skills add mimi0132/UI-token-skill --all
```

`--all` 让 CLI 自动检测并装到你机器上所有 AI Agent(Cursor / Claude Code / Trae / Codex / Copilot / Cline / Roo),**不弹 TUI 选择框**。

需要手动安装到指定目录或本机无网络,看 [references/installation.md](skills/ui-design-system-extractor/references/installation.md)。

---

## 📖 用法

把设计稿(Figma 链接 / 本地图片 / tokens JSON)和你在用的组件库告诉你的 AI Agent:

```
我的项目是 Vue 3 + Element Plus
这是设计稿: https://www.figma.com/design/xxx/MyApp
按这个设计稿覆盖所有视觉(颜色、字体、间距、栅格、圆角、阴影、动效),保留所有交互
```

Agent 会:

| 步骤 | 做什么 | 输出 |
|------|--------|------|
| 1 | 提取 15 维度视觉变量 | 7 个 token 源文件 (`tokens/*.css`) |
| 2 | 映射到目标库的 CSS 变量 | `overrides/<lib>-theme-override.css` ⭐ |
| 3 | 生成 1 个总览预览页 (含 Color System 6 子块 + 50+ 组件 demo) | `preview/comprehensive-preview.html` |
| 4 | 给集成说明 | `README.md` |

集成后,你项目里的 `<el-button>` `<a-button>` `<NButton>` `<Button>` 等组件代码**一行不动**,刷新就变脸。

---

## 🔄 集成示例

### 1. 引入主题文件

```js
// main.js (Vue 3 + Element Plus)
import 'element-plus/dist/index.css'              // 库 CSS
import './overrides/element-plus-theme-override.css'  // ⭐ 主题覆盖
```

```js
// main.js (Vue 3 + 中创组件库)
import 'element-plus/dist/index.css'                     // 1. 库 CSS
import './overrides/zhongchuang-theme-override.css'      // 2. ⭐ override
import 'zhongchuang/dist/index.scss'                     // 3. 中创 scss 补丁
```

### 2. 切暗色模式

```js
// 一行切换
document.documentElement.setAttribute('data-theme', 'dark')
```

### 3. 调单个 token(全组件跟着变)

```css
/* tokens/colors.css */
:root {
  --color-primary-500: #FF6B35;  /* 改这里,所有用 primary 的组件全部跟着变 */
}
```

---

## 📂 项目结构

```
UI-token-skill/
├── skills/
│   └── ui-design-system-extractor/
│       ├── SKILL.md                       # Agent 加载的入口
│       └── references/                    # 按需加载的细节
│           ├── installation.md            # 各 Agent 安装路径
│           ├── token-spec.md              # 7 个 token 文件规范 + 推导算法
│           ├── override-patterns.md       # 各组件库 CSS 变量映射
│           ├── preview-comprehensive.md   # 预览页 8 大类 50+ 组件清单
│           ├── system-prompt.md           # 提取 checklist + 质量门
│           └── figma-extraction.md        # Figma MCP 用法
├── README.md                              # 本文件
├── package.json
└── LICENSE
```

---

## ✅ 支持的组件库

- **Vue**: Element Plus · Ant Design Vue · Naive UI · Vuetify
- **React**: Ant Design · shadcn/ui · MUI · Chakra UI
- **公司内部 fork**: 中创组件库 (Element Plus fork) · 其他 `--el-*` / `--ant-*` 前缀的内部库
- **其它**: fork 自这些库的公司内部组件库也支持,只要走 CSS 变量

> **中创组件库特别说明**: 中创沿用 EP 的 `--el-` 前缀,并扩展了 `--cv-message-*` 等
> 自定义变量,我们的 override 模板 + 集成方式见
> [override-patterns.md#中创组件库](skills/ui-design-system-extractor/references/override-patterns.md#中创组件库-vue-3-element-plus-fork)

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

**改一个 token 没生效?**
检查 import 顺序 — 库 CSS → tokens/theme.css → overrides/。override 必须在最后。

**预览页只有 1 个文件但我有 dashboard-xxx.html?**
那是之前 Agent 跑流程时留下的冗余,删掉即可。`rm preview/dashboard-*.html preview/user-list-*.html preview/preview.html` 然后重新跑一次,只保留 `comprehensive-preview.html`。

**暗色模式不切换?**
检查 `[data-theme="dark"]` 选择器在你的 override 文件里有没有定义。参考 [override-patterns.md](skills/ui-design-system-extractor/references/override-patterns.md)。

**某个组件没变化?**
该组件可能用了硬编码颜色,不在 CSS 变量体系里。看 SKILL.md 的"Anti-Patterns"。

**我用的是 Element Plus 的暗黑 / Fluid / 其他主题?**
都支持,override 文件在它们之后加载就能覆盖。Element Plus 默认用 `--el-color-primary`,hover 状态用 `--el-color-primary-light-3` 等,override 文件会一并映射。

**我用的是中创组件库 (Element Plus fork)?**
直接告诉 Agent "组件库是中创" 即可,会用 `overrides/zhongchuang-theme-override.css` 模板:
- `--el-*` 全套(中创沿用 EP 前缀)
- `--cv-message-success/warning/error/info-border-color` 等中创扩展
- 中创自家的 `_element.scss` 布局补丁在 override 之后 import,两者互补

**我用的是其他公司 fork 库 (前缀 `--my-` / `--ant-xxx-`)?**
见 [override-patterns.md#working-with-forked](skills/ui-design-system-extractor/references/override-patterns.md#working-with-forked--internal-component-libraries),
用 prefix transform 脚本批量重命名模板。

更多问题看 [SKILL.md#faq](skills/ui-design-system-extractor/SKILL.md)。

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
