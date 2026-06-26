---
name: "ui-design-system-extractor"
description: "Extracts design tokens (colors, typography, spacing, components) from Figma files, local design images, or hand-written tokens, then generates a component library that can override the styles of an existing component library (Element Plus / Ant Design / Naive UI / shadcn, etc.) to achieve visual consistency with the source design. Invoke when user provides a design source and wants a styled, ready-to-use component library matching that design, or wants to override an existing UI library's theme to match a design."
---

# UI Design System Extractor

End-to-end pipeline: **Design Source → Design Tokens → Component Library → Style Override**.
Works on **every major AI coding agent** (Cursor, Claude Code, Trae, Codex, GitHub Copilot, Cline, Roo Code …).

## 安装

```bash
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor --all
```

`--all` 自动检测本机所有 AI 编程工具，把技能装到所有检测到的 Agent,无需手动选择。

> 详细安装方式 (按 Agent 选择 / 全局 / 手动) 见 [skills/ui-design-system-extractor/SKILL.md](skills/ui-design-system-extractor/SKILL.md)

## 核心能力

- **多源输入**: Figma 链接 / 本地设计稿图片 / 手写 Design Tokens (JSON / CSS / W3C)
- **完整 token 体系**: colors / typography / spacing / radius / theme,5 个独立 CSS 文件
- **组件库生成**: 8+ 组件 (Button / Input / Card / Modal / Select / Tag / Avatar / Tooltip …),每个自带 `<DEMO>` 块
- **样式覆盖**: 输出的 CSS 变量可直接覆盖 Element Plus / Ant Design / Naive UI / shadcn 等
- **框架可选**: Vue 3 / React / 框架无关
- **CLI 模式**: `--full` / `--style-only` / `--doc` / `--framework` / `--base-lib`

## 工作流程 (概览)

1. **澄清目标**: 框架 + 基础组件库 + 输出模式
2. **提取 tokens**: 5 个 token 文件 (theme → colors → typography → spacing → radius)
3. **生成组件**: 每个组件匹配基础库的 API,样式只引用 token
4. **样式覆盖**: 生成 `<lib>-theme-override.css` 把基础库 CSS 变量映射到设计 token
5. **预览页**: 颜色库 → 字体 → 间距/圆角 → 栅格 → 组件列表
6. **验证**: 零硬编码颜色 / 零硬编码 px / 每个组件含 DEMO

完整流程见 [skills/ui-design-system-extractor/SKILL.md](skills/ui-design-system-extractor/SKILL.md)

## 使用示例

```
我有一个 Figma 设计稿: https://www.figma.com/design/xxx/MyApp
帮我生成 Vue 3 组件库,风格统一到 Element Plus
```

Agent 会自动完成: 调用 Figma MCP 提取数据 → 生成 5 个 token 文件 → 生成 8+ 组件 → 生成 Element Plus 主题覆盖 CSS → 输出预览页。

## 支持的 Agent

| Agent | 状态 |
|-------|------|
| Trae AI | ✅ |
| Cursor | ✅ |
| Claude Code | ✅ |
| Codex | ✅ |
| GitHub Copilot | ✅ |
| Cline | ✅ |
| Roo Code | ✅ |

任何具备 **文件读取 / 写入 / 图片分析 / Figma MCP 访问** 能力的 Agent 都可使用。
