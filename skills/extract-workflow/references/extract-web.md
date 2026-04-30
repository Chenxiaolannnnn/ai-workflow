# 从参考网页提取资产

从任意参考网站 URL 中提取视觉风格，整理为项目视觉规范。

## 角色定位

你将作为 **产品经理 × 设计分析师（复合型）**，从参考网站中提取视觉语言，整理为本项目可引用的视觉规范文档。

## 核心流程

获取 URL -> 确认提取范围 -> 执行提取 -> 整理视觉规范 -> 写入项目目录

### 步骤 1：确认提取范围

获取用户提供的网页 URL 后，向用户确认：

```text
收到参考网址：[URL]

我可以从这个页面提取以下内容：
1. 视觉规范（配色、字体、间距、圆角、阴影、交互风格）→ 写入 src/themes/visual-spec.md
2. 页面结构截图（作为设计参考存档）→ 写入 src/docs/assets/

默认只提取视觉规范，如需截图存档请告知。
```

### 步骤 2：提取页面数据

运行提取脚本获取设计令牌与截图：

```bash
# 提取设计令牌 + 截图
node extract-workflow/scripts/web-extract.mjs <URL> --theme --screenshot --scroll

# 如需提取页面内容（用于了解信息结构）
node extract-workflow/scripts/web-extract.mjs <URL> --all --scroll --viewport 1440x900
```

产出：
- `theme.json` — CSS 设计令牌（配色、字体、间距、圆角、阴影、动画）
- `screenshot.png` — 页面截图
- `content.md` — 页面文本结构（可选）

### 步骤 3：整理视觉规范

从 `theme.json` 中提炼关键设计信息：

**配色提炼原则：**
- 取频次最高的背景色、文字色、主色、边框色
- 若存在 CSS 变量（`cssVariables` 字段），优先使用变量名，它们反映了原设计系统的意图
- 语义色（成功/警告/错误）从按钮、状态标签的颜色中归纳

**字体提炼原则：**
- 主字体取 `typography.families[0]`
- 归纳标题/正文/辅助文字的字号、字重、行高层级

**间距与圆角：**
- 取出现频率最高的 5 个间距值，构成间距体系
- 圆角取主要元素（按钮、卡片、输入框）的实际值

**交互风格：**
- 从 `transitions` 中归纳悬停动效时长与缓动函数
- 记录主要交互反馈约定（颜色变化、阴影变化等）

按 `../rules/output-spec.md` 的格式，将提炼结果写入 `src/themes/visual-spec.md`。

### 步骤 4：截图存档（若用户需要）

将参考截图保存到 `src/docs/assets/ref-<domain>.png`，便于后续设计时对照参考。

### 步骤 5：交付总结

```text
✅ 提取完成。

已写入以下文件：
- src/themes/visual-spec.md（视觉规范，后续创建原型时自动引用）
- src/docs/assets/ref-xxx.png（参考截图，如有）

视觉规范已可被 create-workflow 中的原型创建流程引用。
```

## 注意事项

- `theme.json` 是统计结果，需人工判断过滤噪声（图标颜色、装饰性元素等）
- 若页面有懒加载内容，使用 `--scroll` 确保内容充分渲染后再提取
- 提取的是参考风格，不是逐像素复刻，最终 visual-spec.md 应描述"规律"而非"堆砌数值"

## 参考

`../rules/output-spec.md` | `../SKILL.md`
