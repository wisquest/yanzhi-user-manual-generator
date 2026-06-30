# 研知教育科技 前端配色规范

> 基于 LOGO 主题色 `#657eae`（蓝灰）与 `#ed6c00`（橙），面向讲义 HTML 页面的完整配色系统。

## 设计原则

| 原则 | 说明 |
|------|------|
| 品牌一致性 | 所有页面元素源自 LOGO 两个核心色，确保视觉统一 |
| 双色平衡 | 蓝灰作骨架（标题、边框、链接），橙色作强调（CTA、标注、进度） |
| 层次清晰 | 通过明度梯度区分主次，避免视觉噪音 |
| 可读性优先 | 正文保持高对比度，背景保持低饱和 |

蓝灰 `#657eae` 负责页面结构——标题装饰线、h3 左边框、链接色、blockquote 边框、表格 thead。橙色 `#ed6c00` 负责吸引注意力——按钮、关键标注、进度条、重要 callout。两者在 Hero 渐变中交汇，形成"专业 × 活力"的品牌印象。

---

## 1. 核心色板

### 1.1 主色蓝灰系列

| Token | 色值 | 用途 |
|-------|------|------|
| `--primary-900` | `#3d5580` | Hero 渐变深端、链接 hover、文字强调 |
| `--primary-700` | `#657eae` | 标题装饰线、链接色、h3 左边框、blockquote 边框 |
| `--primary-500` | `#8a9ec5` | 次级文字、图标、表格 thead 渐变浅端 |
| `--primary-200` | `#e8edf5` | blockquote 背景、行内代码背景、信息 callout 背景 |
| `--primary-100` | `#f4f6fa` | 表格斑马纹、卡片 hover 背景 |

### 1.2 强调色橙系列

| Token | 色值 | 用途 |
|-------|------|------|
| `--accent-900` | `#c55800` | 按钮 hover、强调交互深色 |
| `--accent-700` | `#ed6c00` | CTA 按钮、重要标注、进度条、callout 左边框 |
| `--accent-500` | `#f5a623` | 次级强调、标签背景、表格 thead 渐变替代 |
| `--accent-200` | `#fff3e6` | 橙色 callout 背景、浅色标注底色 |

### 1.3 中性色阶

| Token | 色值 | 用途 |
|-------|------|------|
| `--neutral-900` | `#1a1a2e` | 页面主标题、正文强调 |
| `--neutral-700` | `#2d3748` | 正文段落文字 |
| `--neutral-500` | `#4a5568` | h4、caption、辅助文字 |
| `--neutral-400` | `#718096` | figcaption、占位文字 |
| `--neutral-200` | `#e2e8f0` | 分割线、表格边框、列表项下划线 |
| `--neutral-100` | `#f5f7fa` | 页面底色、大面积浅色背景 |

---

## 2. 语义色

### 2.1 Callout 组件配色

| 语义类型 | 背景色 | 左边框色 | 标题文字色 | CSS 类名 |
|----------|--------|----------|------------|----------|
| 信息（Tip） | `#e8edf5` | `#657eae` | `#3d5580` | `.callout-tip` |
| 警告（Warning） | `#fff8ed` | `#f5a623` | `#7a5a12` | `.callout-warning` |
| 成功（Progress） | `#eafbef` | `#38a169` | `#276749` | `.callout-progress` |
| 危险（Danger） | `#fef2f2` | `#e53e3e` | `#9b2c2c` | `.callout-danger` |
| 橙色标注 | `#fff3e6` | `#ed6c00` | `#8b4500` | `.callout-accent` |

### 2.2 语义色变量

```css
--success-bg: #eafbef; --success-border: #38a169; --success-text: #276749;
--warning-bg: #fff8ed; --warning-border: #f5a623; --warning-text: #7a5a12;
--danger-bg: #fef2f2;  --danger-border: #e53e3e;  --danger-text: #9b2c2c;
--info-bg: #e8edf5;    --info-border: #657eae;    --info-text: #3d5580;
```

---

## 3. 渐变方案

