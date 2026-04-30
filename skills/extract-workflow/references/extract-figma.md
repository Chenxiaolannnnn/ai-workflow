# 从 Figma 设计稿提取资产

从 Figma 设计稿提取视觉规范，写入项目主题文件。

## 角色定位

你将作为 **产品经理 × 设计分析师（复合型）**，从 Figma 中读取设计信息，整理为项目可直接使用的视觉规范文档。

## 核心流程

识别输入 -> 确认提取范围 -> 读取数据 -> 整理产物 -> 写入项目目录

### 步骤 1：识别输入并分流

看用户提供什么，直接路由：

| 输入 | 路径 |
| --- | --- |
| 粘贴 CSS 文本 / `.css` 文件 | **A：CSS 提取** |
| Figma 文件 URL（`figma.com/file/*` 或 `figma.com/design/*`） | **询问 B1 或 B2** |
| 只提到 Figma、无具体内容 | 引导用户三选一：粘贴 CSS / 提供 .css 文件 / 提供 Figma URL |

URL 场景下，先问：

```text
收到 Figma 链接。两种提取方式：

1. 当普通网页抓取（快速）
   - 只对公开的 Figma Sites / Prototype 发布链接有效
   - 编辑器 URL 需登录、Canvas 内容无法直接提取
2. Figma 官方 MCP（推荐，结构化）
   - 需安装：https://help.figma.com/hc/en-us/articles/32132100833559
   - 未安装我会引导

选哪种？
```

### 步骤 2：确认提取范围

```text
我会从 Figma 数据中提炼以下内容写入 src/themes/visual-spec.md：
- 配色（主色、背景、文字、边框、语义色）
- 字体（字体族、字号层级、字重、行高）
- 间距与圆角
- 阴影
- 可识别的组件风格（按钮、卡片、输入框等）

[如走 MCP] 还可额外提取：命名 styles、variables（设计令牌）、组件清单

确认即开始，或告诉我要先聚焦哪一部分。
```

### 步骤 3：读取数据

**路径 A（CSS）**
- 从粘贴文本或 `.css` 文件内容解析
- 只取"风格"属性（color / font / spacing / radius / shadow 等），忽略布局绝对值（`position: absolute`、`left`、`top`）
- CSS 变量（`--foo: #xxx`）优先使用变量名，反映原设计意图

**路径 B1（网页）**
- 转到 `./extract-web.md` 流程，使用 `web-extract.mjs` 抓取截图与 CSS 变量
- 本文档不再重复网页提取细节

**路径 B2（MCP）**
- 先探测工具列表是否存在 `mcp__figma*` 开头的工具
- 不存在 → 输出引导：
  ```text
  未检测到 Figma MCP。安装步骤：
  1. Figma Desktop → Preferences → Enable Dev Mode MCP Server
  2. Claude 客户端 MCP 配置中添加 Figma MCP
  3. 重启 Claude 后重试
  文档：https://help.figma.com/hc/en-us/articles/32132100833559

  暂不装可改用：粘贴 CSS / 走路径 B1（当网页处理）
  ```
- 存在 → 调用 MCP 拉取 styles、variables、components、selected frame（工具名以运行时为准，不硬编码）

### 步骤 4：整理视觉规范

按 `../rules/output-spec.md` 的视觉规范格式整理，输出到 `src/themes/visual-spec.md`。

提炼要点：
- **配色**：归类主色 / 背景 / 文字（主次）/ 边框 / 语义色；保留原格式（hex 或 rgba）
- **字体**：主字体 + 备选；归纳 3+ 级字号层级（标题/正文/辅助）
- **间距**：高频值归 2-4 档
- **圆角**：归小/中/大
- **阴影**：原样保留，按用途（卡片/弹窗/悬浮）分组
- **组件风格**：按选择器名或 MCP 组件名归纳外观描述

增量策略：文件已存在时合并，冲突项标 `待确认`，不静默覆盖。

### 步骤 5：交付总结

```text
✅ 提取完成。

已写入 src/themes/visual-spec.md：
- 配色：[N 个]
- 字体：[主字体 + N 级层级]
- 间距/圆角/阴影：[简述]
- 组件风格：[识别到的组件]

如需补充其他页面或模块，继续提供 CSS / URL / 走 MCP 即可（增量合并）。
如需基于此创建 HTML 原型，可直接使用 create-workflow 技能，视觉规范会自动对齐。
```

## 注意事项

- 不确定的色值或字段用 `待确认` 标注，不凭空填写
- Figma Copy as CSS 的布局属性（`position`、`left`、`top`、绝对尺寸）不写入规范
- 路径 B2 的 Token / MCP 会话数据不写入任何文件
- 多次提取注意增量合并，不覆盖用户已确认的值

## 参考

`../rules/output-spec.md` | `./extract-web.md` | `../SKILL.md`
