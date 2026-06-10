---
name: generating-html-manual
description: Use when the user has a Markdown user manual and wants to convert it to a styled standalone HTML page with left sidebar navigation, back-to-top button, and company branding. Triggers on keywords like "HTML manual", "convert to HTML", "生成HTML手册", "转HTML", "HTML版本".
---

# Generating HTML Manual

## Overview

Convert a Markdown user manual into a self-contained, styled HTML page with sidebar catalogue navigation, back-to-top button, and company branding. Output is a folder containing the HTML file plus all referenced media assets.

## When to Use

- User provides a Markdown user manual (`.md` file) and wants an HTML version
- User asks to "convert manual to HTML", "generate HTML manual", "转HTML", "生成HTML手册"
- Before generating HTML, you MUST ask the user to select languages (Step 0). Default: Simplified Chinese only if the user confirms it.

## Prerequisites

1. **Markdown user manual** — the `.md` file to convert
2. **Media files** — any images or assets referenced in the markdown (relative paths)

**REQUIRED REFERENCES:**
- Read `color-spec.md` in this skill's directory for the complete color system and design tokens.
- Read `company_style/` in this skill's directory for logo assets.

**DEVELOPMENT METHODOLOGY — TDD Required (RED-GREEN-REFACTOR):**

This skill generates a standalone single HTML webpage. **HTML code generation (Steps 3-4) MUST follow the superpowers TDD cycle:**

1. **RED — Write failing tests first:** Before writing ANY HTML/CSS/JS code, write test assertions for each structure, style, and behavior. Tests must verify:
   - Correct heading hierarchy and ID slugs (anchor IDs are lowercase English words with hyphen separators, e.g., `system-settings-export-path`; Chinese titles are translated to English for anchors)
   - TOC link targets resolve to valid heading IDs
   - Sidebar toggle behavior — menu icon (☰) positioned left of the logo in the header toggles sidebar fold/unfold; sidebar defaults to visible (280px width), content area expands when sidebar is folded; toggle works at all screen sizes
   - Anchor scroll offset prevents fixed header from hiding targets
   - Back-to-top button visibility (hidden at top, visible after scroll > 400px)
   - Sidebar toggle works correctly at all screen sizes (menu icon always visible, fold/unfold animation smooth, content area reflows correctly)
   - Contrast rules (no dark text on dark backgrounds)
   - Media path correctness (all `src`/`href` point to files that exist in `media/`)
   - Print styles hide navigation chrome
2. **GREEN — Write minimal code to pass:** Generate the HTML body, apply the template, wire up interactivity — one test at a time. Each increment of HTML/CSS/JS must correspond to a test that was already written and seen to fail.
3. **REFACTOR — Improve while keeping tests green:** Deduplicate styles, optimize selectors, streamline event handlers, improve semantic markup. Never add new behavior during refactoring.

**REQUIRED SUB-SKILL:** Use `superpowers:test-driven-development` to guide this process — it defines the RED-GREEN-REFACTOR discipline and rationalization countermeasures that keep TDD honest.

**Fallback:** If `superpowers:test-driven-development` is not available, still follow the TDD cycle described above. Do NOT skip testing because the skill is unavailable.

**Iron Law:** No HTML code before its corresponding test exists. Code written before tests → delete it and start over. No exceptions.

## Workflow

```dot
digraph html_manual {
  "Ask user for language selection" [shape=box];
  "Read markdown input" [shape=box];
  "Parse headings for TOC" [shape=box];
  "Identify media references" [shape=box];
  "Generate output folder" [shape=box];
  "Copy media files to output folder" [shape=box];
  "Convert markdown to HTML body" [shape=box];
  "Apply HTML template with sidebar + branding + i18n" [shape=box];
  "Write index.html to output folder" [shape=box];
  "Verify all media paths are valid" [shape=box];
  "Done" [shape=doublecircle];

  "Ask user for language selection" -> "Read markdown input";
  "Read markdown input" -> "Parse headings for TOC";
  "Parse headings for TOC" -> "Identify media references";
  "Identify media references" -> "Generate output folder";
  "Generate output folder" -> "Copy media files to output folder";
  "Copy media files to output folder" -> "Convert markdown to HTML body";
  "Convert markdown to HTML body" -> "Apply HTML template with sidebar + branding + i18n";
  "Apply HTML template with sidebar + branding + i18n" -> "Write index.html to output folder";
  "Write index.html to output folder" -> "Verify all media paths are valid";
  "Verify all media paths are valid" -> "Done";
}
```

### Step 0: Language Selection (MANDATORY)

**Before any HTML generation begins**, you MUST ask the user which languages the HTML manual should support. Present these options:

1. **仅支持简体中文** (Only Simplified Chinese)
2. **支持简体中文、英文、俄语** (Simplified Chinese, English, Russian)
3. **自定义语言** (Custom — user specifies which languages to support)

