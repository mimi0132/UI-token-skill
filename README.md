# UI Design System Extractor

<div align="center">

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Skill](https://img.shields.io/badge/skill-v1.0.0-blue.svg)](skills/ui-design-system-extractor/SKILL.md)

**从设计稿一键生成可覆盖现有组件库样式的设计系统**

</div>

从设计稿 (Figma / 本地图片 / 手写 tokens) 一键生成完整的设计系统 + 组件库,输出
的 CSS 变量可直接覆盖 Element Plus / Ant Design / Naive UI / shadcn / MUI / Chakra
等主流组件库,达到与设计稿 100% 视觉一致。

兼容 **所有主流 AI 编程工具** (Trae / Cursor / Claude Code / Codex / GitHub Copilot
/ Cline / Roo Code)。

## ✨ 特性

- **多源输入**: Figma 链接、本地设计稿图片、手写 Design Tokens (JSON / CSS / W3C)
- **5 个 token 文件**: theme / colors / typography / spacing / radius,完全可独立覆盖
- **8+ 组件库**: Button / Input / Card / Modal / Select / Tag / Avatar / Tooltip 等,每个自带 DEMO 块
- **样式覆盖**: 一键生成 `<lib>-theme-override.css`,覆盖现有组件库视觉
- **框架可选**: Vue 3 / React / 框架无关
- **三种模式**: `--full` / `--style-only` / `--doc`
- **暗色模式**: 强制要求所有 token 有 dark variant
- **多 Agent 兼容**: 一个 SKILL.md,所有 AI 编程工具都能用

## 🚀 安装

### 一键安装 (推荐)

```bash
npx skills add <your-github-user>/ui-design-system-extractor --all
```

`--all` 自动检测本机所有 AI 编程工具并安装,无需手动选择。

### 自定义安装

```bash
# 只装到 Claude Code 和 Cursor
npx skills add <your-github-user>/ui-design-system-extractor -a claude-code -a cursor -y

# 全局安装
npx skills add <your-github-user>/ui-design-system-extractor -g -y
```

### 手动安装 (无 CLI)

把 `skills/ui-design-system-extractor/` 复制到你的 Agent 的 skills 目录:

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
```

---

## 📖 使用示例

### Figma → Vue 3 + Element Plus

```
我有一个 Figma 设计稿: https://www.figma.com/design/xxx/MyApp
帮我生成 Vue 3 组件库,风格统一到 Element Plus
```

Agent 会自动完成:
1. 调用 Figma MCP 提取所有节点的样式数据
2. 推导完整 Design Token 体系 (颜色/字体/间距/圆角/阴影)
3. 生成 5 个 token CSS 文件
4. 生成 8+ Vue 组件 (匹配 Element Plus API)
5. 生成 `element-plus-theme-override.css` 一键覆盖
6. 输出预览页面 (颜色库 → 字体 → 间距/圆角 → 栅格 → 组件列表)

### 本地设计稿 → React + shadcn

```
把这张设计图 /Users/me/dashboard.png 做成 React 组件库,能覆盖 shadcn
```

### 手写 tokens → 纯样式覆盖

```
我有一份 design-tokens.json,想覆盖 Ant Design 的样式,不要生成组件
```

Agent 进入 `--style-only` 模式,只输出 5 个 token 文件 + `antd-theme-override.css`。

---

## 🎨 输出结构

```
ui-design-system-extractor/
├── tokens/
│   ├── theme.css          # 入口 + 组件 token
│   ├── colors.css         # 颜色库 (primary/secondary/neutral/semantic)
│   ├── typography.css     # 字体系统 (family/size/weight/line-height)
│   ├── spacing.css        # 间距 (4px 基础 + 语义)
│   └── radius.css         # 圆角 (sm/md/lg/xl/full)
├── components/            # 8+ 组件,匹配基础库 API
│   ├── Button.vue
│   ├── Input.vue
│   ├── Card.vue
│   └── ...
├── overrides/             # 样式覆盖
│   └── <lib>-theme-override.css
├── preview/               # 预览页
│   └── preview.html
├── index.ts
├── README.md
└── design-spec.md         # --doc 模式才有
```

---

## 🛠 CLI 模式

| 开关              | 用途                                          |
|-------------------|-----------------------------------------------|
| `--full`          | 默认。tokens + 组件 + 预览 + README            |
| `--style-only`    | 只输出 5 个 token 文件 + 覆盖指南             |
| `--doc`           | `--full` + `design-spec.md` 文档              |
| `--framework vue` | Vue 3 输出 (默认)                              |
| `--framework react` | React 输出                                   |
| `--base-lib`      | `element-plus` / `antd` / `naive-ui` / `shadcn` / `mui` / `chakra` / `none` |

**典型用法**:

```bash
# 第一次:先只出设计 Token 给设计师确认
npx skills add <user>/ui-design-system-extractor --all
# 然后在 AI 中: "根据 figma xxx 输出 --style-only"

# 设计师确认后:出完整组件库 + 设计文档
"按刚才的 tokens,生成完整组件库, --doc"
```

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

## 📐 设计原则

1. **Token 优先** — 组件中不出现任何硬编码颜色 / px,全部引用 token
2. **每个组件自带 DEMO** — 预览页直接解析 `<DEMO_START>...</DEMO_START>`
3. **API 匹配基础库** — Element Plus 风格就输出 `<el-button>`,不输出 `<MyButton>`
4. **暗色模式必备** — 所有 token 都要有 `[data-theme="dark"]` 变体
5. **强制分离 5 个 token 文件** — 颜色和字体绝不能合并到 theme.css

---

## 📚 参考文档

- [SKILL.md (安装版)](skills/ui-design-system-extractor/SKILL.md) — Agent 加载的完整指令
- [Installation Reference](skills/ui-design-system-extractor/references/installation.md) — 按 Agent 详细安装
- [System Prompt Reference](skills/ui-design-system-extractor/references/system-prompt.md) — 设计哲学与质量门
- [Token Specification](skills/ui-design-system-extractor/references/token-spec.md) — 详细 token 规范
- [Override Patterns](skills/ui-design-system-extractor/references/override-patterns.md) — 各 UI 库的 CSS 变量映射
- [Figma Extraction](skills/ui-design-system-extractor/references/figma-extraction.md) — Figma MCP 使用

---

## 📦 项目结构

```
.
├── SKILL.md                                # 仓库根目录 (GitHub 显示)
├── README.md
├── package.json
├── .gitignore
├── LICENSE
└── skills/
    └── ui-design-system-extractor/
        ├── SKILL.md                        # 完整安装版
        └── references/
            ├── installation.md
            ├── system-prompt.md
            ├── token-spec.md
            ├── override-patterns.md
            └── figma-extraction.md
```

---

## 📄 License

MIT
