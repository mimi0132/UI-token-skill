---
name: "ui-design-system-extractor"
description: "Extracts design tokens (colors, typography, spacing, components) from Figma files, local design images, or hand-written tokens, then generates a component library that can override the styles of an existing component library (Element Plus / Ant Design / Naive UI / shadcn, etc.) to achieve visual consistency with the source design. Invoke when user provides a design source and wants a styled, ready-to-use component library matching that design, or wants to override an existing UI library's theme to match a design."
---

# UI Design System Extractor

End-to-end pipeline: **Design Source → Design Tokens → Component Library → Style Override**.
Supports Vue 3, React, and framework-agnostic outputs. The user provides ONE of three input
sources, picks a target framework + base component library, and the skill produces a working
component library whose styles can replace the base library to match the design.

---

## 安装 (Installation)

### 一键安装到所有 Agent (推荐)

```bash
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor --all
```

`--all` 等价于 `--skill '*' --agent '*' -y`,**跳过所有交互提示**,自动检测本机所有 AI 编程工具并安装到所有检测到的 Agent。

### 自定义安装

```bash
# 只装到 Claude Code 和 Cursor
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor -a claude-code -a cursor -y

# 全局安装 (所有项目可用)
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor -g -y

# 安装到指定项目
npx skills add YOUR_GITHUB_USER/ui-design-system-extractor -p /path/to/project -y
```

### 不安装直接使用

在任意 AI 聊天框中粘贴以下任一内容,Agent 会自动加载并按规范工作:

```
Read https://raw.githubusercontent.com/YOUR_GITHUB_USER/ui-design-system-extractor/main/skills/ui-design-system-extractor/SKILL.md
```

或直接贴整段 SKILL.md 内容。

### 卸载

```bash
npx skills remove ui-design-system-extractor --all
```

---

## 1. When to Invoke

Invoke this skill when the user message contains ANY of the following signals:

- A Figma link / file key / node URL and an intent like "做一套组件库", "按这个设计稿生成组件", "统一风格"
- A local design image (PNG / JPG / Sketch export) + "提取设计", "生成组件", "做一套 UI"
- A design token file (JSON / CSS / SCSS / W3C token format) + "覆盖 Element Plus / Ant Design / shadcn 的样式"
- Phrases: "样式替换", "换肤", "统一设计风格", "design tokens 提取", "组件库生成", "主题定制"

Do NOT invoke when:

- User only wants a single component (use a generic frontend skill)
- User wants pure CSS tweaks to an existing project without generating a library
- Source is a code repo, not a design asset

---

## 2. Hard Constraints (MUST)

These constraints come from prior runs. Violating any of them is a hard failure.

1. **Five required token files** MUST be produced (in this fixed order):
   `theme.css` → `colors.css` → `typography.css` → `spacing.css` → `radius.css`
   (Plus `components/`, `index.ts`, `README.md` for full mode.)
2. **Color tokens and typography tokens MUST be in separate files** — never inline into
   `theme.css`. This is what makes the system overridable.
3. **Each component MUST include a `<DEMO>` block** that renders all of its variants
   side by side. A component without a demo is incomplete.
4. **At least 8 components** in full mode: Button, Input, Card, Modal/Dialog, Select,
   Tag, Avatar, Tooltip, Tabs, Toast/Notification — pick the 8 that match the design
   source's visible components.
5. **Generated components MUST be directly usable** — import path works, no manual
   fixes required, prop types match the base library.
6. **Preview page (if generated) MUST display, in this order**: color palette demo →
   typography demo → spacing/radius demo → grid system demo → component list demo.
7. **HTML in JS template strings MUST use entity escaping** for `<` / `>` (e.g. `&lt;div&gt;`)
   to avoid JSX/Vue template syntax errors.

---

## 3. Inputs

Accept exactly ONE primary source. Ask the user to confirm before proceeding if ambiguous.

| Source Type    | How to Detect                              | Tool / Action                       |
|----------------|--------------------------------------------|-------------------------------------|
| Figma link     | `figma.com/(file|design)/...` URL           | `mcp_Figma_AI_Bridge.get_figma_data`|
| Local image    | `.png`/`.jpg`/`.jpeg`/`.sketch` path        | `mcp_Figma_AI_Bridge.download_figma_images` (after Figma upload) OR ask user to describe the design |
| Hand-written   | `*.json` / `*.css` / `*.scss` / `*.tokens`  | `Read` + parse directly             |

If source is a Figma link, ALWAYS call `get_figma_data` first; do not guess values.

---

## 4. Workflow

Execute these steps **in order**. Pause and ask the user only at the marked checkpoints.

### Step 0 — Clarify Output Target (CHECKPOINT)

Ask the user:

1. **Target framework**: Vue 3 / React / framework-agnostic?
2. **Base component library to override** (optional but recommended):
   - Vue: Element Plus / Naive UI / Ant Design Vue / Vuetify
   - React: Ant Design / MUI / shadcn/ui / Chakra UI
   - Or "none — generate a standalone library"
