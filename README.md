# yanzhi-user-manual-generator

一个 Claude Code 插件，用于从项目源码或需求文档生成美观的用户手册。

## 安装

### 第一步：添加 Marketplace

在 Claude Code 中运行：

```
/plugin marketplace add zhang-songhan/yanzhi-user-manual-generator
```

### 第二步：安装插件

```
/plugin install yanzhi-user-manual-generator
```

## Skills

### writing-user-manual

根据项目源码或需求文档，生成结构化的 Markdown 用户手册。

- 自动分析代码库，提取功能模块和业务流程
- 生成包含产品概览、功能说明、操作步骤和截图占位符（`【图X：...】`）的完整手册
- 支持**新建**模式和**更新**模式（刷新已有手册）
- **Web 前端项目**（React/Vue/Angular 等）：可自动截取真实页面截图，无需手动创建
- 非 Web 项目（CLI/桌面/移动端）：生成截图占位符，由用户手动填充

**触发关键词：** `用户手册`、`使用手册`、`用户指南`、`user manual`、`user guide`、`更新手册`

### generating-html-manual

将 Markdown 用户手册转换为带样式的独立 HTML 页面，包含侧边栏导航和公司品牌标识。**HTML 代码生成采用 superpowers TDD（测试驱动开发）RED-GREEN-REFACTOR 流程**，确保结构、样式和交互行为经过验证。

功能特性：

- 左侧目录导航，自动追踪当前阅读位置（支持折叠/展开，状态持久化）
- 响应式布局（桌面、平板、手机）
- 页头页脚展示公司 Logo
- 返回顶部按钮
- 锚点滚动偏移（防止固定页头遮挡标题）
- 打印优化样式
- Mermaid 流程图实时渲染
- **截图说明自动缩短**：原 Markdown 中的长截图描述在 HTML 中自动精简为 10 字以内的简洁标签
- 输出单个 `index.html` + 媒体资源文件夹，可直接分发

**触发关键词：** `生成HTML手册`、`转HTML`、`HTML版本`、`HTML manual`、`convert to HTML`

## 典型工作流

### Web 前端项目（自动截图）

```
1. /writing-user-manual       →  生成用户手册 → 自动截取真实页面截图
2. /generating-html-manual    →  转换为精美的独立 HTML 页面（含截图）
```

### 非 Web 项目（手动截图）

```
1. /writing-user-manual       →  生成带截图占位符的 Markdown 用户手册
2. 手动截图并保存到 screenshots/  →  按截图清单创建真实截图
3. /generating-html-manual    →  转换为精美的独立 HTML 页面（含截图）
```

## 许可证

MIT
