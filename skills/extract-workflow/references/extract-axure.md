# 从 Axure 原型提取资产

从 Axure 原型链接中提取视觉规范与示例数据。

## 角色定位

你将作为 **产品经理 × 设计分析师（复合型）**，从 Axure 原型中读取设计信息，整理为项目可直接使用的资产文档。

## 核心流程

获取链接 -> 确认提取范围 -> 执行提取 -> 整理产物 -> 写入项目目录

### 步骤 1：确认提取范围

获取用户提供的 Axure 原型链接后，首先运行站点地图提取，了解原型规模：

```bash
node extract-workflow/scripts/axure-extract.mjs <AXURE_URL> --all
```

拿到 `sitemap.json` 后，向用户确认本次提取范围：

```text
已获取站点地图，共 [X] 个页面，模块包括：[列出主要模块]。

我可以提取以下内容，请告诉我需要哪些：
1. 视觉规范（配色、字体、间距、组件风格）→ 写入 src/themes/visual-spec.md
2. 示例数据（页面中的列表字段与示例内容）→ 写入 src/data/

默认全部提取，如只需要其中部分请告知。
```

### 步骤 2：提取截图与设计令牌

选取 3-5 个视觉代表性页面（首页、登录页、主功能页），提取截图与设计令牌：

```bash
node extract-workflow/scripts/axure-extract.mjs <AXURE_URL> --pages <page1>,<page2>,<page3>
```

每个页面产出：
- `screenshot.png` — 页面截图，用于视觉分析
- `theme.json` — 设计令牌（配色、字体、间距、圆角）

### 步骤 3：按需提取示例数据

如需提取示例数据，追加 `--advanced` 参数：

```bash
node extract-workflow/scripts/axure-extract.mjs <AXURE_URL> --pages <target> --advanced
```

额外产出：
- `content.md` — 页面文本内容，用于识别字段与示例数据

### 步骤 4：整理并写入产物

根据提取结果，按 `../rules/output-spec.md` 的格式规范整理并写入以下文件：

**视觉规范**（若用户需要）：
- 合并多页面 `theme.json`，提炼主色、字体、间距、圆角、组件风格
- 输出到 `src/themes/visual-spec.md`

**示例数据**（若用户需要）：
- 从 `content.md` 识别列表页、表单页中的字段与示例内容
- 跨页面合并同类字段，统一命名
- 每个数据对象输出一个 `src/data/<name>.md`

### 步骤 5：交付总结

提取完成后告知用户：

```text
✅ 提取完成。

已写入以下文件：
- src/themes/visual-spec.md（视觉规范，后续创建原型时自动引用）
- src/data/xxx.md（示例数据，原型创建时按需复用）

如需基于这份原型创建 HTML 原型，可直接使用 create-workflow 技能，视觉规范与示例数据会被自动检索复用。
```

## 注意事项

- 资源获取禁止批量并发，必须等一个完成后再获取下一个
- 多页面 theme.json 存在冲突时，以核心业务页面为准，并在 visual-spec.md 中注明
- 不确定的字段或业务规则，显式标记"待确认"，不凭空推断

## 参考

`../rules/output-spec.md` | `../SKILL.md`
