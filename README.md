# UI Design System Extractor

**从设计稿给现有组件库换皮肤 —— 保留交互,只换视觉。**

项目已经在用 Element Plus / Ant Design / Naive UI / shadcn / MUI,拿到一份新设计稿后,你想让它**看起来像新设计**(颜色、字体、间距、栅格、圆角、阴影、动效全跟着新设计走),但**所有点击 / 键盘 / 弹窗 / 表单 / a11y 一行不动**?用这个 skill。

只输出 1 个 CSS 文件,import 一次,搞定。

---

## 安装

```bash
npx skills add mimi0132/UI-token-skill --all
```

`--all` 让 CLI 自动检测并装到你机器上所有 AI Agent(Cursor / Claude Code / Trae / Codex / Copilot / Cline / Roo 等),**不弹 TUI 选择框**。

---

## 用

把设计稿(Figma 链接 / 本地图片 / tokens JSON)和你在用的组件库告诉你的 AI Agent,直接说就行:

```
我的项目是 Vue 3 + Element Plus
这是设计稿: https://www.figma.com/design/xxx/MyApp
按这个设计稿重新做所有视觉(颜色、字体、间距、栅格、圆角、阴影、动效),保留所有交互
```

Agent 会:
1. 提取全维度视觉变量
2. 输出 `overrides/<lib>-theme-override.css`
3. 给一份预览页让你在浏览器里看效果
4. 给集成说明

**你项目里 `<el-button>` / `<Button>` 的代码一行不动,刷完页面就变脸。**

---

## 支持的组件库

Element Plus · Ant Design · Ant Design Vue · Naive UI · shadcn/ui · MUI · Chakra UI · Vuetify

(Fork 自这些库的公司内部组件库也支持,见 [SKILL.md](skills/ui-design-system-extractor/SKILL.md))

---

## 详细文档

- [SKILL.md](skills/ui-design-system-extractor/SKILL.md) — Agent 加载的完整指令、替换/保留边界、调参指南、FAQ
- [references/installation.md](skills/ui-design-system-extractor/references/installation.md) — 按 Agent 分别安装
- [references/override-patterns.md](skills/ui-design-system-extractor/references/override-patterns.md) — 各组件库的 CSS 变量映射表
- [references/token-spec.md](skills/ui-design-system-extractor/references/token-spec.md) — 7 个 token 文件的规范

---

## 许可

MIT