3. **Output mode**:
   - `--full` (default): tokens + components + preview + README
   - `--style-only`: only the 5 token files + override guide
   - `--doc`: full + `design-spec.md` documentation

### Step 1 — Extract Design Tokens

From the chosen source, produce these five files. Use **CSS custom properties** with
semantic naming (e.g. `--color-primary-500`, `--font-size-md`, `--space-4`).

#### `theme.css` (entry point)
- Imports the four token files in fixed order
- Defines `:root` reset, dark-mode variant (`[data-theme="dark"]`)
- Maps semantic tokens → component tokens (e.g. `--button-bg: var(--color-primary-500)`)

#### `colors.css`
- Primary / secondary / accent / neutral / semantic (success/warning/danger/info) scales
- Each scale: 50, 100, 200, 300, 400, 500, 600, 700, 800, 900
- Both HEX and `color-mix(in srgb, ...)` derivations for tints

#### `typography.css`
- Font families (heading + body + mono)
- Type scale: `xs` (12px) / `sm` (14px) / `md` (16px) / `lg` (18px) / `xl` (20px) / `2xl` (24px) / `3xl` (30px) / `4xl` (36px)
- Line heights + font weights as separate tokens
- Letter-spacing for display sizes

#### `spacing.css`
- 4px base: `space-0` (0) → `space-16` (64px)
- Plus semantic: `--space-component-padding`, `--space-section-gap`

#### `radius.css`
- `radius-none` (0) / `radius-sm` (2px) / `radius-md` (6px) / `radius-lg` (12px) / `radius-xl` (16px) / `radius-full` (9999px)

### Step 2 — Generate Components

For each of the 8+ components:

1. **Read the design source** to find visual hints (corner radius, padding, color usage).
2. **Match the base library's API** if override mode:
   - Element Plus: `<el-button>`, props `type`/`size`/`variant`
   - Ant Design: `<Button>`, props `type`/`size`/`variant`
   - shadcn: `asChild` pattern, CVA variants
3. **Write the component file** with:
   - Typed props (TS) / PropTypes (JS)
   - All base-library variants supported
   - A `<DEMO>` JSX/template block showing every variant + state (hover, active, disabled, loading)
4. **Write the matching styles** using ONLY tokens from the 5 token files — never hard-code
   a hex value or px in a component file.

### Step 3 — Style Override (if base library chosen)

Produce a `<lib>-theme-override.css` file that imports the 5 token files and re-declares
the base library's CSS variables. Example for Element Plus:

```css
@import './theme.css';

:root {
  --el-color-primary: var(--color-primary-500);
  --el-color-primary-light-3: var(--color-primary-300);
  --el-color-primary-light-5: var(--color-primary-100);
  --el-color-primary-light-7: var(--color-primary-50);
  --el-color-primary-light-8: var(--color-primary-50);
  --el-color-primary-light-9: var(--color-primary-50);
  --el-color-primary-dark-2: var(--color-primary-700);
  --el-border-radius-base: var(--radius-md);
  --el-font-family: var(--font-family-body);
  /* ... map all relevant base-lib vars to design tokens */
}
```

Append a usage snippet to `README.md`:

```ts
// main.ts
import 'ui-design-system-extractor/dist/lib/element-plus-theme-override.css'
```

### Step 4 — Preview Page (`--full` mode only)

Generate `preview.html` (or `src/preview.ts` → built to `dist/preview.html`) that renders
in this exact order:

1. **Color palette** — swatch grid grouped by scale (primary, secondary, neutral, semantic),
   each swatch shows token name + HEX + click-to-copy.
2. **Typography system** — sample text for each size/weight combo, with the token name.
3. **Spacing & radius** — visual bars showing each space step; rounded rectangles showing
   each radius step.
4. **Grid system** — 12-column overlay, 5 breakpoints (sm/md/lg/xl/2xl), spacing scale ruler.
5. **Component list** — one card per component, each card showing its `<DEMO>` block inline.

### Step 5 — Document & Export

- `index.ts` re-exports all components and tokens.
- `README.md` covers: install, import the override CSS, customize tokens, extend with
  custom components, troubleshooting.
- `design-spec.md` (`--doc` mode only): markdown documentation of the extracted design
  system (color rationale, type ramp, spacing philosophy, component specs).

### Step 6 — Verify

Before declaring done, run these checks:

- [ ] All 5 token files exist and import each other correctly
- [ ] `grep -r "#[0-9a-fA-F]\{3,8\}" components/` returns ZERO raw hex values
- [ ] `grep -r "[0-9]px" components/` returns ZERO raw px (use `var(--space-*)`)
- [ ] Every component file contains a `<DEMO>` block
- [ ] Preview page renders all 5 sections in the right order
- [ ] `index.ts` re-exports all public symbols

---

