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
/plugin yanzhi-user-manual-generator
```

## Skills

### writing-user-manual

根据项目源码或需求文档，生成结构化的 Markdown 用户手册。

- 自动分析代码库，提取功能模块和业务流程
- 生成包含产品概览、功能说明、操作步骤和截图占位符（`【图X：...】`）的完整手册
- 支持**新建**模式和**更新**模式（刷新已有手册）

**触发关键词：** `用户手册`、`使用手册`、`用户指南`、`user manual`、`user guide`、`更新手册`

### generating-html-manual

将 Markdown 用户手册转换为带样式的独立 HTML 页面，包含侧边栏导航和公司品牌标识。

功能特性：

- 左侧目录导航，自动追踪当前阅读位置
- 响应式布局（桌面、平板、手机）
- 页头页脚展示公司 Logo
- 返回顶部按钮
- 打印优化样式
- 输出单个 `index.html` 文件，可直接分发

**触发关键词：** `生成HTML手册`、`转HTML`、`HTML版本`、`HTML manual`、`convert to HTML`

## 典型工作流

```
1. /writing-user-manual    →  生成带截图占位符的 Markdown 用户手册
2. /generating-html-manual →  转换为精美的独立 HTML 页面
```

## 许可证

MIT
