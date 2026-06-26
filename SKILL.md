---
name: "ui-design-system-extractor"
description: "Extracts all visual design tokens (colors, typography, font-size, font-weight, line-height, spacing, radius, shadow, border, grid, container, breakpoints, motion) from a design source (Figma / image / hand-written tokens), then generates a single CSS override file that re-skins an existing component library (Element Plus / Ant Design / Naive UI / shadcn / MUI / Chakra / Vuetify) by mapping those tokens onto the library's CSS variables. Preserves all interaction behavior (clicks, keyboard, forms, modals, a11y). Invoke when user has an existing component library and wants to change its visual style to match a new design without changing any component code."
---

# UI Design System Extractor — Theme Override

> **Single purpose**: 给现有的组件库做视觉再设计 —— **保留所有交互行为,只替换视觉层**。
>
> **替换或延伸**(从新设计稿提取):颜色 / 字号 / 字重 / 行高 / 字体 / 间距 / 圆角 / 阴影 / 边框 / 栅格 / 容器 / 断点 / 动效曲线 / 焦点环。
>
> **保留**(原组件库自带,不改一行):点击 / 键盘 / 焦点 / 表单校验 / 弹窗定位 / 动画时序 / 无障碍属性 / 滚动行为 / API / props。
>
> **它做什么**: 从设计稿提取全维度视觉 token,生成 1 个 CSS 覆盖文件,引入到用户项目,所有现有的 `<el-button>` / `<Button>` / shadcn 组件按新设计稿自动变脸,行为零变化。
>
> **它不做什么**: 不生成新组件、不改 API、不动用户已有代码、不改交互行为。

## 安装

```bash
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor --all
```

`--all` 自动检测本机所有 AI 编程工具并安装,无需手动选择。
详见 [skills/ui-design-system-extractor/SKILL.md](skills/ui-design-system-extractor/SKILL.md)

## 典型场景

你已经在用 **Element Plus / Ant Design / shadcn / Naive UI** 等组件库,代码里写满了 `<el-button>` / `<Button>`,你拿到一张设计稿 / Figma 文件 / 颜色规范,**只想换风格,不想动组件代码** —— 用这个 skill。

## 它帮你做的事

1. **读懂设计稿** — 从 Figma / 图片 / tokens 提取颜色、字体、间距、圆角、阴影
2. **整理成 5 个 token 文件** — colors / typography / spacing / radius / theme
3. **生成 1 个 override CSS** — 把设计 token 映射到你现有组件库的 CSS 变量
4. **生成预览页** — 你能立刻看到新风格套到现有组件上是什么效果
5. **告诉你怎么集成** — 一行 import,完事

## 输出长什么样

```
output/
├── tokens/
│   ├── theme.css             # 入口,组件层 token
│   ├── colors.css            # 颜色库 (primary/neutral/semantic, 50-900)
│   ├── typography.css        # 字体系统
│   ├── spacing.css           # 间距 (4px 基准)
│   └── radius.css            # 圆角
├── overrides/
│   └── element-plus-theme-override.css    ← 唯一要导入的文件
├── preview/
│   └── preview.html          # 打开就能看到新风格
└── README.md                 # 集成说明
```

**关键文件只有一个**: `overrides/<lib>-theme-override.css`。
用户把这个文件 + `tokens/` 复制到自己项目,在入口导入一次,所有现有组件自动变脸。

## 集成示例 (Vue 3 + Element Plus)

```ts
// main.ts
import 'element-plus/dist/index.css'
import './ui-theme/tokens/theme.css'                  // 设计 token
import './ui-theme/overrides/element-plus-theme-override.css'  // 覆盖
```

完事。代码里所有 `<el-button type="primary">` 自动变成设计稿里的主色,所有圆角、阴影、字体都跟着改。

## 集成示例 (React + Ant Design)

```tsx
// main.tsx
import 'antd/dist/reset.css'
import './ui-theme/tokens/theme.css'
import './ui-theme/overrides/antd-theme-override.css'
```

## 集成示例 (React + shadcn)

```tsx
// main.tsx
import './ui-theme/tokens/theme.css'
import './ui-theme/overrides/shadcn-theme-override.css'  // 含 tailwind.config.js 调整
```

## 支持的组件库

Vue: Element Plus, Naive UI, Ant Design Vue, Vuetify
React: Ant Design, shadcn/ui, MUI, Chakra UI
其他: 用户可指定(只要它的 CSS 变量公开)

## 支持的设计源

- Figma 链接 / file key / node URL — 通过 Figma AI Bridge MCP 抓取
- 本地 PNG / JPG / Sketch 导出 — Agent 视觉分析
- 手写 JSON / CSS / SCSS / W3C tokens — 直接读取

## 详细工作流

[skills/ui-design-system-extractor/SKILL.md](skills/ui-design-system-extractor/SKILL.md)
[references/override-patterns.md](skills/ui-design-system-extractor/references/override-patterns.md) — 各 UI 库的 CSS 变量映射表
