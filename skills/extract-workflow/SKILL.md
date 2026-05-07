---
name: extract-workflow
description: 从 Axure 原型链接、Figma 设计稿（CSS / 网页 / 官方 MCP）或参考网页 URL 中提取视觉规范与示例数据，统一输出到项目资产目录供 create-workflow 使用。
---

# 资料提取工作流

从外部设计资产中提取并整理项目所需的视觉规范与示例数据。

## 核心原则

- 先识别输入来源，再只加载一个最相关的引用文档。
- 提取产物统一输出到项目资产目录，路径规范见 `./rules/output-spec.md`。
- 先与用户确认提取范围再执行，不要一次性执行所有提取动作。
- 提取完成后，主动告知用户哪些产物已可被 `create-workflow` 引用。

## 意图分流

| 用户提供 | 判断依据 | 加载文档 |
| --- | --- | --- |
| Axure 原型链接 | URL 含 `axshare`、`axure.cloud`、`start.html#p=` | `./references/extract-axure.md` |
| Figma 设计稿 | 粘贴 Copy as CSS / `.css` 文件 / Figma 文件 URL / 仅提到 Figma | `./references/extract-figma.md` |
| 普通网页 URL | 非 Axure / 非 Figma 的任意网址 | `./references/extract-web.md` |

## 分流优先级

1. Axure 特征 URL → Axure
2. 任何 Figma 输入 → Figma
3. 其他 URL → 网页
4. 输入不明确 → 使用下方模板询问

## 模糊意图时的首次回复模板

```text
收到，我可以帮你从外部资产中提取设计资料。

请告诉我输入来源：
1. Axure 原型链接
2. Figma 设计稿（粘贴 CSS / .css 文件 / Figma 文件 URL）
3. 参考网站 URL
```

## 提取产物

| 产物类型 | 输出路径 | 被谁使用 |
| --- | --- | --- |
| 视觉规范 | `src/themes/visual-spec.md` | `create-workflow` 所有原型 |
| 示例数据 | `src/data/<name>.md` | `create-workflow` 原型创建（按需复用） |

详细格式见 `./rules/output-spec.md`。

## 引用文档

- `./references/extract-axure.md`
- `./references/extract-figma.md`
- `./references/extract-web.md`
- `./rules/output-spec.md`