| 名称 | 值 | 用途 |
|------|------|------|
| Hero 渐变 | `linear-gradient(135deg, #3d5580 0%, #657eae 100%)` | 页面顶部 Hero 区域 |
| 表头渐变 | `linear-gradient(135deg, #657eae, #8a9ec5)` | 表格 thead 背景 |
| 按钮渐变 | `linear-gradient(135deg, #ed6c00, #f5a623)` | CTA 按钮背景 |
| 橙蓝装饰 | `linear-gradient(135deg, #657eae 0%, #ed6c00 100%)` | 特殊装饰元素（慎用，仅限品牌展示区） |

---

## 4. 组件级配色对照

以下对照表将颜色映射到现有 HTML 讲义页面的具体 CSS 规则。

### 4.1 Hero 区域

| CSS 规则 | 属性 | 色值/变量 |
|----------|------|-----------|
| `.hero` | `background` | `var(--gradient-hero)` |
| `.hero .chapter-label` | `background` | `rgba(255,255,255,0.15)` |

### 4.2 内容区标题

| CSS 规则 | 属性 | 色值/变量 |
|----------|------|-----------|
| `.content h1` | `font-size` | `1.2rem` |
| `.content h1` | `font-weight` | `700` |
| `.content h1` | `color` | `var(--neutral-900)` |
| `.content h1` | `margin` | `0 0 16px` |
| `.content h2` | `border-bottom` | `3px solid var(--primary-700)` |
| `.content h3` | `border-left` | `4px solid var(--primary-700)` |
| `.content h3` | `color` | `var(--neutral-700)` |
| `.content h4` | `color` | `var(--neutral-500)` |
| `.content em` | `color` | `var(--primary-700)` |

### 4.3 目录

| CSS 规则 | 属性 | 色值/变量 |
|----------|------|-----------|
| `.toc h2` | `color` | `var(--primary-700)` |
| `.toc > ol > li > a::before` | `color` | `var(--primary-700)` |
| `.toc li a:hover` | `color` | `var(--primary-700)` |

### 4.4 表格

| CSS 规则 | 属性 | 色值/变量 |
|----------|------|-----------|
| `.content thead` | `background` | `var(--gradient-table)` |
| `.content tbody td` | `border-bottom` | `1px solid var(--neutral-200)` |
| `.content tbody tr:nth-child(even)` | `background` | `var(--primary-100)` |
| `.content tbody tr:hover` | `background` | `var(--primary-200)` |

### 4.5 引用块

| CSS 规则 | 属性 | 色值/变量 |
|----------|------|-----------|
| `.content blockquote` | `border-left` | `4px solid var(--primary-700)` |
| `.content blockquote` | `background` | `var(--primary-200)` |
| `.content blockquote strong` | `color` | `var(--primary-900)` |

### 4.6 代码块

| CSS 规则 | 属性 | 色值/变量 |
|----------|------|-----------|
| `.content pre` | `background` | `#1e1e2e` |
| `.content pre` | `color` | `#cdd6f4` |
| `.content pre .comment` | `color` | `#6c7086` |
| `.content pre .keyword` | `color` | `#cba6f7` |
| `.content pre .string` | `color` | `#a6e3a1` |
| `.content pre .command` | `color` | `#89b4fa` |
| `.content code`（行内） | `background` | `var(--primary-200)` |
| `.content code`（行内） | `color` | `#4a5f8a` |

### 4.7 Callout 组件

| CSS 类名 | 背景 | 左边框 | 图标色 | 标题色 |
|----------|------|--------|--------|--------|
| `.callout-tip` | `var(--info-bg)` | `var(--info-border)` | `var(--info-border)` | `var(--info-text)` |
| `.callout-warning` | `var(--warning-bg)` | `var(--warning-border)` | `var(--warning-border)` | `var(--warning-text)` |
| `.callout-progress` | `var(--success-bg)` | `var(--success-border)` | `var(--success-border)` | `var(--success-text)` |
| `.callout-danger` | `var(--danger-bg)` | `var(--danger-border)` | `var(--danger-border)` | `var(--danger-text)` |
| `.callout-accent` | `var(--accent-200)` | `var(--accent-700)` | `var(--accent-700)` | `#8b4500` |

---

## 5. 完整 CSS 变量定义

将以下代码复制到 HTML 页面的 `<style>` 标签顶部即可启用整个配色系统：

