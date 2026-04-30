# 组件来源与 antd MCP 策略

本文档定义原型或组件创建时 UI 组件的来源优先级，以及 antd MCP 的使用方式。

## 📋 适用范围

仅作用于**创建原型**或**创建组件**流程中，涉及具体 UI 元素（按钮、表单、表格、菜单、弹窗、日期选择器等）的选型与实现。

## 🧭 组件来源优先级

按以下顺序判断，命中即停：

1. **用户明确指定的组件库或实现方式** → 严格遵循
2. **`src/components/` 中已有的同类组件** → 直接内联复用（见 `create-prototype.md` 步骤 3）
3. **antd（Ant Design）** → 通过 antd MCP 查询规范与实现（默认）
4. **原生 HTML + CSS 自行实现** → 仅当以上来源都不适用时

## 🔌 antd MCP 使用方式

### 何时使用

以下任一情况触发：
- 用户未指定组件库，且原型/组件包含标准 UI 元素
- 需要参考某个组件的 props、样式令牌、交互状态
- 需要组件的官方示例代码作为实现参考

### 可用工具

antd MCP 提供以下查询工具：

| 工具 | 用途 |
| --- | --- |
| `antd_list` | 列出所有可用组件（按名称/分类/描述） |
| `antd_info` | 查询组件的 props、类型、默认值（含 `detail` 参数可取完整信息） |
| `antd_doc` | 获取组件的完整 markdown 文档 |
| `antd_demo` | 获取组件的官方示例代码（列表 / 指定 demo） |
| `antd_token` | 查询设计令牌（全局 / 组件级） |
| `antd_semantic` | 查询组件的 `classNames` / `styles` 语义化定制结构 |
| `antd_changelog` | 查询版本变更或 API diff |

### 典型查询顺序

1. 不确定选哪个组件 → `antd_list`
2. 选定组件后查接口 → `antd_info`
3. 需要实现参考 → `antd_demo`（先无参列出所有 demo，再取具体的）
4. 需要配色/间距对齐 → `antd_token`

### 与 `src/themes/visual-spec.md` 的关系

- **样式优先级**：`visual-spec.md`（项目主题） > antd 默认样式
- 从 antd 取组件的**结构和交互规范**，用项目 `visual-spec.md` 覆盖**视觉样式**（颜色、圆角、字号、间距）
- 若项目 `visual-spec.md` 未定义某项，fallback 到 `antd_token` 返回的默认值

## ⚙️ 未安装时的引导

如果当前环境中 **antd MCP 工具不可用**（调用 `antd_list` 等工具报"工具不存在"或类似错误），按以下流程引导用户安装：

```text
检测到 antd MCP 未安装。为了按 Ant Design 规范参考组件，建议安装。

安装参考：https://ant.design/docs/react/mcp-cn

安装完成后请重新发起当前请求，或告诉我"已装好"继续。

如果你暂不想安装，可以告诉我：
- "跳过 antd"：本次用原生 HTML + CSS 实现
- "用 xxx 组件库"：我将按你指定的库处理
```

- 用户确认跳过时，降级到"原生实现"，并在 spec.md 中注明"组件来源：原生（未使用 antd）"
- 用户指定其他库时，按用户指定执行
- 不要自行尝试安装 MCP（安装需要改 Claude 配置，由用户在自己环境执行）

## 📝 在 spec.md 中的记录

创建的 `spec.md` 需在"组件清单"章节列出本次使用的组件，每项包含：

| 组件名 | 来源 | antd 组件（如适用） | 备注 |
| --- | --- | --- | --- |
| 例：主操作按钮 | antd | `Button` (type="primary") | 复用 visual-spec 主色 |
| 例：自定义数据卡片 | 原生 | - | 无 antd 对应项 |

## ✅ 检查清单

- [ ] 已按优先级判断组件来源，未无理由跳过 antd 默认策略
- [ ] 使用 antd 组件时，已通过 MCP 查询过 `antd_info` 或 `antd_demo`
- [ ] 样式已与 `src/themes/visual-spec.md` 对齐，未原样照搬 antd 默认视觉
- [ ] spec.md 中已列出组件清单并标注来源
- [ ] antd MCP 不可用时，已向用户明确告知并给出处理方案