Save the user's choice as the `LANGS` variable. Format: **comma-separated language codes** (no spaces between items).

| Choice | `LANGS` value |
|--------|---------------|
| 1 (仅中文) | `zh` |
| 2 (中英俄) | `zh,en,ru` |
| 3 (自定义) | User-specified codes, e.g., `zh,ja,ko` or `en,zh` |

Supported language codes: `zh` (Simplified Chinese), `en` (English), `ru` (Russian), `ja` (Japanese), `ko` (Korean), `fr` (French), `de` (German), `es` (Spanish), `pt` (Portuguese), `ar` (Arabic).

**If `LANGS` contains more than one language** (i.e., the string contains a comma), the HTML page MUST include full multi-language switching support. See the [Multi-Language Support](#multi-language-support) section for implementation details.

**If `LANGS` has only one language** (e.g., `zh`), skip all i18n — generate a single-language page as before. The single language becomes the page language and all UI chrome uses that language.

### Step 1: Read and Parse

1. Read the input markdown file
2. Parse all headings (`##` and `###`) to build the sidebar TOC
3. **Detect and strip the `anchor` fenced code block** (```` ```anchor ... ``` ````) — this is git metadata, not user-facing content. Record its presence but exclude it from the HTML body.
4. Scan for media references: `![alt](path)`, `<img src="path">`, `[text](path.pdf)` etc.
5. Scan for mermaid/fenced diagram blocks: ```` ```mermaid ```` code blocks
6. Record the relative paths of all referenced media files

### Step 2: Create Output Folder

1. Create a new folder next to the input markdown file, named `{manual-name}-html/`
2. Create the `media/` directory structure based on language count:
   - **Single-language** (`LANGS` has 1 item, e.g., `zh`): Create one `media/` subfolder
   - **Multi-language** (`LANGS` has >1 item, e.g., `zh,en,ru`): Create `media/{lang}/` subfolder for **each** language in `LANGS` (e.g., `media/zh/`, `media/en/`, `media/ru/`)
3. Copy all referenced media files (screenshots, diagrams, etc.):
   - **Single-language:** Copy to `media/` directly
   - **Multi-language:** Copy each media file to **ALL** language subfolders (e.g., `1-login.png` → `media/zh/1-login.png` + `media/en/1-login.png` + `media/ru/1-login.png`). Screenshots are identical across languages — this isolation ensures each language has its own complete media set.
4. Copy the company logo files from this skill's `company_style/` directory:
   - **Single-language:** Copy to `media/`
   - **Multi-language:** Copy to **ALL** language subfolders (e.g., `media/zh/`, `media/en/`, `media/ru/`)

### Step 3: Convert Markdown to HTML

Convert the markdown content to well-structured HTML following these rules:

| Markdown | HTML Output |
|----------|------------|
| `## Heading` | `<h2>` with `id` attribute (slug) |
| `### Heading` | `<h3>` with `id` attribute (slug) |
| `**bold**` | `<strong>bold</strong>` |
| `> **note**: text` | `<blockquote>` with `.callout-tip` class |
| `> **warning**: text` | `<blockquote>` with `.callout-warning` class |
| `![alt](path)` | `<figure><img><figcaption>` structure, `max-width` capped at 720px |
| `【图X：...】` | `<figure>` with `.screenshot-placeholder` class; placeholder text rendered as small caption **below** the image |
| Tables | Standard `<table>` with `<thead>` and `<tbody>` |
| `1. item` | `<ol><li>item</li></ol>` |
| `- item` | `<ul><li>item</li></ul>` |
| `` `code` `` | `<code>code</code>` |
| Code blocks | `<pre><code>...</code></pre>` |
| ````mermaid` blocks | `<pre class="mermaid">` — render as live diagrams, NOT raw code |

**Skip anchor code block:** The markdown may contain an `anchor` fenced code block at the very beginning of the file (before the title), recording git branch/commit info:

````markdown
```anchor
branch: main
commit: abc123...
message: feat: ...
```
````

**Strip this block entirely from the HTML output.** It is machine-readable metadata for version tracking, not user-facing content. Parse it in Step 1 to detect its presence, then exclude it from the HTML body in Step 3.

**Skip inline TOC sections:** If the markdown contains a "目录"、"Table of Contents"、"内容提要" or similar TOC section (a heading followed by a list of internal links), **omit it from the HTML body**. The sidebar already provides navigation — duplicating the TOC in the content area wastes space and confuses readers.

**Skip screenshot index table:** If the markdown ends with a screenshot index table (a section titled "截图索引"、"截图索引表"、"Screenshot Index" or similar, containing a table that maps screenshot placeholders to file paths), **omit this entire section from the HTML output**. This table is a build-time reference for tracking which screenshots exist — it is not end-user content and does not belong in the published manual.

**Heading ID slugs (锚点名称格式):** Generate anchor IDs as **lowercase English words separated by short dashes** (`-`). Chinese titles must be translated to English for anchor generation. Different levels of headings are also separated by short dashes.

- **`h2` headings**: Translate the heading text to lowercase English, using hyphens between words. Examples: `login-page`, `faq`, `system-settings`, `user-management`
- **`h3` headings**: Use `parent-heading-current-heading` format (parent h2's English anchor - current h3's English text), all lowercase with hyphens. Examples: `system-settings-export-path`, `user-management-role-permissions`, `data-reports-access-logs`
- **`h4+` headings**: Continue chaining: `parent-grandparent-current-heading` format (e.g., `system-settings-security-password-policy`)
- **Do NOT include heading numbers** (like `1.1`, `2.3.1`, `01_`, `1、`, etc.) in anchor names — strip them entirely
- **Do NOT keep Chinese characters** in anchors — translate/transliterate to English first, then convert to lowercase hyphenated form
- Replace spaces with hyphens; remove punctuation marks (colons `:`、`：`, brackets `（）`, quotation marks `""`、`「」`, periods `。`, commas `，`、`,`, slashes `/`)
- Ensure uniqueness by appending `-2`, `-3` etc. for duplicates

**Media path rewrite:** All media references in the HTML must use relative paths:
- **Single-language:** `media/{filename}`
- **Multi-language:** `media/{DEFAULT_LANG}/{filename}` (where `{DEFAULT_LANG}` is the first language code in `LANGS`)

**Screenshot image sizing:** All screenshot images (`<img>` inside `<figure>`) must use a fixed height with wide aspect ratio (5:8 = height:width, matching 1200×1920px source screenshots): `height: 450px; object-fit: contain; object-position: center; max-width: 720px; width: 100%`. The `height: 450px` ensures the browser reserves exactly 450px of vertical space **before** images load, preventing anchor scroll positions from drifting when lazy-loaded images arrive. The `max-width: 720px` matches the 5:8 ratio (450 × 8/5 = 720), giving screenshots a landscape/widescreen display. The `object-fit: contain` preserves aspect ratio without distortion. **Also add explicit `width` and `height` HTML attributes** to each `<img>` tag (obtain actual pixel dimensions via `sips` or similar tool) so the browser can compute the intrinsic aspect ratio even before CSS is applied.

**Image opacity — CRITICAL: All images must be fully opaque.** NEVER apply `opacity` CSS to any `<img>`, `<figure>`, or image container element. Images (screenshots, diagrams, icons, logos) must always render at 100% opacity (`opacity: 1`). Specifically forbidden:
- `opacity: 0.X` or `opacity: X%` on `<img>`, `<figure>`, or any parent that wraps images
- `transition: opacity` that animates images to semi-transparency
- `:hover { opacity: ... }` on images — hover effects on images must NOT change opacity
- Using opacity to create "grayed out" or "disabled" visual states on images
- Any CSS filter or blend mode that reduces image visibility

Rationale: Screenshots and diagrams are informational content, not decorative elements. Semi-transparent images degrade readability, reduce contrast, and make the manual look unprofessional. The page already has a clean design system — opacity tricks are unnecessary and harmful.

**Image placeholder captions:** Placeholder text like `【图X：...】` describes what the image should contain. Render this text as a small caption (`<figcaption>`) **below** the image/placeholder, styled in a smaller font size (`0.85em`) and muted color (`var(--neutral-400)`). The placeholder text is an annotation, not a heading.

**Screenshot description shortening (CRITICAL):** The original Markdown may contain lengthy screenshot descriptions (e.g., `【图1：登录页面全貌，展示渐变背景、公司Logo、玻璃质感登录卡片、用户名输入框、密码输入框、"登录"按钮的整体布局】`). When converting to HTML, **shorten every screenshot caption to 10 characters or fewer** (not counting the prefix `图X：`). The caption must be a concise label, not a detailed description:

| Original Markdown | HTML `<figcaption>` |
|---|---|
| `【图1：登录页面全貌，展示渐变背景、公司Logo、玻璃质感登录卡片】` | `图1：登录页面` |
| `【图2：首页概览，包含顶部导航栏、数据统计卡片、图表区域】` | `图2：首页概览` |
| `【图3：用户管理界面，展示用户列表、搜索框、新增按钮】` | `图3：用户管理` |

**Rule:** Extract only the core page/feature name (≤10 characters after `图X：`). Discard all descriptive detail — the screenshot itself shows what the text describes. The caption is a label, not a replacement for viewing the image.

**Button icon descriptions:** Detect sections that describe button/icon mappings based on two features occurring together in proximity: (1) text containing "图标说明" or "图标列表" or "icon reference", and (2) one or more image links (`![alt](path)` or `<img>`) paired with icon names. When both features are detected, convert the section to a clean **table** with two columns: 图标 (icon image) and 名称 (name). Do NOT assume a specific markdown format (blockquote, list, paragraph, figure) — the detection is content-driven, not format-driven. Use `<img class="icon-inline">` for the icon column, with CSS `height: 1.3em; width: auto` so icons render at text-line height. Example table structure:
```html
<p><strong>按钮图标说明</strong></p>
<table>
  <thead><tr><th>图标</th><th>名称</th></tr></thead>
  <tbody>
    <tr><td><img src="media/icons/xxx.png" alt="名称" class="icon-inline"></td><td>名称</td></tr>
  </tbody>
</table>
```

### Step 4: Apply HTML Template

Generate a complete standalone HTML page with the following structure and design specs. All styles and scripts must be inline — single `index.html` output, with the exception of the Mermaid.js CDN script (see Mermaid & Flowchart Handling section).

#### Page Structure

- **Header** — fixed top bar with menu toggle icon (☰) on the far left, followed by company logo, page title, and version string. The menu icon toggles the sidebar fold/unfold. When multi-language is active (`LANGS` has >1 item), a **language switcher widget** sits in the **top-right corner** of the header (to the right of the version string).
- **Sidebar** — fixed left navigation with TOC. Defaults to visible (280px width). Can be folded/unfolded via the menu icon in the header. When folded, the content area expands to fill the available space.
- **Content** — main body area with max-width 900px, centered. Content area has `margin-left: 280px` when sidebar is visible; expands to full width when sidebar is folded.
- **Footer** — company logo and copyright
- **Back-to-top button** — fixed bottom-right, appears on scroll

#### Template Placeholders

| Placeholder | Source |
|-------------|--------|
| `{{TITLE}}` | First `#` heading or filename |
| `{{VERSION}}` | Version string if found (e.g., "V2.3.0"), otherwise empty |
| `{{CONTENT}}` | Converted HTML body from Step 3 |
| `{{TOC}}` | Generated sidebar TOC from headings |
| `{{DEFAULT_LANG}}` | First language code from `LANGS` (e.g., `zh`) — used in JS i18n init |
| `{{LANGS_LIST}}` | Full `LANGS` string (e.g., `zh,en,ru`) — used to generate language switcher buttons |
| `{{I18N_OBJECT}}` | Complete JavaScript `I18N` object with translations for all languages in `LANGS` |
| `{{LANG_SWITCHER}}` | HTML markup for the language switcher button row (empty string if single-language) |

#### Layout Specs

| Element | Spec |
|---------|------|
| Header height | 64px, fixed top, `z-index: 1000`. Contains menu toggle icon (☰) on the far left, then logo, title, version. When multi-language: language switcher buttons at the far right. |
| Sidebar width | 280px, fixed left, `z-index: 900`. Defaults to visible. Toggled by the header menu icon. Use CSS `transition` on `transform` or `margin-left` for smooth fold/unfold animation. |
| Content max-width | 900px, centered. Content area has `margin-left: 280px` when sidebar is visible; transitions to `margin-left: 0` (or auto-centered) when sidebar is folded. |
| Anchor scroll offset | CSS: `scroll-margin-top: 80px` on all `h2`/`h3`. JS: intercept TOC link clicks, call `scrollIntoView()` with manual offset for the 64px header + 16px breathing room |
| Back-to-top trigger | Scroll > 400px |
| Print | Hide header, sidebar, back-to-top; full-width content |

#### Contrast & Readability Rules

**NEVER use dark text (black, `#000`, `#1a1a2e`, `var(--neutral-900)`) on dark backgrounds.** Any element with a dark or deeply colored background must use light-colored text:

| Background type | Text color |
|----------------|------------|
| Hero gradient (`var(--gradient-hero)`) | `#ffffff` white |
| Table thead gradient (`var(--gradient-table)`) | `#ffffff` white |
| Code blocks (`#1e1e2e`) | `#cdd6f4` (light) |
| Any dark-colored section/div | `#ffffff` white or `var(--primary-200)` |
| CTA buttons (orange gradient) | `#ffffff` white |

**Header and footer bars:** Must use a light/white background because the horizontal company logo has **black text** and would be invisible on dark surfaces.

#### Sidebar Toggle Behavior

The sidebar can be folded/unfolded via the menu icon (☰) in the header:

1. **Menu icon position:** The ☰ icon sits to the **left of the logo** in the header bar. It is always visible.
2. **Default state:** Sidebar visible (unfolded), 280px width. Content area has `margin-left: 280px`.
3. **Folded state:** Sidebar hidden. Content area expands to full width (centered via auto margins, max-width 900px).
4. **Toggle action:** Clicking the menu icon toggles between folded and unfolded states. Use CSS transitions for smooth animation.
5. **localStorage persistence:** Persist the sidebar fold state in `localStorage` so the user's preference is remembered across page loads.
6. **TOC link clicks:** When a TOC link is clicked, scroll to the target heading with proper offset. Do NOT fold the sidebar on link click — the user controls sidebar state explicitly via the menu icon.

#### Anchor Scroll Behavior

TOC link clicks must scroll to the target heading with proper offset to prevent the fixed header from obscuring content:

1. **CSS fallback:** Add `scroll-margin-top: 80px` on all `h2` and `h3` elements. This handles browser-native anchor navigation (`#fragment` in URL).
2. **JavaScript interception:** Attach click handlers to all sidebar TOC links (`<a>` inside `.toc-list`). Prevent default, then:
   - Find the target element by `id` (extracted from `href` attribute)
   - Calculate scroll position: `target.getBoundingClientRect().top + window.pageYOffset - headerHeight - 16`
   - Use `window.scrollTo({ top: position, behavior: 'smooth' })`
   - The offset must account for: 64px header + 16px breathing room = 80px total
3. **Edge cases:**
   - Target element doesn't exist → silently ignore (don't throw)
   - Already at target → no scroll needed