```css
:root {
  /* 主色蓝灰 */
  --primary-900: #3d5580;
  --primary-700: #657eae;
  --primary-500: #8a9ec5;
  --primary-200: #e8edf5;
  --primary-100: #f4f6fa;

  /* 强调色橙 */
  --accent-900: #c55800;
  --accent-700: #ed6c00;
  --accent-500: #f5a623;
  --accent-200: #fff3e6;

  /* 中性色 */
  --neutral-900: #1a1a2e;
  --neutral-700: #2d3748;
  --neutral-500: #4a5568;
  --neutral-400: #718096;
  --neutral-200: #e2e8f0;
  --neutral-100: #f5f7fa;

  /* 语义色 */
  --success-bg: #eafbef; --success-border: #38a169; --success-text: #276749;
  --warning-bg: #fff8ed; --warning-border: #f5a623; --warning-text: #7a5a12;
  --danger-bg: #fef2f2;  --danger-border: #e53e3e;  --danger-text: #9b2c2c;
  --info-bg: #e8edf5;    --info-border: #657eae;    --info-text: #3d5580;

  /* 渐变 */
  --gradient-hero: linear-gradient(135deg, #3d5580 0%, #657eae 100%);
  --gradient-table: linear-gradient(135deg, #657eae, #8a9ec5);
  --gradient-btn: linear-gradient(135deg, #ed6c00, #f5a623);
}
```

---

## 6. 迁移对照表

从现有页面（`#667eea` / `#764ba2` 紫色系）迁移到品牌色系的替换清单：

| 旧色值 | 新色值/变量 | 涉及位置 |
|--------|------------|----------|
| `#667eea` | `var(--primary-700)` / `#657eae` | Hero 渐变、h2 底线、h3 左边框、链接色、blockquote 边框、TOC 高亮、行内 code 背景 |
| `#764ba2` | `var(--primary-900)` / `#3d5580` | Hero 渐变深端、表格 thead 渐变深端 |
| `#5a4fcf` | `var(--primary-900)` / `#3d5580` | blockquote strong、行内 code 文字 |
| `#eef0f5` | `var(--primary-200)` / `#e8edf5` | 代码行内背景、TOC 分割线 |
| `#f8faff` | `var(--primary-100)` / `#f4f6fa` | 表格斑马纹 |
| `#eef1ff` | `var(--primary-200)` / `#e8edf5` | 表格 hover 背景 |
| `#f0f3ff` | `var(--primary-200)` / `#e8edf5` | blockquote 背景 |

---

## 7. 色值推导说明

### 7.1 主色蓝灰色阶

以 `#657eae` 为基准：

- **加深**：降低明度 20% 得到 `#3d5580`（用于 hover、渐变深端）
- **提亮**：增加明度 20% 得到 `#8a9ec5`（用于辅助文字、渐变浅端）
- **极浅**：饱和度降至 15%，明度提升到 92% 得到 `#e8edf5`（用于背景底色）
- **近白**：饱和度降至 8%，明度提升到 97% 得到 `#f4f6fa`（用于斑马纹）

### 7.2 强调色橙色阶

以 `#ed6c00` 为基准：

- **加深**：降低明度 20% 得到 `#c55800`（用于按钮 hover）
- **提亮**：增加明度 + 降低饱和度得到 `#f5a623`（用于次级强调）
- **极浅**：饱和度降至 15%，明度提升到 95% 得到 `#fff3e6`（用于 callout 背景）

### 7.3 中性色

中性色与品牌色保持色温一致——`#1a1a2e` 偏蓝调，与蓝灰主色同色温，避免暖调中性色与冷调品牌色冲突。

---

## 8. 可访问性

| 组合 | 对比度 | WCAG 等级 | 备注 |
|------|--------|-----------|------|
| `#1a1a2e` 正文 / `#ffffff` 白底 | 14.3:1 | AAA | 所有正文文字 |
| `#2d3748` 次正文 / `#ffffff` 白底 | 10.7:1 | AAA | 段落文字 |
| `#ffffff` 白字 / `#3d5580` 深底 | 6.2:1 | AA | Hero 区域文字 |
| `#ed6c00` 橙色 / `#ffffff` 白底 | 3.1:1 | 未达标 | 仅用于装饰性元素，不用于正文 |
| `#ed6c00` 橙色 / `#1a1a2e` 深底 | 4.6:1 | AA Large | 按钮文字需用深底或白字 |
