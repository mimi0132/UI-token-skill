# UI Design System Extractor — Theme Override

<div align="center">

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Skill](https://img.shields.io/badge/skill-v1.0.0-blue.svg)](skills/ui-design-system-extractor/SKILL.md)

**从设计稿给现有组件库做视觉再设计 — 保留交互,替换全部视觉变量**

</div>

你的项目已经在用 Element Plus / Ant Design / Naive UI / shadcn / MUI / Chakra。
你想让它**看起来像新的设计稿**(颜色、字体、间距、栅格、圆角、阴影、动效 — 全部按新设计走),
但**保留组件库的所有交互行为**(点击、键盘、表单、弹窗、动画时序、a11y,原封不动)。

这个 skill 帮你做这件事:从设计稿提取**所有视觉变量**,
生成 **1 个 CSS 覆盖文件**,引入到项目里,所有现有组件按新设计稿自动变脸,行为零变化。

兼容 **所有主流 AI 编程工具** (Trae / Cursor / Claude Code / Codex / Copilot / Cline / Roo Code)。

---

## 🎯 这个 skill 做什么

> **1. 识别** — 从 Figma 链接 / 设计图 / 手写 tokens 读出**所有视觉变量**
> **2. 替换 / 延伸** — 把目标组件库的视觉层按新设计稿重新映射;不存在的视觉模式则扩展
> **3. 保留** — 组件库自带的交互行为一行不动
> **4. 集成** — `import` 1 行,搞定

### 替换 / 保留的边界

| 类别 | 处理 | 原因 |
|------|------|------|
| 颜色 / 字号 / 字重 / 行高 / 字体 | 🔁 替换 | 纯视觉,新设计稿说了算 |
| 圆角 / 阴影 / 边框 | 🔁 替换 | 纯视觉,新设计稿说了算 |
| 间距 / 栅格 / 容器 / 断点 | 🔁 替换 | 视觉,影响布局结构 |
| 动效曲线 / 时长 / 焦点环 | 🔁 替换 | 视觉风格,影响"性格" |
| **点击 / 键盘 / 焦点 / 表单校验** | 🛡️ **保留** | 交互逻辑,组件库已验证 |
| **弹窗定位 / 滚轮锁定 / Esc 关闭** | 🛡️ **保留** | 交互逻辑,组件库已验证 |
| **API / props / 受控非受控** | 🛡️ **保留** | API 契约,改了就破坏代码 |
| **i18n 文案 / 错误提示** | 🛡️ **保留** | 内容层,不是视觉层 |

如果新设计稿有**原组件库不支持的视觉模式**(比如 5xl 字号、更复杂阴影、品牌色 glow),
Agent 会**新增 token + 用 direct CSS selector 扩展**,而不是要求用户改组件代码。

### 它做(只做)这一件事

给一个用 Element Plus 的 Vue 3 项目做视觉再设计 — 1 个 CSS 文件,搞定:

```diff
  // main.ts
  import 'element-plus/dist/index.css'
+ import './ui-theme/overrides/element-plus-theme-override.css'

  // 你原来的代码一行不用动
  // <el-button> 自动按新设计稿变色 / 变圆角 / 变字体 / 变间距
  // 点击 / 键盘 / 弹窗 / 表单校验 完全保留
```

### 它不做什么

- ❌ **不**生成新组件 (Button.vue / Button.tsx / 等等)
- ❌ **不**修改你已有的任何代码
- ❌ **不**改 API / props / 组件结构
- ❌ **不**调整交互行为
- ❌ **不**引入新的组件库

只改视觉。**只改视觉。**

---

## ✨ 特性

- **多源输入** — Figma 链接、本地图片、手写 Design Tokens (JSON / CSS / W3C)
- **覆盖全维度视觉变量** — 颜色、字体、字号、字重、行高、间距、圆角、阴影、边框、栅格、容器、断点、动效
- **保留交互行为** — 点击 / 键盘 / 焦点 / 表单 / 弹窗 / a11y,组件库自带的一行不动
- **8 个组件库开箱即用** — Element Plus、Ant Design、Ant Design Vue、Naive UI、shadcn/ui、MUI、Chakra UI、Vuetify
- **1 个文件,3 行集成** — 复制粘贴 1 行 import,搞定
- **可延伸** — 原库没暴露的视觉模式(如 5xl 字号、品牌 glow)会用 direct CSS 扩展,不会要求你改组件
- **暗色模式** — `[data-theme="dark"]` 切换,默认生成
- **可二次调参** — 改一个 hex,全局生效
- **多 Agent 兼容** — 一个 SKILL.md,所有 AI 编程工具都能用

---

## 🚀 安装

### 一键安装(推荐)

```bash
npx skills add <your-github-user>/ui-design-system-extractor --all
```

### 自定义安装

```bash
# 只装到 Claude Code 和 Cursor
npx skills add <your-github-user>/ui-design-system-extractor -a claude-code -a cursor -y

# 全局安装
npx skills add <your-github-user>/ui-design-system-extractor -g -y
```

### 手动安装

把 `skills/ui-design-system-extractor/` 复制到 Agent 的 skills 目录:

| Agent          | 复制到                                  |
|----------------|-----------------------------------------|
| Trae AI        | `.trae/skills/` 或 `~/.trae-cn/skills/` |
| Cursor         | `.cursor/skills/` 或 `~/.cursor/skills/` |
| Claude Code    | `.claude/skills/` 或 `~/.claude/skills/` |
| Codex          | `.codex/skills/` 或 `~/.codex/skills/`   |
| GitHub Copilot | `.github/skills/`                       |
| Cline          | `.cline/skills/` 或 `~/.cline/skills/`   |
| Roo Code       | `.roo/skills/` 或 `~/.roo/skills/`       |

### 不安装直接用

在任意 AI 聊天框粘贴:

```
Read https://raw.githubusercontent.com/<your-github-user>/ui-design-system-extractor/main/skills/ui-design-system-extractor/SKILL.md

帮我把这个 Figma 设计稿 [URL] 提取出视觉 token,生成 Element Plus 的样式覆盖文件
```

---

## 📖 使用示例

### Figma → Element Plus 全量换肤

```
我有一个 Figma 设计稿: https://www.figma.com/design/xxx/MyApp
我的项目是 Vue 3 + Element Plus
按这个设计稿重新设计所有视觉(颜色、字体、间距、栅格、圆角、阴影),但保留 Element Plus 的所有交互
```

Agent 会:
1. 调用 Figma MCP 提取所有节点的视觉变量(颜色 / 圆角 / 阴影 / 字体 / 字号 / 字重 / 间距 / 栅格)
2. 输出 7 个 token CSS 文件(`tokens/colors.css` / `typography.css` / `spacing.css` / `radius.css` / `shadow.css` / `border.css` / `layout.css`)
3. 输出 `overrides/element-plus-theme-override.css`(全维度映射,约 120 行)
4. 对原组件库没暴露的视觉模式(比如新设计用了 5xl 字号),通过 direct CSS 扩展
5. 输出一份预览页,你能在浏览器里看效果
6. 给一份 README,告诉你怎么集成 + 怎么局部调参

### 本地设计图 → Ant Design 全量换肤

```
这是设计图 /Users/me/dashboard.png
我的项目是 React + Ant Design 5
按这个图重新做所有视觉(颜色、字体、栅格、圆角、阴影),但保留 Ant Design 5 的所有交互
```

### 手写 tokens → Naive UI 全量换肤

```
我有一份 design-tokens.json:

{
  "color": { "primary": { "500": "#6366F1" } },
  "typography": { "body": { "fontFamily": "Inter" } },
  "radius": { "md": "12px" },
  "spacing": { "4": "16px" }
}

我的项目是 Naive UI,按这份 token 重新做所有视觉,保留所有交互行为。
```

### 仅换部分维度(子集模式)

如果只想换一部分视觉,告诉 Agent 保留哪些维度不动。例:

```
只换颜色和圆角,字体、间距、阴影、栅格全部沿用 Element Plus 默认
```

可选的子集模式:
- `full` — 全维度视觉替换(默认)
- `color-only` — 只换颜色
- `color-radius` — 颜色 + 圆角
- `color-typography` — 颜色 + 字体
- `custom:<list>` — 自定义要替换的维度

### 暗色模式

```
除了亮色主题,再生成一个暗色模式
```

---

## 🎨 输出结构

```
ui-theme/
├── tokens/
│   ├── theme.css                # 入口文件,@import 其他 6 个
│   ├── colors.css               # 主色/中性/语义/背景/文字/边框
│   ├── typography.css           # 字体族/字号/字重/行高
│   ├── spacing.css              # 4px 基础 + 语义间距
│   ├── radius.css               # 圆角刻度
│   ├── shadow.css               # 阴影刻度
│   ├── border.css               # 边框宽度/样式
│   ├── layout.css               # 栅格列数/容器宽度/断点
│   └── motion.css               # 动效曲线/时长(可选)
├── overrides/
│   └── <lib>-theme-override.css # ⭐ 主输出,1 个文件
├── preview/
│   └── preview.html             # 浏览器预览,验收用
└── README.md                    # 集成说明 + 调参指南
```

7 个 token 文件是**透明的中间产物** — 让你能看清、来回改。
**真正部署只需要 1 个 `overrides/<lib>-theme-override.css`。**

---

## 🔌 集成示例

### Element Plus (Vue 3)

```ts
// main.ts
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import './ui-theme/overrides/element-plus-theme-override.css'  // 👈 这一行
import App from './App.vue'

createApp(App).use(ElementPlus).mount('#app')
```

### Ant Design (React)

```tsx
// main.tsx
import 'antd/dist/reset.css'
import './ui-theme/overrides/antd-theme-override.css'  // 👈 这一行
import { ConfigProvider } from 'antd'

export function App() {
  return <ConfigProvider><Router /></ConfigProvider>
}
```

### shadcn/ui (React + Tailwind)

```tsx
// main.tsx
import './ui-theme/overrides/shadcn-theme-override.css'  // 👈 这一行
import './globals.css'
```

### Naive UI (Vue 3)

```vue
<!-- App.vue -->
<script setup lang="ts">
import { NConfigProvider } from 'naive-ui'
import { themeOverrides } from './naive-theme'  // 见 references/override-patterns.md
import './ui-theme/tokens/theme.css'
</script>

<template>
  <NConfigProvider :theme-overrides="themeOverrides">
    <router-view />
  </NConfigProvider>
</template>
```

---

## 🎨 调参

集成完之后,改设计只需要改 token 文件:

```css
/* tokens/colors.css */
:root {
  --color-primary-500: #6366F1;  /* 改这个,所有按钮、链接、焦点色都跟着变 */
}
```

改完 token,刷新页面,所有使用 `--color-primary-500` 的地方自动更新。**不需要改任何组件代码。**

### 暗色模式切换

```js
// 任何地方
document.documentElement.setAttribute('data-theme', 'dark')   // 开
document.documentElement.setAttribute('data-theme', 'light')  // 关
```

---

## 🧰 支持的组件库

| 库                | 框架       | 覆盖度    | 备注                             |
|-------------------|------------|----------|----------------------------------|
| **Element Plus**  | Vue 3      | ⭐⭐⭐⭐⭐  | 完整覆盖,推荐 Vue 项目首选       |
| **Ant Design**    | React      | ⭐⭐⭐⭐⭐  | 完整覆盖,推荐 React 项目首选     |
| **Ant Design Vue**| Vue 3      | ⭐⭐⭐⭐   | 完整覆盖                          |
| **Naive UI**      | Vue 3      | ⭐⭐⭐⭐   | JS theme overrides 路径更完整    |
| **shadcn/ui**     | React      | ⭐⭐⭐⭐   | 注意 HSL 格式要求                |
| **MUI**           | React      | ⭐⭐⭐    | v6 / Joy UI 完整;v5 受 var() 限制 |
| **Chakra UI**     | React      | ⭐⭐⭐⭐   | semanticTokens 完整              |
| **Vuetify**       | Vue 3      | ⭐⭐⭐⭐   | 完整覆盖                          |
| Bootstrap         | -          | ❌        | 没有 CSS 变量体系,不适用        |
| Foundation        | -          | ❌        | 没有 CSS 变量体系,不适用        |

---

## 🧰 支持的 Agent

| Agent            | 状态 | 安装目录                                |
|------------------|------|-----------------------------------------|
| **Trae AI**      | ✅    | `.trae/skills/`, `~/.trae-cn/skills/`   |
| **Cursor**       | ✅    | `.cursor/skills/`, `~/.cursor/skills/`  |
| **Claude Code**  | ✅    | `.claude/skills/`, `~/.claude/skills/`  |
| **Codex**        | ✅    | `.codex/skills/`, `~/.codex/skills/`    |
| **GitHub Copilot** | ✅  | `.github/skills/`                       |
| **Cline**        | ✅    | `.cline/skills/`, `~/.cline/skills/`    |
| **Roo Code**     | ✅    | `.roo/skills/`, `~/.roo/skills/`        |
| **Continue.dev** | ✅    | `.continue/skills/`                     |
| **Aider**        | ✅    | `.aider/skills/`                        |

任何具备 **文件读取 / 写入 / 图片分析 / Figma MCP 访问** 能力的 AI 工具都可以使用。

---

## 🛠 常见问题

### "我改了 token 颜色但组件没变"

引入顺序错了。必须先引组件库的 CSS,再引覆盖文件:

```ts
// ✅ 正确
import 'element-plus/dist/index.css'
import './ui-theme/overrides/element-plus-theme-override.css'

// ❌ 错误
import './ui-theme/overrides/element-plus-theme-override.css'
import 'element-plus/dist/index.css'
```

### "暗色模式不切换"

切换按钮要操作 `data-theme` 属性,不是改 class:

```js
// ✅ 正确
document.documentElement.setAttribute('data-theme', 'dark')

// ❌ 错误
document.documentElement.classList.add('dark')
```

### "某个组件不响应"

有些组件用了 hardcoded 颜色,不在 CSS 变量体系里。打开 DevTools 看 computed style,提 issue。

### "我换了主色,组合组件里有些状态没变(按钮 hover / 禁用色 / 输入框 focus)"

这是 CSS 变量级联的**已知边界**,不是 bug。常见三种情况:

1. **SCSS 派生色**:`--el-color-primary-light-3` 是 SCSS 编译时算好的 `mix(white, #409EFF, 30%)`,改 `--el-color-primary` 不会自动重算。
2. **硬编码颜色**:少数组件(动画帧、复杂 picker)源码里写死了 hex。
3. **组合组件自己的 token**:`<el-card>` 读 `--el-fill-color-blank`,不是 `--el-color-primary`。

skill 的解法是**完整覆盖所有派生色 + 关键组合组件的复述断言**,并提供一份 DevTools 自检脚本让你在预览页自己跑一遍。

### 怎么自检覆盖完整度

在预览页打开 DevTools,跑这段:

```js
// 找所有没经过 CSS 变量、用了硬编码 rgb 的目标组件
const hardcoded = []
document.querySelectorAll('.el-button, .el-input, .el-card, .el-tag, .el-dialog, .el-table, .el-menu, .el-pagination, .el-form').forEach(el => {
  const cs = getComputedStyle(el)
  for (const prop of ['color', 'background-color', 'border-color']) {
    const v = cs[prop]
    if (v.startsWith('rgb') && !v.includes('var(')) {
      hardcoded.push({ tag: el.tagName, class: el.className, prop, value: v })
    }
  }
})
console.table(hardcoded)
```

如果有结果,就是漏网的地方。把对应的组件类名 + 颜色提给 Agent,会让它在 override 文件里追加 direct CSS 复述。

### "我想保留字体和间距,只换颜色和圆角"

告诉 Agent "只换颜色和圆角,其它保持默认" — 它会跳过对应的 token 文件,只输出与颜色 / 圆角相关的 CSS 变量。

可选子集模式(在提问时勾选):
- `full` — 全维度视觉替换(默认)
- `color-only` — 只换颜色
- `color-radius` — 颜色 + 圆角
- `color-typography` — 颜色 + 字体
- `custom:<list>` — 自定义要替换的维度,例 `custom:colors,radius,shadow`

### "设计稿只给了 1 个主色,怎么覆盖组件库 40+ 颜色变量"

设计稿给的是**采样点**,不是完整调色板。Agent 不会瞎填 40 个 hex,而是用**单锚点 + 数学推导**生成完整体系:

```css
/* 你给 1 个主色,Agent 推导 9 个停靠点 */
--color-primary-500: #6366F1;              /* 来自设计稿 */
--color-primary-50:  color-mix(in srgb, var(--color-primary-500)  8%, white);
--color-primary-100: color-mix(in srgb, var(--color-primary-500) 16%, white);
/* ... 9 个推导停靠点,改 500 一次,全部联动 ... */
--color-primary-900: color-mix(in srgb, var(--color-primary-500) 44%, black);
```

中性色 / 语义色同理:
- **中性灰**: 抽 design 没给,自动用主色色相 + 0% 饱和度推导
- **成功/警告/危险/信息**: 抽 design 没给,用绿/琥珀/红/蓝默认锚点 + 推导

**改一个 hex,整张调色板重新算**。换主色不痛苦,改间距不痛苦,任何后续调参都是改 1 个值。

### "除了颜色,其他变量也能这么推导吗"

**能,有 6 类可推导变量**,全部用"单锚点 + 数学"自动扩展:

| 变量 | 锚点 | 数学 | 改 1 个值的效果 |
|---|---|---|---|
| **颜色** | `--color-primary-500` | `color-mix(in srgb, …, white\|black)` | 整张调色板重算 |
| **间距** | `--space-base: 4px` | `calc(var(--space-base) * N)` | 整个间距刻度缩放 |
| **圆角** | `--radius-base: 6px` | `calc(var(--radius-base) * M)` | 整体变锐 / 变柔 |
| **字号** | `--font-size-base: 16px` + `--font-size-ratio: 1.25` | `calc(var(--font-size-base) * pow(var(--font-size-ratio), N))` | 整个字号阶重新调 |
| **行高** | `--line-height-base: 1.5` | direct / `calc` | 行高节奏统一调 |
| **边框宽度** | `--border-width-base: 1px` | `calc(var(--border-width-base) * N)` | 改粗细 |
| **动效时长** | `--motion-duration-base: 200ms` | `calc(var(--motion-duration-base) * K)` | 整体快 / 慢 |

**改 6-8 个锚点,所有视觉变量重新算**。"想要整体更紧凑" → 改 `--space-base: 4px` → `3px` 一行;"想要 iOS 风格更柔" → 改 `--radius-base: 6px` → `12px` 一行。

**6 类不能推导** — 字体族 / 字重 / 阴影 / 栅格列数 / 断点 / 动效曲线 — 这些用行业固定值或 3-5 个命名 token。

### "新设计有 5xl 字号,组件库只有 h1-h6"

Agent 会**新增 token + direct CSS 扩展**,不要求你改组件代码:

```css
:root {
  --el-font-size-display: 5.5rem;  /* 新增的 token */
}
.el-heading-display {
  font-size: var(--el-font-size-display);
}
```

### "我想换交互行为(比如改成非受控)"

不做。这个 skill 只换视觉。改交互是另一个 skill(组件重构)的事。

---

## 📐 设计原则

1. **保留交互,只换视觉** — 颜色 / 字体 / 间距 / 栅格 / 圆角 / 阴影 / 动效全替换;点击 / 键盘 / 表单 / 弹窗保留
2. **1 个文件搞定** — 主输出是 `overrides/<lib>-theme-override.css`,其它都是辅助
3. **7 个 token 文件透明** — 颜色 / 字体 / 间距 / 圆角 / 阴影 / 边框 / 布局独立可改
4. **不写新组件** — 永远不输出 `.vue` / `.tsx` / `.jsx` 业务组件
5. **API 完全不变** — 你的 `<el-button>` 仍然是 `<el-button>`,只是视觉变了
6. **覆盖所有变体** — light/dark 都要有,primary 的 light-3/5/7/8/9/dark-2 都要有
7. **可延伸** — 原库没暴露的视觉模式,用 `:where(.lib-class)` + direct CSS 扩展,保持低优先级

---

## 📚 参考文档

- [SKILL.md (安装版)](skills/ui-design-system-extractor/SKILL.md) — Agent 加载的完整指令
- [Installation Reference](skills/ui-design-system-extractor/references/installation.md) — 按 Agent 详细安装
- [System Prompt Reference](skills/ui-design-system-extractor/references/system-prompt.md) — 提取检查表与质量门
- [Token Specification](skills/ui-design-system-extractor/references/token-spec.md) — 5 个 token 文件的详细规范
- [Override Patterns](skills/ui-design-system-extractor/references/override-patterns.md) — 各 UI 库的完整 CSS 变量映射(主输出参考)
- [Figma Extraction](skills/ui-design-system-extractor/references/figma-extraction.md) — Figma MCP 使用方法

---

## 📄 许可证

MIT © UI Design System Extractor Contributors