4. **Back-to-top button:** Use the same smooth scroll approach: `window.scrollTo({ top: 0, behavior: 'smooth' })`

### Step 5: Verify

After writing `index.html`:
1. Check all `src="media/..."` references point to files that exist in the output folder
2. If any media files are missing, warn the user with the list of missing files
3. Report the output folder path to the user

## TOC Generation

Build the sidebar TOC from parsed headings:

**Rules:**
- Include only `h2` and `h3` headings in the TOC
- Skip `h1` (it's the title in the header) and `h4+` (too deep for sidebar)
- Use `.toc-h2` class for `##` headings, `.toc-h3` class for `###` headings
- Generate anchor IDs: lowercase English words with hyphen separators, with heading levels chained by hyphens. Chinese titles must be translated to English first (e.g., "系统设置 > 导出路径" → `system-settings-export-path`). See Heading ID slugs in Step 3 for full rules.
- Each TOC item is an `<li>` containing an `<a>` linking to the heading's `id`

## Callout Conversion

Convert markdown blockquotes with specific markers to styled callouts:

| Blockquote starts with | CSS class |
|------------------------|-----------|
| `> **说明**：` or `> **提示**：` or `> **Tip**:` | `.callout-tip` |
| `> **注意**：` or `> **Warning**:` | `.callout-warning` |
| `> **危险**：` or `> **Danger**:` | `.callout-danger` |
| Regular blockquote (no marker) | Default blockquote (blue-gray) |

## Mermaid & Flowchart Handling

**CRITICAL: Never output raw Mermaid code in the HTML.** All mermaid/fenced diagram code blocks must be rendered as live interactive diagrams.

### Detection

Scan the markdown for fenced code blocks with the `mermaid` language tag:

````markdown
```mermaid
graph TD
  A[Start] --> B[End]
```
````

### Conversion

1. Convert each ```` ```mermaid ```` block to `<pre class="mermaid">` containing **only the Mermaid DSL** (no markdown fences)
2. Do NOT wrap in `<code>` — the Mermaid library targets `<pre class="mermaid">` directly
3. Include the Mermaid.js library via CDN in the HTML `<head>`:
   ```html
   <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
   ```
4. Initialize Mermaid at the end of the `<body>`, after all content:
   ```html
   <script>
   mermaid.initialize({ startOnLoad: true, theme: 'default' });
   </script>
   ```

### Styling

- Set `max-width: 100%` on `.mermaid` SVG output to prevent overflow on mobile
- Give `<pre class="mermaid">` a **fixed height** matching screenshot images: `height: 450px; overflow: auto; display: block`. Use `display: block` (NOT `display: flex`) — flex centering clips the top of tall diagrams even with scrollbars
- Add a subtle border and background to the `<pre class="mermaid">` container so it's visually distinct
- Mermaid text color defaults should remain readable against the page background

### Example

| Markdown Input | HTML Output |
|---------------|-------------|
| ```` ```mermaid\ngraph LR\n  A --> B\n```` ``` | `<pre class="mermaid">graph LR\n  A --> B\n</pre>` |

## Media Handling

**Image references:**
1. Find all `![alt](path)` in the markdown
2. Resolve relative paths from the markdown file's location
3. Copy each image to `{output}/media/{filename}`
4. Rewrite the `<img src>` in HTML:
   - **Single-language:** `media/{filename}`
   - **Multi-language:** `media/{DEFAULT_LANG}/{filename}` where `{DEFAULT_LANG}` is the first language in `LANGS`

**Non-image files (PDFs, docs):**

1. Same copy-to-media process
2. Rewrite link `href`:
   - **Single-language:** `media/{filename}`
   - **Multi-language:** `media/{DEFAULT_LANG}/{filename}`

**Company logos:**
- Always copy all files from `company_style/` to the `media/` directory:
  - **Single-language:** `{output}/media/`
  - **Multi-language:** ALL language subfolders (`{output}/media/zh/`, `{output}/media/en/`, etc.)
- In HTML, reference logos from `media/` (single-lang) or `media/{DEFAULT_LANG}/` (multi-lang)
- Header uses `wisquest_horizontal_logo.png`
- Footer uses `wisquest_horizontal_logo_widemargin.png`
- Circle logo available for favicon if desired
- **Horizontal logos have black text — NEVER place them on dark backgrounds.** The header and footer bars must use a light/white background (`#ffffff` or `var(--neutral-100)`) to keep logo text legible

## Multi-Language Support

When `LANGS` contains more than one language (comma-separated), the generated HTML must include a full i18n (internationalization) system. When `LANGS` has only one language, skip all i18n — generate a single-language page directly in that language.

### LANGS Variable

`LANGS` is a comma-separated string of language codes. The **first language** in the list is the **default language** shown on first page load.

| `LANGS` | Languages | i18n Required? |
|---------|-----------|----------------|
| `zh` | Simplified Chinese only | ❌ No — single-language page |
| `zh,en,ru` | Chinese (default), English, Russian | ✅ Yes — full i18n |
| `en,zh,ja,ko` | English (default), Chinese, Japanese, Korean | ✅ Yes — full i18n |

### Language Switching Widget

When multi-language is active, add a language switcher in the **top-right corner of the header**:

- **Position:** Inside the header bar, right-aligned. Displayed to the right of the version string, before any other controls.
- **Appearance:** A row of compact language-code buttons (e.g., `ZH` `EN` `RU`). Each button has the `.lang-switch-btn` class. The active language button uses the accent color (`var(--accent-700)`) as background or border to distinguish it.
- **Interaction:** Clicking a button switches the page language immediately. Persist the choice in `localStorage` (key: `manual-lang`).
- **On page load:** Read `localStorage` for saved preference; fall back to the first language in `LANGS` (the default).
- **Style notes:** Buttons should be compact (`font-size: 0.75rem; padding: 2px 6px; border-radius: 3px`), with a subtle border (`1px solid var(--neutral-200)`). Active button gets `background: var(--accent-700); color: #fff; border-color: var(--accent-700)`.

### data-i18n Attributes

All localizable UI chrome text must use `data-i18n` attributes. Each translatable element gets a unique key:

```html
<span data-i18n="toc-title">目录</span>
<button data-i18n="back-to-top">返回顶部</button>
<p data-i18n="footer-copyright">© 2026 研知教育科技 版权所有</p>
```

The initial text content (before any JS runs) must be in the **default language** (first in `LANGS`). The i18n system replaces it on language switch.

### Translation Map (JavaScript)

Include a JavaScript `I18N` object in the page, mapping every `data-i18n` key to translations for each language in `LANGS`:

```javascript
const I18N = {
  zh: {
    'toc-title': '目录',
    'back-to-top': '返回顶部',
    'footer-copyright': '© 2026 研知教育科技 版权所有',
    'fold-sidebar': '折叠侧边栏',
    'expand-sidebar': '展开侧边栏',
    'lang-switcher-label': '语言'
  },
  en: {
    'toc-title': 'Contents',
    'back-to-top': 'Back to Top',
    'footer-copyright': '© 2026 WisQuest EdTech. All rights reserved.',
    'fold-sidebar': 'Collapse Sidebar',
    'expand-sidebar': 'Expand Sidebar',
    'lang-switcher-label': 'Language'
  },
  ru: {
    'toc-title': 'Содержание',
    'back-to-top': 'Наверх',
    'footer-copyright': '© 2026 WisQuest EdTech. Все права защищены.',
    'fold-sidebar': 'Свернуть боковую панель',
    'expand-sidebar': 'Развернуть боковую панель',
    'lang-switcher-label': 'Язык'
  }
};
```

### Language Switching Implementation

```javascript
function switchLanguage(lang) {
  if (!I18N[lang]) return;
  document.documentElement.lang = lang === 'zh' ? 'zh-CN' : lang;
  document.querySelectorAll('[data-i18n]').forEach(el => {
    const key = el.getAttribute('data-i18n');
    if (I18N[lang] && I18N[lang][key]) {
      el.textContent = I18N[lang][key];
    }
  });
  localStorage.setItem('manual-lang', lang);
  // Update switcher button active states
  document.querySelectorAll('.lang-switch-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.lang === lang);
  });
}

// On page load: restore preference
document.addEventListener('DOMContentLoaded', () => {
  const saved = localStorage.getItem('manual-lang');
  const defaultLang = '{{DEFAULT_LANG}}'; // first language in LANGS
  switchLanguage(saved && I18N[saved] ? saved : defaultLang);
});
```

Replace `{{DEFAULT_LANG}}` with the first language code from `LANGS` during HTML generation.

### Content Language (Markdown Body)

The markdown content itself (headings, paragraphs, etc.) is in the source language and is **NOT translated automatically** by the i18n system. The i18n system handles only the **UI chrome**:

| Element | i18n? | Notes |
|---------|-------|-------|
| Header title | ✅ Yes | `data-i18n` |
| TOC heading ("目录") | ✅ Yes | `data-i18n` |
| Back-to-top button | ✅ Yes | `data-i18n` |
| Footer copyright | ✅ Yes | `data-i18n` |
| Language switcher labels | ✅ Yes | `data-i18n` |
| Sidebar toggle aria-label | ✅ Yes | `data-i18n` |
| Body content (headings, paragraphs) | ❌ No | Original markdown language |
| Screenshot captions | ❌ No | Original markdown language |
| Table headers | ❌ No | Original markdown language |

### Company Name Rules

**CRITICAL — Company name must match the active language:**

| Language | Company Name | Footer Copyright |
|----------|-------------|------------------|
| `zh` (Chinese) | 研知教育科技 | © 2026 研知教育科技 版权所有 |
| `en` (English) | WisQuest EdTech | © 2026 WisQuest EdTech. All rights reserved. |
| `ru` (Russian) | WisQuest EdTech | © 2026 WisQuest EdTech. Все права защищены. |
| Any other non-Chinese | WisQuest EdTech | © 2026 WisQuest EdTech. All rights reserved. |

**Key rule: 研知教育科技 is ONLY used in the Chinese (`zh`) version. All non-Chinese versions MUST use `WisQuest EdTech`.** This applies to footer copyright, page title, and any company name reference in the UI chrome.

### No Chinese in Non-Chinese Content

**CRITICAL:** When generating the i18n translation map for non-Chinese languages:
- NEVER include Chinese characters (汉字) in English, Russian, or any other non-Chinese translation strings
- The company name in all non-Chinese versions is `WisQuest EdTech` (English), NOT `研知教育科技`
- Verify every translation value is in the correct script for that language
- The `<html lang>` attribute must reflect the current language: `zh-CN` for Chinese, the bare language code otherwise

## Output Structure

**Single-language** (`LANGS=zh`):

```
{manual-name}-html/
├── index.html          # Complete standalone HTML
└── media/              # Flat media directory
    ├── 1-login.png
    ├── 2-settings.png
    ├── wisquest_horizontal_logo.png
    ├── wisquest_horizontal_logo_widemargin.png
    └── wisquest_white_circle_background.png
```

**Multi-language** (`LANGS=zh,en,ru` — media isolated per language):

```
{manual-name}-html/
├── index.html          # Complete standalone HTML with i18n
└── media/
    ├── zh/             # Chinese media (default language)
    │   ├── 1-login.png
    │   ├── 2-settings.png
    │   ├── wisquest_horizontal_logo.png
    │   ├── wisquest_horizontal_logo_widemargin.png
    │   └── wisquest_white_circle_background.png
    ├── en/             # English media (same screenshots + logos)
    │   ├── 1-login.png
    │   ├── 2-settings.png
    │   └── ... (logos)
    └── ru/             # Russian media (same screenshots + logos)
        ├── 1-login.png
        ├── 2-settings.png
        └── ... (logos)
```

**Key rule:** Screenshots are identical across all language subfolders — the same set of media files is copied to every `media/{lang}/` directory. Logos follow the same pattern.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using external CSS/JS files | All styles and scripts must be inline — single `index.html` |
| Absolute paths for media | Always use relative `media/` paths |
| Missing heading IDs | Every heading needs an `id` for TOC linking |
| Forgetting to copy logos | Always copy all `company_style/` files |
| Not handling duplicate heading text | Append `-2`, `-3` etc. to duplicate slugs |
| Using non-Chinese UI text | Default to Simplified Chinese for all chrome text |
| Overwriting original markdown | Output to a new folder, never modify the source |
| Large images not optimized | Consider warning user if images exceed 2MB |
| Missing print styles | Include `@media print` to hide navigation elements |
| Horizontal logo on dark background | Logo text is black — header/footer must be white/light |
| Screenshot images too large | Set `max-width: 720px; height: 450px` on all `<figure>` images (5:8 ratio, matching 1200×1920 source) |
| Screenshot caption too long (detailed description) | Shorten to ≤10 characters after `图X：` — captions are labels, not image descriptions |
| Placeholder text above image | Place `【图X：...】` caption below image as `<figcaption>` |
| Dark text on dark background | Use white/light text on any dark-colored element |
| TOC section duplicated in body | Sidebar already shows TOC — omit "目录" sections from content |
| Screenshot index table included in HTML | Omit "截图索引" section entirely — it's a build-time reference, not end-user content |
| Outputting raw Mermaid code in HTML | Convert ```` ```mermaid ```` blocks to `<pre class="mermaid">` with CDN + initialization |
| Button icon descriptions not converted to tables | Detect "图标说明" sections by content (icon-description text + image links), not by markdown format. Convert to clean 2-column tables (图标 \| 名称) with `.icon-inline` class |
| Anchor scroll hidden behind fixed header | Add `scroll-margin-top: 80px` to all `h2` and `h3` headings |
| Lazy images shift anchor scroll position | Give screenshot `<img>` explicit `width`/`height` attrs + CSS `height: 450px; max-width: 720px; object-fit: contain` |
| Images have opacity applied (semi-transparent) | All images must be fully opaque — NEVER apply `opacity`, `transition: opacity`, or `:hover { opacity }` to `<img>`, `<figure>`, or image containers |
| Mermaid diagram different height than screenshots | Give `<pre class="mermaid">` `height: 450px` to match screenshot height |
| Mermaid diagram top clipped with flex | Use `display: block` (NOT `display: flex`) on `<pre class="mermaid">` to avoid overflow clipping |
| Menu icon missing or in wrong position | Place ☰ menu icon to the **left of the logo** in the header, not to the right or standalone |
| Sidebar toggle folds sidebar on TOC link click | Do NOT fold sidebar when a TOC link is clicked — sidebar state is controlled only by the menu icon |
| Sidebar fold state not persisted | Persist sidebar fold state in `localStorage` so preference survives page reloads |
| Sidebar toggle has no animation | Use CSS `transition` on `transform` or `margin-left` for smooth fold/unfold animation |
| Fixed header covers anchor target on TOC click | Intercept TOC link clicks with JS, use `window.scrollTo()` with manual offset (header 64px + 16px padding) |
| Anchor offset only uses CSS scroll-margin-top | CSS-only approach doesn't handle dynamic header height changes — add JS interception as primary method, CSS as fallback |
| Anchor code block rendered in HTML output | The `anchor` fenced code block is git metadata — strip it from HTML output entirely, just like TOC and screenshot index sections |
| Anchor IDs contain heading numbers (e.g., `1.1-login`, `01_faq`) | Remove all heading numbers — anchors use clean lowercase English text only, e.g., `login`, `faq` |
| Anchor IDs use Chinese characters (e.g., `系统登录-登录系统`) | Translate Chinese headings to English for anchor IDs: use lowercase English words with hyphen separators, e.g., `system-login-login-system` |
| Anchor IDs use pinyin (e.g., `xi-tong-deng-lu`) | Translate Chinese headings to actual English words, not pinyin — e.g., `system-login` not `xi-tong-deng-lu` |
| Language selection not asked before HTML generation | Step 0 is MANDATORY — always ask the user for language preferences before any other work |
| i18n not implemented when `LANGS` has multiple languages | When `LANGS` contains a comma, implement full `data-i18n` + language switcher + translation map |
| Language switcher missing or in wrong position | Language switcher must be in the **top-right corner of the header**, to the right of the version string |
| Language switcher uses dropdown instead of button row | Use a row of compact language-code buttons (`ZH` `EN` `RU`), not a `<select>` dropdown |
| Language preference not persisted across page loads | Save to `localStorage` (key: `manual-lang`) and restore on page load |
| Chinese company name in non-Chinese translations | `研知教育科技` ONLY in `zh`. All non-Chinese versions use `WisQuest EdTech` |
| Chinese characters appear in non-Chinese i18n strings | Verify every non-Chinese translation value contains no 汉字 — use only the target language's script |
| `<html lang>` not updated on language switch | Update `document.documentElement.lang` when switching languages |
| `data-i18n` initial text not in default language | The HTML body's initial text content must match the default language (first in `LANGS`) |
| Multi-language HTML but media not isolated per language | When `LANGS` has >1 language, create `media/{lang}/` subfolders for each language and copy screenshots + logos to ALL subfolders |
| Multi-language HTML uses flat `media/` structure | Multi-language mode requires `media/{lang}/` structure; flat `media/` without language subfolders is only for single-language mode |
| Screenshots not copied to all language subfolders in multi-lang mode | Every screenshot must be copied to ALL language media subfolders (e.g., `media/zh/`, `media/en/`, `media/ru/`) |
| Media `src` paths don't include language prefix in multi-lang mode | Use `media/{DEFAULT_LANG}/{filename}` for all image/link paths when multi-language |
| Logos only copied to default language folder in multi-lang mode | Copy logos to ALL language subfolders — each language needs its own complete media set |
