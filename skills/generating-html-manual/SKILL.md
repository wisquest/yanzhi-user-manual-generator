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
- Default language: **Simplified Chinese** unless user specifies otherwise

## Prerequisites

1. **Markdown user manual** — the `.md` file to convert
2. **Media files** — any images or assets referenced in the markdown (relative paths)

**REQUIRED REFERENCES:**
- Read `color-spec.md` in this skill's directory for the complete color system and design tokens.
- Read `company_style/` in this skill's directory for logo assets.

**DEVELOPMENT METHODOLOGY — TDD Required (RED-GREEN-REFACTOR):**

This skill generates a standalone single HTML webpage. **HTML code generation (Steps 3-4) MUST follow the superpowers TDD cycle:**

1. **RED — Write failing tests first:** Before writing ANY HTML/CSS/JS code, write test assertions for each structure, style, and behavior. Tests must verify:
   - Correct heading hierarchy and ID slugs
   - TOC link targets resolve to valid heading IDs
   - Sidebar is always visible (no collapse/hide behavior) — sidebar width remains 280px at all times on desktop, no toggle button exists
   - Anchor scroll offset prevents fixed header from hiding targets
   - Back-to-top button visibility (hidden at top, visible after scroll > 400px)
   - Responsive breakpoints (≤1024px: sidebar and content stack vertically or sidebar becomes a fixed overlay accessible via a toggle; ≤640px reduced fonts)
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
  "Read markdown input" [shape=box];
  "Parse headings for TOC" [shape=box];
  "Identify media references" [shape=box];
  "Generate output folder" [shape=box];
  "Copy media files to output folder" [shape=box];
  "Convert markdown to HTML body" [shape=box];
  "Apply HTML template with sidebar + branding" [shape=box];
  "Write index.html to output folder" [shape=box];
  "Verify all media paths are valid" [shape=box];
  "Done" [shape=doublecircle];

  "Read markdown input" -> "Parse headings for TOC";
  "Parse headings for TOC" -> "Identify media references";
  "Identify media references" -> "Generate output folder";
  "Generate output folder" -> "Copy media files to output folder";
  "Copy media files to output folder" -> "Convert markdown to HTML body";
  "Convert markdown to HTML body" -> "Apply HTML template with sidebar + branding";
  "Apply HTML template with sidebar + branding" -> "Write index.html to output folder";
  "Write index.html to output folder" -> "Verify all media paths are valid";
  "Verify all media paths are valid" -> "Done";
}
```

### Step 1: Read and Parse

1. Read the input markdown file
2. Parse all headings (`##` and `###`) to build the sidebar TOC
3. Scan for media references: `![alt](path)`, `<img src="path">`, `[text](path.pdf)` etc.
4. Scan for mermaid/fenced diagram blocks: ```` ```mermaid ```` code blocks
5. Record the relative paths of all referenced media files

### Step 2: Create Output Folder

1. Create a new folder next to the input markdown file, named `{manual-name}-html/`
2. Create `media/` subfolder inside it for copied assets
3. Copy all referenced media files from their original locations to `media/`
4. Copy the company logo files from this skill's `company_style/` directory to `media/`

### Step 3: Convert Markdown to HTML

Convert the markdown content to well-structured HTML following these rules:

| Markdown | HTML Output |
|----------|------------|
| `## Heading` | `<h2>` with `id` attribute (slug) |
| `### Heading` | `<h3>` with `id` attribute (slug) |
| `**bold**` | `<strong>bold</strong>` |
| `> **note**: text` | `<blockquote>` with `.callout-tip` class |
| `> **warning**: text` | `<blockquote>` with `.callout-warning` class |
| `![alt](path)` | `<figure><img><figcaption>` structure, `max-width` capped at fixed value |
| `【图X：...】` | `<figure>` with `.screenshot-placeholder` class; placeholder text rendered as small caption **below** the image |
| Tables | Standard `<table>` with `<thead>` and `<tbody>` |
| `1. item` | `<ol><li>item</li></ol>` |
| `- item` | `<ul><li>item</li></ul>` |
| `` `code` `` | `<code>code</code>` |
| Code blocks | `<pre><code>...</code></pre>` |
| ````mermaid` blocks | `<pre class="mermaid">` — render as live diagrams, NOT raw code |

**Skip inline TOC sections:** If the markdown contains a "目录"、"Table of Contents"、"内容提要" or similar TOC section (a heading followed by a list of internal links), **omit it from the HTML body**. The sidebar already provides navigation — duplicating the TOC in the content area wastes space and confuses readers.

**Skip screenshot index table:** If the markdown ends with a screenshot index table (a section titled "截图索引"、"截图索引表"、"Screenshot Index" or similar, containing a table that maps screenshot placeholders to file paths), **omit this entire section from the HTML output**. This table is a build-time reference for tracking which screenshots exist — it is not end-user content and does not belong in the published manual.

**Heading ID slugs:** Generate from heading text — lowercase, replace spaces/special chars with hyphens, ensure uniqueness by appending `-2`, `-3` etc. for duplicates.

**Media path rewrite:** All media references in the HTML must point to `media/` relative path.

**Screenshot image sizing:** All screenshot images (`<img>` inside `<figure>`) must have a fixed height: `height: 450px; object-fit: contain; object-position: center; max-width: 600px; width: 100%`. This ensures the browser reserves exactly 450px of vertical space **before** images load, preventing anchor scroll positions from drifting when lazy-loaded images arrive. The `object-fit: contain` preserves aspect ratio without distortion. **Also add explicit `width` and `height` HTML attributes** to each `<img>` tag (obtain actual pixel dimensions via `sips` or similar tool) so the browser can compute the intrinsic aspect ratio even before CSS is applied.

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

**Button icon descriptions:** If the markdown contains "按钮图标说明" sections (blockquotes with a list of icon names and corresponding `<figure><img>` elements), convert them to clean **tables** with two columns: 图标 (icon image) and 名称 (name). Use `<img class="icon-inline">` for the icon column, with CSS `height: 1.3em; width: auto` so icons render at text-line height. This is far more readable than the original blockquote+list+figure format. Example table structure:
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

- **Header** — fixed top bar with company logo, page title, and version string. On mobile (≤1024px), contains a hamburger toggle for the sidebar overlay.
- **Sidebar** — fixed left navigation with TOC, always visible on desktop (>1024px). No collapse/hide functionality on desktop — the sidebar is a permanent fixture. On mobile (≤1024px), sidebar becomes an overlay that can be toggled via the hamburger button in the header.
- **Content** — main body area with max-width 900px, centered
- **Footer** — company logo and copyright
- **Back-to-top button** — fixed bottom-right, appears on scroll

#### Template Placeholders

| Placeholder | Source |
|-------------|--------|
| `{{TITLE}}` | First `#` heading or filename |
| `{{VERSION}}` | Version string if found (e.g., "V2.3.0"), otherwise empty |
| `{{CONTENT}}` | Converted HTML body from Step 3 |
| `{{TOC}}` | Generated sidebar TOC from headings |

