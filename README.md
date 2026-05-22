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

### auto-capture

自动为用户手册中的截图占位符捕获真实截图，通过交互式导航运行中的应用程序完成。

采用**快照-分析-操作**循环——读取当前界面、判断点击目标、验证操作结果，而非盲目执行脚本。

四种截图模式：

| 模式 | 平台 | 技术方案 |
|------|------|----------|
| 浏览器模式 | Web 应用 | Playwright MCP |
| Windows 原生模式 | WPF / WinForms | pywinauto |
| macOS 原生模式 | Cocoa / AppKit | osascript (AppleScript) |
| 引导式桌面模式 | Linux / 回退方案 | 逐步引导用户操作 |

**触发关键词：** `自动截图`、`截图`、`填充截图`、`auto capture`、`take screenshots`

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
2. /auto-capture           →  自动导航应用，用真实截图填充占位符
3. /generating-html-manual →  转换为精美的独立 HTML 页面
```

## 许可证

MIT
