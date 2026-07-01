# 中创组件库 — 布局补丁资源

> 这里是 **中创组件库** 的 5 个布局补丁文件(SCSS 源码),由公司项目组提供,
> 复制自公司内部中创组件库仓库的 `styles/element/` 目录。

## 文件清单

| 文件 | 作用 | 影响的组件 |
|------|------|----------|
| [`_element.scss`](./_element.scss) | 入口文件 (`@use` 其他 4 个) | — |
| [`_avatar.scss`](./_avatar.scss) | 头像样式 | `el-avatar` |
| [`_message.scss`](./_message.scss) | 消息提示样式 | `el-message` (success/warning/error/info/primary) |
| [`_popconfirm.scss`](./_popconfirm.scss) | 气泡确认框按钮颜色 | `el-popconfirm` |
| [`_tooltip.scss`](./_tooltip.scss) | 弹出层浅色/深色边框 + 阴影 | `el-popper` (影响 el-tooltip / el-popover / el-dropdown / el-select) |

## 这些补丁是什么

中创组件库 = **Element Plus 1.x/2.x 的内部 fork**。这个 fork 在 EP 之上:

- ✅ 沿用 EP 的 `--el-*` CSS 变量前缀(没改)
- ➕ 扩展了 `--cv-message-*-border-color` 等 `--cv-*` 自定义变量
- 🔧 配套了 5 个 SCSS 补丁,做 EP 没做的布局细节调整:
  - `padding` / `font-size` / `line-height` 精确重写
  - 部分 `!important` 强制覆盖
  - 表格 / 抽屉 / 对话框 / 输入框 / 分页等组件的**尺寸细节**

## 这些补丁不是我们 skill 生成的

- ❌ **不在**我们 skill 的覆盖范围
- ❌ **不能**用设计稿生成
- ✅ 只能**保留**为中创组件库的固定样式

**原因**: 我们 skill 的边界 = "**只换视觉变量,不动布局**"(颜色 / 字号 / 圆角 / 阴影 / 字体色),
布局细节(padding / line-height / `!important` 重写)**不在**我们范围内。

## 集成方式

**方式 A: 从 skill 仓库直接复制到项目(推荐)**

```bash
# 装完 skill 后,把这些文件复制到项目 styles/ 目录
cp -r ~/.trae-cn/mcps/.../ui-design-system-extractor/resources/zhongchuang/ \
     ~/my-app/src/styles/zhongchuang/

# main.ts
import './styles/zhongchuang/_element.scss'   // 入口,会自动 @use 其他 4 个
```

**方式 B: 直接 import skill 仓库里的资源(开发期可,生产期建议复制)**

```ts
// main.ts (开发期,假设 skill 装在 ~/.trae-cn)
import '~/.trae-cn/mcps/.../resources/zhongchuang/_element.scss'
```

## 与 override 加载顺序

```ts
// main.ts (关键: 顺序固定)
import 'element-plus/dist/index.css'                          // 1. EP 库 CSS
import './ui-theme/tokens/theme.css'                            // 2. 我们 token
import './ui-theme/overrides/zhongchuang-theme-override.css'    // 3. CSS 变量覆盖
import './styles/zhongchuang/_element.scss'                    // 4. 中创 scss 布局补丁
```

- 步骤 3 改**视觉变量**(色/字号/圆角/阴影) — 由我们 skill 生成
- 步骤 4 改**布局细节**(padding/line-height/header 高度) — 由中创固定提供

两者**互补不冲突**。

## 这些文件是只读的吗

**是**。这些是**中创组件库的源码**快照,不是我们 skill 的产物。

- ✅ 可以用 — 直接复制到项目,作为中创组件库的一部分
- ❌ 不要改 — 改了会破坏中创组件库在公司里的一致性
- ❌ 不要在 issue 里讨论补丁的修改建议 — 这是中创组件库维护者的事,不是 skill 维护者的事

## 升级说明

当中创组件库升级时(EP 大版本升级、中创内部版本号变更),本目录的文件需要由**项目组**重新同步:

1. 从中创组件库最新仓库 pull 这 5 个文件
2. 替换本目录(同提交一次 PR,标明中创版本号)
3. 用户重新 `npx skills update` 即可拿到最新

我们 skill 不自动升级这些文件 — 它们是中创组件库的一部分,不是 skill 的一部分。

## 目录

```
resources/zhongchuang/
├── README.md               # 本文件
├── _element.scss           # 入口 (@use 其他 4 个)
├── _avatar.scss            # 头像
├── _message.scss           # 消息
├── _popconfirm.scss        # 气泡确认
└── _tooltip.scss           # 弹出层
```