#### Layout Specs

| Element | Spec |
|---------|------|
| Header height | 64px, fixed top, `z-index: 1000` |
| Sidebar width | 280px, fixed left, `z-index: 900`. Always visible on desktop (>1024px) — no collapse/hide. |
| Content max-width | 900px, centered. Content area has `margin-left: 280px` on desktop to make room for the permanent sidebar. |
| Anchor scroll offset | CSS: `scroll-margin-top: 80px` on all `h2`/`h3`. JS: intercept TOC link clicks, call `scrollIntoView()` with manual offset for the 64px header + 16px breathing room |
| Back-to-top trigger | Scroll > 400px |
| Mobile breakpoint | ≤1024px: sidebar overlay mode (fixed position, full-height, shadow backdrop), toggle in header |
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

#### Responsive Behavior

- **Desktop (>1024px):** Sidebar always visible (280px), content offset with `margin-left: 280px`. No collapse/hide toggle exists — the sidebar is a permanent part of the layout.
- **Tablet/Mobile (≤1024px):** Sidebar hidden by default, hamburger toggle in header. When toggled open, sidebar displays as an overlay (full-height, shadow backdrop, closes on backdrop click or nav link click). Header has the hamburger button only on mobile.
- **Small mobile (≤640px):** Reduced heading font sizes. Overlay sidebar takes full width.

#### Sidebar Always Visible

The sidebar TOC navigation is a permanent part of the page layout:

1. **Desktop (>1024px):** Sidebar is always visible at 280px width. No collapse button, no hide functionality, no toggle in header. The sidebar cannot be hidden by the user — it is a fixed part of the layout at all times.
2. **Content offset:** The main content area always has `margin-left: 280px` on desktop to accommodate the permanent sidebar.
3. **No localStorage persistence:** Since the sidebar never collapses, there is no need for `localStorage` sidebar state tracking.
4. **No CSS transitions for sidebar width:** The sidebar width is static — no `transition` on `width` or `margin-left` needed.
5. **Mobile (≤1024px):** Sidebar defaults to hidden (overlay mode). The header displays a hamburger button (☰) that toggles the sidebar as a full-height overlay with dark backdrop. Clicking the backdrop or a TOC link closes the overlay.
6. **Header toggle button:** The hamburger toggle button only appears on mobile (≤1024px). On desktop, there is no toggle button — the header contains only the logo, title, and version.

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
   - Mobile overlay mode (≤1024px) → close sidebar overlay after TOC link click
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
- Generate URL-friendly slugs: lowercase, Chinese characters kept as-is, spaces to `-`, remove punctuation
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
4. Rewrite the `<img src>` to `media/{filename}` in HTML

