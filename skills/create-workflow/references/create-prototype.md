# 创建原型 / 组件

创建一个新的交互式 HTML 原型页面或 UI 组件的标准流程。

## 角色定位

你将作为 **产品经理 × 交互设计师（复合型）**，协助用户完成原型或组件的创建。产出物为带交互效果的 HTML 文件，面向产品评审与演示，不涉及工程开发。

## 核心流程

收到用户需求 -> 阅读规范 -> 需求对齐 -> 设计（spec.md） -> 产出 HTML 原型 -> 验收

### 步骤 1：阅读规范文档

**必须完整阅读以下文档**，不可跳过：

1. **`/rules/design-guide.md`** 设计流程
   - 资料收集、业务场景识别、数据来源查找、内容规划、视觉设计
2. **`/rules/visual-spec.md`** 视觉规范入口
   - 指向 `src/themes/visual-spec.md`，读取项目已有的视觉规范
3. **`/rules/component-source.md`** 组件来源与 antd MCP 策略
   - 决定原型/组件中具体 UI 组件从哪里来
4. **`./templates/spec-template.md`** 规格文档模板
   - 使用此模板生成 `spec.md`

### 步骤 2：需求对齐

判断用户提供的信息是否足够开始设计，按以下情况处理：

- **已有 `spec.md`**：直接跳到步骤 3，不需要重新对齐
- **需求已明确**（用户描述了页面/组件目标、功能点、使用场景）：直接进入步骤 3
- **需求不足**：使用以下模板补齐，等待用户回复后再继续

```text
收到，准备创建原型或组件。

请详细描述您的需求：
```

### 步骤 3：检查示例数据与已有组件

**示例数据**：检查 `src/data/` 目录：
- 若有与当前页面相关的示例数据文件，直接引用其字段定义与示例内容
- 若无，在 spec.md 中自行定义所需字段，并在完成原型后建议用户是否需要单独创建数据文档

**已有组件**：检查 `src/components/` 目录：
- 扫描已有组件的 `index.html`，识别与当前页面功能匹配的组件（如导航栏、表格、弹窗、表单等）
- 有匹配的组件时，提取其 HTML 结构、CSS 样式、JavaScript 逻辑，内联复用到新原型中，不重新实现
- 复用时保持组件行为与原实现一致，确保交互风格统一

**外部组件库**：若 `src/components/` 中无匹配组件，按 `rules/component-source.md` 决定来源（默认走 antd MCP）。

### 步骤 4：设计与规格文档

**必须先写入 `spec.md` 文件，再产出 HTML，不可跳过此步骤。**

- 根据用户需求、视觉规范和示例数据完成布局与交互方向设计
- 使用 `./templates/spec-template.md` 模板生成并**写入**规格文档文件
- 在 spec.md 的"组件清单"章节列出本次用到的 antd 组件（或其他来源组件），每项注明来源
- spec.md 写入完成后，告知用户规格文档已就绪，再继续产出 HTML

### 步骤 5：产出 HTML 原型

- 根据 spec.md 实现带交互效果的 HTML 原型
- 所有样式内联或写在同一文件内，确保单文件可独立打开预览
- 如果原型内容复杂，可按不同组件及区域分多次实现
- 交互效果使用原生 JavaScript 实现（点击、展开、切换、表单反馈等）
- 严格遵循 `src/themes/visual-spec.md` 中的视觉规范，保持与其他原型风格一致
- 涉及 UI 组件时，按 `rules/component-source.md` 的策略选型与实现

## 输出文件

如果目标是原型：

- `src/prototypes/<page-name>/spec.md` - 规格文档
- `src/prototypes/<page-name>/index.html` - 交互原型

如果目标是组件：

- `src/components/<component-name>/spec.md` - 规格文档
- `src/components/<component-name>/index.html` - 组件演示

## 参考

`rules/design-guide.md` | `rules/visual-spec.md` | `rules/component-source.md`