## 5. Output Directory Layout

```
ui-design-system-extractor/
├── SKILL.md
├── package.json
├── tsconfig.json
├── tokens/
│   ├── theme.css
│   ├── colors.css
│   ├── typography.css
│   ├── spacing.css
│   └── radius.css
├── components/
│   ├── Button.tsx (or .vue)
│   ├── Input.tsx
│   ├── Card.tsx
│   └── ... (8+ files)
├── overrides/
│   └── <base-lib>-theme-override.css
├── preview/
│   └── preview.html
├── dist/
│   └── (built output)
├── index.ts
├── README.md
└── design-spec.md   (only in --doc mode)
```

---

## 6. CLI Flags (if implemented as a tool)

| Flag           | Effect                                                  |
|----------------|---------------------------------------------------------|
| `--full`       | Default. Tokens + components + preview + README.        |
| `--style-only` | Only the 5 token files + override guide. No components. |
| `--doc`        | `--full` + `design-spec.md`.                            |
| `--framework`  | `vue` / `react` / `agnostic`. Default: auto-detect.     |
| `--base-lib`   | `element-plus` / `antd` / `naive-ui` / `shadcn` / `none`. |
| `--source`     | Path or URL to the design source.                       |
| `--out`        | Output directory. Default: `./ui-design-system-extractor/`. |

---

## 7. Examples

### Example 1 — Figma → Vue 3 + Element Plus

User: "用这个 Figma 文件 https://www.figma.com/design/abc123/MyApp 生成一套 Vue 3 组件库,风格要统一到 Element Plus"

Action:
1. Call `mcp_Figma_AI_Bridge.get_figma_data` on the URL.
2. Confirm: framework = Vue 3, base lib = Element Plus, mode = `--full`.
3. Generate all 5 token files, 8+ Vue SFCs, `element-plus-theme-override.css`, preview,
   README.

### Example 2 — Local image → React + shadcn

User: "把这张设计图 /Users/me/dashboard.png 做成 React 组件库,要能覆盖 shadcn"

Action:
1. If Figma upload is available, use `download_figma_images` after upload; otherwise
   ask the user to describe key visual values.
2. Confirm: framework = React, base lib = shadcn/ui, mode = `--full`.
3. Generate tokens, 8+ TSX components, `shadcn-theme-override.css`, preview, README.

### Example 3 — Hand-written tokens → style override only

User: "我有一份 design-tokens.json,想覆盖 Ant Design 的样式"

Action:
1. `Read` the JSON file.
2. Confirm: framework = React, base lib = Ant Design, mode = `--style-only`.
3. Produce 5 token files + `antd-theme-override.css` + a short override-guide README.
   Skip component generation.

---

## 8. Anti-Patterns (DO NOT)

- Do NOT hard-code any color, font-size, spacing, or radius inside a component file.
  Always reference a token.
- Do NOT skip the DEMO block on any component.
- Do NOT merge `colors.css` and `typography.css` into `theme.css`. They MUST be separate
  files for the override pattern to work.
- Do NOT generate fewer than 8 components in `--full` mode.
- Do NOT use raw HTML inside JS template strings without entity escaping.
- Do NOT silently pick a framework — always ask at Step 0.
- Do NOT output a component library without testing it can be imported and rendered
  (preview page exists for a reason).

---

## 9. References

Load these as needed during execution:

- [references/installation.md](references/installation.md) — Per-agent install instructions
  (Trae / Cursor / Claude Code / Codex / Copilot / Cline / Roo Code)
- [references/system-prompt.md](references/system-prompt.md) — Design system rationale
  and rules when the user asks "why this token"
- [references/token-spec.md](references/token-spec.md) — Detailed token specification
  (naming convention, dark mode, contrast checks)
- [references/override-patterns.md](references/override-patterns.md) — Per-base-lib CSS
  variable mapping (Element Plus / Ant Design / Naive UI / shadcn / MUI / Chakra)
- [references/figma-extraction.md](references/figma-extraction.md) — Figma AI Bridge MCP
  usage and fallback strategies

---

## 10. Supported Agents

| Agent       | Install Location            | Notes                              |
|-------------|-----------------------------|------------------------------------|
| Trae AI     | `.trae/skills/<name>/`      | Native Trae skills directory       |
| Cursor      | `.cursor/skills/<name>/`    | Detected by `npx skills`           |
| Claude Code | `.claude/skills/<name>/`    | Detected by `npx skills`           |
| Codex       | `.codex/skills/<name>/`     | Detected by `npx skills`           |
| GitHub Copilot | `.copilot/skills/<name>/` | Detected by `npx skills`           |
| Cline       | `.cline/skills/<name>/`     | Detected by `npx skills`           |
| Roo Code    | `.roo/skills/<name>/`       | Detected by `npx skills`           |

Any agent that can **read files, write files, and analyze images** can use this skill
even without explicit detection — just paste the SKILL.md URL.
