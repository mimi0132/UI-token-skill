# UI Design System Extractor

**从设计稿给现有组件库换皮肤 —— 保留交互,只换视觉。**

项目用的是 Element Plus / Ant Design / Naive UI / shadcn / MUI / Chakra 等,拿到一份新设计稿,你想让所有组件**看起来像新设计**(颜色、字体、间距、栅格、圆角、阴影、边框、动效全部跟着走),但**点击 / 键盘 / 焦点 / 表单 / 弹窗 / 无障碍 / 滚动 / 动画时序一行不动**?用这个 skill。

**只 import 一个 CSS 文件**,搞定。

---

## 安装

```bash
npx skills add mimi0132/UI-token-skill --all
```

`--all` 让 CLI 自动检测并装到你机器上所有 AI Agent(Cursor / Claude Code / Trae / Codex / Copilot / Cline / Roo),**不弹 TUI 选择框**。

---

## 用法

把设计稿(Figma 链接 / 本地图片 / tokens JSON)和你在用的组件库告诉你的 AI Agent,直接说就行:

```
我的项目是 Vue 3 + Element Plus
这是设计稿: https://www.figma.com/design/xxx/MyApp
按这个设计稿覆盖所有视觉(颜色、字体、间距、栅格、圆角、阴影、动效),保留所有交互
```

Agent 会:
1. 提取 15 个维度的视觉变量
2. 生成 7 个 token 源文件 + 1 个 override 文件
3. 给一份预览页让你在浏览器里看效果
4. 给集成说明

集成后,你项目里的 `<el-button>` `<a-button>` `<NButton>` `<Button>` 等组件代码一行不动,刷新就变脸。

---

## 支持的组件库

- **Vue**: Element Plus · Ant Design Vue · Naive UI · Vuetify
- **React**: Ant Design · shadcn/ui · MUI · Chakra UI
- **其它**: fork 自这些库的公司内部组件库也支持,只要走 CSS 变量

---

## 详细文档

- [SKILL.md](skills/ui-design-system-extractor/SKILL.md) — Agent 加载的完整指令、替换/保留边界、调参指南、FAQ
- [references/installation.md](skills/ui-design-system-extractor/references/installation.md) — 按 Agent 分别安装
- [references/token-spec.md](skills/ui-design-system-extractor/references/token-spec.md) — 7 个 token 文件的规范与推导算法
- [references/override-patterns.md](skills/ui-design-system-extractor/references/override-patterns.md) — 各组件库的 CSS 变量映射表
- [references/preview-comprehensive.md](skills/ui-design-system-extractor/references/preview-comprehensive.md) — 预览页 8 大类 50+ 组件清单

---

## 许可

MIT