**Non-image files (PDFs, docs):**

1. Same copy-to-media process
2. Rewrite link `href` to `media/{filename}`

**Company logos:**
- Always copy all files from `company_style/` to `{output}/media/`
- Header uses `研知教育科技_horizontal_logo.png`
- Footer uses `研知教育科技_horizontal_logo_widemargin.png`
- Circle logo available for favicon if desired
- **Horizontal logos have black text — NEVER place them on dark backgrounds.** The header and footer bars must use a light/white background (`#ffffff` or `var(--neutral-100)`) to keep logo text legible

## Language Default

- Always use `lang="zh-CN"` on `<html>` tag
- UI text in Chinese: "目录" (TOC), "返回顶部" (Back to top)
- Footer copyright in Chinese
- If user specifies a different language, adapt UI text accordingly

## Output Structure

```
{manual-name}-html/
├── index.html          # Complete standalone HTML
└── media/
    ├── 图1-登录页面.png
    ├── 图2-首页概览.png
    ├── 研知教育科技_horizontal_logo.png
    ├── 研知教育科技_horizontal_logo_widemargin.png
    └── 研知教育科技_white_circle_background.png
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using external CSS/JS files | All styles and scripts must be inline — single `index.html` |
| Absolute paths for media | Always use relative `media/` paths |
| Missing heading IDs | Every heading needs an `id` for TOC linking |
| Forgetting to copy logos | Always copy all `company_style/` files |
| Hardcoded sidebar visible on mobile | Use responsive CSS + JS toggle |
| Not handling duplicate heading text | Append `-2`, `-3` etc. to duplicate slugs |
| Using non-Chinese UI text | Default to Simplified Chinese for all chrome text |
| Overwriting original markdown | Output to a new folder, never modify the source |
| Large images not optimized | Consider warning user if images exceed 2MB |
| Missing print styles | Include `@media print` to hide navigation elements |
| Horizontal logo on dark background | Logo text is black — header/footer must be white/light |
| Screenshot images too large | Set `max-width: 600px` on all `<figure>` images |
| Screenshot caption too long (detailed description) | Shorten to ≤10 characters after `图X：` — captions are labels, not image descriptions |
| Placeholder text above image | Place `【图X：...】` caption below image as `<figcaption>` |
| Dark text on dark background | Use white/light text on any dark-colored element |
| TOC section duplicated in body | Sidebar already shows TOC — omit "目录" sections from content |
| Screenshot index table included in HTML | Omit "截图索引" section entirely — it's a build-time reference, not end-user content |
| Outputting raw Mermaid code in HTML | Convert ```` ```mermaid ```` blocks to `<pre class="mermaid">` with CDN + initialization |
| Button icon descriptions as messy blockquotes | Convert "按钮图标说明" to clean 2-column tables (图标 \| 名称) with `.icon-inline` class |
| Anchor scroll hidden behind fixed header | Add `scroll-margin-top: 80px` to all `h2` and `h3` headings |
| Lazy images shift anchor scroll position | Give screenshot `<img>` explicit `width`/`height` attrs + CSS `height: 450px; object-fit: contain` |
| Images have opacity applied (semi-transparent) | All images must be fully opaque — NEVER apply `opacity`, `transition: opacity`, or `:hover { opacity }` to `<img>`, `<figure>`, or image containers |
| Mermaid diagram different height than screenshots | Give `<pre class="mermaid">` `height: 450px` to match screenshot height |
| Mermaid diagram top clipped with flex | Use `display: block` (NOT `display: flex`) on `<pre class="mermaid">` to avoid overflow clipping |
| Sidebar has collapse/hide button on desktop | Remove any collapse toggle from desktop view — sidebar is always visible on desktop (>1024px), no toggle button in header |
| Toggle button in header on desktop | Header toggle button (☰) should only appear on mobile (≤1024px). Desktop header has no toggle — just logo, title, version |
| Sidebar uses localStorage for state | No localStorage needed for sidebar state — sidebar is always visible on desktop, no collapse state to persist |
| Sidebar has CSS width transitions | Remove `transition: width` or `transition: margin-left` — sidebar width is static on desktop, no animation needed |
| Fixed header covers anchor target on TOC click | Intercept TOC link clicks with JS, use `window.scrollTo()` with manual offset (header 64px + 16px padding) |
| Anchor offset only uses CSS scroll-margin-top | CSS-only approach doesn't handle dynamic header height changes — add JS interception as primary method, CSS as fallback |
