# PPTX HTML Constraints

Rules for writing HTML slides that can be converted to editable PPTX. These are physical limitations of the PowerPoint file format (OOXML), not stylistic preferences. Violating them causes export failure or corrupted output.

---

## The 4 Hard Rules

### Rule 1: All text must be in `<p>` or `<h1>`-`<h6>` tags

```html
<!-- WRONG: bare text in a div -->
<div class="title">Q3 revenue grew 23%</div>

<!-- CORRECT: text wrapped in a paragraph tag -->
<div class="title"><h1>Q3 revenue grew 23%</h1></div>
<div class="body"><p>New users drove growth</p></div>
```

PowerPoint requires text to be inside a "text frame" (`<a:txBody>` in OOXML). A bare `<div>` has no text frame equivalent. Also, `<span>` cannot carry main text — it's inline-only and can't be positioned independently. Use `<span>` only for inline styling within `<p>` or `<h*>`.

### Rule 2: No CSS gradients — solid colors only

```css
/* WRONG */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* CORRECT: solid color */
background: #FF6B6B;

/* CORRECT: multi-color stripe via flex children */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red  { background: #FF6B6B; }
.teal { background: #4ECDC4; }
```

PPTX shape fill only supports solid fill via the current export pipeline. If you need a multi-color effect, use adjacent `<div>` elements with different solid colors.

### Rule 3: Backgrounds, borders, and shadows go on `<div>`, never on text tags

```html
<!-- WRONG: <p> has background -->
<p style="background: #FFD700; border-radius: 4px;">Highlighted content</p>

<!-- CORRECT: outer div carries visual properties, inner text tag carries text -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Highlighted content</p>
</div>
```

In PowerPoint, a "shape" (rectangle, rounded rect) and a "text frame" are separate objects. `<p>` becomes a text frame only — visual properties belong on the wrapping shape (`<div>`).

### Rule 4: Use `<img>` tags for images, never `background-image`

```html
<!-- WRONG -->
<div style="background-image: url('chart.png')"></div>

<!-- CORRECT -->
<img src="chart.png" style="position: absolute; left: 50pt; top: 20pt; width: 300pt; height: 200pt;" />
```

The export script only extracts images from `<img>` elements. CSS `background-image` is invisible to the converter.

### Rule 5: No manual bullet symbols — use `<ul>/<li>` or `<ol>/<li>`

```html
<!-- WRONG: manual bullet symbols in <p> tags -->
<p>- First point</p>
<p>- Second point</p>
<p>1. Step one</p>
<p>2. Step two</p>

<!-- CORRECT: use proper list elements -->
<ul style="font-size: 12pt; color: #A6ADC8; padding-left: 20pt;">
  <li>First point</li>
  <li>Second point</li>
</ul>
<ol style="font-size: 12pt; color: #A6ADC8; padding-left: 20pt;">
  <li>Step one</li>
  <li>Step two</li>
</ol>
```

The export script rejects `<p>` tags that start with bullet symbols (`-`, `*`, `•`) or numbered patterns (`1.`, `2.`). Lists must use semantic HTML list elements. This is because PowerPoint has native list objects that correspond to `<ul>/<ol>`, and manual symbols would render as plain text with no list formatting.

### Rule 6: No inline SVG elements — use `<img>` instead

```html
<!-- WRONG: inline SVG causes className.includes is not a function error -->
<div style="position: absolute; top: 60pt; right: 80pt;">
  <svg xmlns="http://www.w3.org/2000/svg" width="48" height="46" viewBox="0 0 48 46">
    <path fill="#863bff" d="M25.946..."/>
  </svg>
</div>

<!-- CORRECT: reference SVG as an image file -->
<div style="position: absolute; top: 60pt; right: 80pt;">
  <img src="../images/favicon.svg" style="width: 48pt; height: 46pt;" />
</div>
```

The export script iterates DOM elements and calls `el.className.includes()`. SVG elements have `SVGAnimatedString` for `className` (not a regular string), causing a `TypeError`. To use logos or icons, save the SVG file to the `images/` directory and reference it via `<img>` tags.

---

## Font Family Requirement

### Rule 14: Use explicit font names — never generic families (`system-ui`, `-apple-system`, `sans-serif`)

The export script (`html2pptx.js`) extracts the font name via `computed.fontFamily.split(',')[0]`. Generic family names are **not valid PowerPoint font names** — PowerPoint falls back to Calibri/SimSun, producing completely different character widths than the HTML preview.

```css
/* WRONG: system-ui → PowerPoint uses Calibri → width mismatch */
body { font-family: system-ui, -apple-system, "PingFang SC", sans-serif; }

/* CORRECT: explicit font names that PowerPoint recognizes */
body { font-family: "PingFang SC", "Microsoft YaHei", Arial, sans-serif; }
```

**Why this matters more than Rule 10**: Rule 10 assumes the HTML and PPTX use the *same font* with 5-10% width expansion. If the fonts are completely different (PingFang SC vs Calibri), Rule 10's percentages are meaningless — CJK characters in Calibri can be 30-50% wider than in PingFang SC.

**Font stack rationale**:
- `"PingFang SC"` — first choice, best CJK rendering on macOS, recognized by PowerPoint for Mac
- `"Microsoft YaHei"` — fallback for Windows, metrically similar to PingFang SC
- `Arial` — Latin fallback, universally available
- `sans-serif` — last resort (only used if nothing above is available)

---

## Canvas Size

Use `960pt × 540pt` for the body. This maps to PowerPoint's `LAYOUT_WIDE` (13.333″ × 7.5″), the modern 16:9 standard.

```css
body { width: 960pt; height: 540pt; }     /* recommended */
body { width: 1280px; height: 720px; }     /* equivalent */
body { width: 13.333in; height: 7.5in; }   /* equivalent */
```

---

## HTML Slide Template

Each slide is a standalone HTML file:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;
    font-family: "PingFang SC", "Microsoft YaHei", Arial, sans-serif;
    background: #FEFEF9;
    overflow: hidden;
  }
</style>
</head>
<body>

<!-- Slide canvas — only this uses absolute positioning -->
<div style="position: absolute; top: 0; left: 0; width: 960pt; height: 540pt; background: #FEFEF9;">

  <!-- Accent bar (decorative — absolute OK) -->
  <div style="position: absolute; top: 0; left: 0; width: 960pt; height: 6pt; background: #1A4A8A;"></div>

  <!-- Content area — ONE absolute wrapper, flow layout inside -->
  <div style="position: absolute; top: 30pt; left: 60pt; right: 60pt; bottom: 30pt;">

    <!-- Title (flow) -->
    <div style="margin-bottom: 20pt;">
      <h1 style="font-size: 28pt; color: #1A1A1A; font-weight: 700;">Slide Title</h1>
      <p style="font-size: 12pt; color: #888888; margin-top: 6pt;">Subtitle description</p>
    </div>

    <!-- Two-column layout using flex (flow) -->
    <div style="display: flex; gap: 16pt;">
      <div style="flex: 1; background: #1A4A8A; border-radius: 4pt; padding: 12pt 16pt;">
        <h2 style="font-size: 14pt; color: #FFFFFF; font-weight: 700;">Point One</h2>
        <p style="font-size: 11pt; color: rgba(255,255,255,0.85); margin-top: 6pt;">Description</p>
      </div>
      <div style="flex: 1; background: #1A4A8A; border-radius: 4pt; padding: 12pt 16pt;">
        <h2 style="font-size: 14pt; color: #FFFFFF; font-weight: 700;">Point Two</h2>
        <p style="font-size: 11pt; color: rgba(255,255,255,0.85); margin-top: 6pt;">Description</p>
      </div>
    </div>

    <!-- List (flows below columns) -->
    <div style="margin-top: 20pt;">
      <ul style="font-size: 13pt; color: #1A1A1A; padding-left: 24pt; list-style: disc; line-height: 1.6;">
        <li>First point</li>
        <li>Second point</li>
        <li>Third point</li>
      </ul>
    </div>

  </div>
</div>

</body>
</html>
```

---

## Anti-Overflow Layout Rules

HTML `overflow: hidden` clips content visually, but PPTX does not clip — content extends beyond the slide edge. Combined with font rendering differences (PPT text is typically slightly wider/taller than browser text), dense layouts that look fine in HTML will overflow in PPTX.

**Rule 7: Leave buffer space**

- **Bottom margin**: Minimum 30pt from the last content element to the body bottom edge. PowerPoint's presentation UI partially covers the bottom of slides.
- **Top margin**: Minimum 30pt from the first content element to the body top edge.
- **Side margins**: Minimum 50pt left and right.
- **Content density**: No more than 4-5 content blocks (cards, lists, tables) per slide. If content is tight, split into two slides.

**Rule 8: Conservative font sizing for dense content**

When a slide has many elements, use smaller fonts than the template defaults:
- Body text: 10-11pt instead of 12-13pt (for slides with 4+ content blocks)
- List items: 10-11pt instead of 12pt
- Card descriptions: 10pt instead of 11-12pt
- Headlines stay at 28-36pt (these rarely overflow)

**Rule 9: Prefer fewer items over tight packing**

If you find yourself adjusting positioning to fit content, split the slide instead:
- Bad: 8 list items at 10pt crammed into 180pt height
- Good: 4 items on slide A, 4 items on slide B

**Quick overflow check**: After writing HTML, count the total height of all absolutely positioned elements. If the lowest element's bottom edge exceeds 510pt (540pt - 30pt buffer), the slide will likely overflow in PPTX.

### Rule 10: Text container width must account for PPTX font widening

PowerPoint renders text ~5-10% wider than browsers for the same font-size. Text that fits perfectly on one line in HTML will wrap to two lines in PPTX. This is the #1 cause of "HTML looks right but PPTX overflows."

**Countermeasures**:
- **Body text: 85-90% container width**. If a card visually spans 400pt, set the text container width to 340-360pt. The visual gap in HTML is acceptable — PPTX will fill it.
- **Chinese titles/headers: 75-80% container width**. Chinese characters are significantly wider than Latin. A title like "OKR 目标管理与绩效评估 — 战略对齐的核心闭环" MUST use only 75-80% of the visual card width. If in doubt, shorten the title — brevity wins. PPTX wrapping of Chinese titles is the #1 remaining overflow issue.
- **Shorten Chinese text aggressively**: "支持钉钉SSO + 组织架构自动同步" → "钉钉 SSO + 组织同步". "企业协作 SaaS 赛道 — 市场趋势与机会窗口" → "企业协作 SaaS 赛道与机会窗口". Remove em-dashes (—) and split long titles into a shorter title + subtitle.
- **Never let text fill edge-to-edge**: If text is more than 85% of container width (or 80% for Chinese titles), shorten the text, reduce font size, or widen the container.

```html
<!-- WRONG: text container fills entire card width -->
<div style="position: absolute; left: 60pt; top: 100pt; width: 400pt; background: #313244; padding: 14pt;">
  <p style="font-size: 12pt;">这一行中文在HTML中刚好放得下但PPTX中会换行导致溢出</p>
</div>

<!-- CORRECT: text container is 85% of card width -->
<div style="position: absolute; left: 60pt; top: 100pt; width: 340pt; background: #313244; padding: 14pt;">
  <p style="font-size: 12pt;">这一行在HTML和PPTX中都能正确显示</p>
</div>
```

### Rule 11: Use flow layout for content blocks — avoid absolute positioning for content that can wrap

**The problem**: When multiple content blocks use `position: absolute` with hardcoded `top` values, text wrapping (especially Chinese text) causes blocks to overlap. The next block's `top` doesn't adjust to the previous block's actual height.

```html
<!-- WRONG: absolute positioning for content blocks — overlaps when text wraps -->
<div style="position: absolute; top: 100pt; left: 60pt; width: 400pt;">
  <ul><!-- 5 items, each wraps to 2 lines = ~160pt actual height --></ul>
</div>
<div style="position: absolute; top: 240pt; left: 60pt; width: 400pt;">
  <p>This block starts at 240pt but the block above extended to 260pt → OVERLAP</p>
</div>
```

**Correct approach**: Use a **flow container** (`position: absolute` on the outer wrapper only, then flow layout inside):

```html
<!-- CORRECT: one absolute wrapper, flow layout inside -->
<div style="position: absolute; top: 90pt; left: 60pt; right: 60pt;">
  <!-- Title area (flow) -->
  <div style="margin-bottom: 16pt;">
    <h1 style="font-size: 26pt; color: #FFF;">Slide Title</h1>
    <p style="font-size: 12pt; color: #888; margin-top: 4pt;">Subtitle</p>
  </div>

  <!-- Two-column content using flex (flow) -->
  <div style="display: flex; gap: 16pt;">
    <div style="flex: 1; background: #1A1A2E; border-radius: 6pt; padding: 12pt;">
      <h2 style="font-size: 12pt; color: #863bff;">Column A</h2>
      <ul style="font-size: 10pt; padding-left: 16pt; margin-top: 6pt;">
        <li>Item 1 — text wraps naturally, no overlap</li>
        <li>Item 2</li>
      </ul>
    </div>
    <div style="flex: 1; background: #1A1A2E; border-radius: 6pt; padding: 12pt;">
      <h2 style="font-size: 12pt; color: #47BFFF;">Column B</h2>
      <ul style="font-size: 10pt; padding-left: 16pt; margin-top: 6pt;">
        <li>Item 1</li>
        <li>Item 2</li>
      </ul>
    </div>
  </div>

  <!-- Footer flows below columns automatically -->
  <div style="margin-top: 16pt;">
    <p style="font-size: 9pt; color: #666;">Source: file.js:10-20</p>
  </div>
</div>
```

**Layout hierarchy**:
1. **Slide wrapper**: ONE `position: absolute; top: 0; left: 0; width: 960pt; height: 540pt;` — the canvas
2. **Content area**: ONE `position: absolute` container for the main content region (e.g., `top: 80pt; left: 60pt; right: 60pt; bottom: 40pt;`)
3. **Everything inside**: Flow layout — `margin`, `padding`, `display: flex`, `gap`. NO more `position: absolute` for individual content blocks.

**Exceptions where absolute is OK**:
- Decorative elements (accent bars, background shapes) that have fixed dimensions
- Cover slides with simple centered layouts
- Single images with known dimensions

**Why this matters**: Chinese text wraps unpredictably. A list item that's one line in your mental model may be 2-3 lines in practice. Flow layout automatically adjusts; absolute positioning does not.

### Rule 12: Avoid `flex-wrap: wrap` for call chains and flow diagrams — use single text elements with inline formatting

**The problem**: `display: flex; flex-wrap: wrap` creates a flowing layout in HTML where items automatically wrap to new lines. But html2pptx converts each flex child into an **independently positioned** PPTX element. Since PPTX has no flex layout, elements are placed at their browser-extracted positions. PPTX's wider text rendering causes pill-shaped `<div>` elements to be slightly wider, shifting wrap points — arrows (`->`, `→`) and pills end up on unexpected rows.

```html
<!-- WRONG: flex-wrap causes unpredictable wrapping in PPTX -->
<div style="display: flex; align-items: center; gap: 6pt; flex-wrap: wrap;">
  <div style="background: #863bff; border-radius: 4pt; padding: 4pt 10pt;">
    <p style="font-size: 9pt; color: #FFF;">POST /api/tasks</p>
  </div>
  <p style="font-size: 10pt; color: #6C7086;">-></p>
  <div style="background: #1E1E2E; border-radius: 4pt; padding: 4pt 10pt;">
    <p style="font-size: 9pt; color: #7AA2F7;">withAuth() 鉴权</p>
  </div>
  <p style="font-size: 10pt; color: #6C7086;">-></p>
  <div style="background: #1E1E2E; border-radius: 4pt; padding: 4pt 10pt;">
    <p style="font-size: 9pt; color: #9ECE6A;">computeTaskRoi() 评分</p>
  </div>
</div>
```

**Correct approach**: Use a single `<p>` with `<span>` elements for inline color coding. Each chain becomes one text element in PPTX — no flex, no wrapping issues.

```html
<!-- CORRECT: single text element with inline formatting -->
<p style="font-size: 9pt; color: #CDD6F4; line-height: 2.0;">
  <span style="color: #863bff; font-weight: 600;">POST /api/tasks</span>
  <span style="color: #6C7086;"> → </span>
  <span style="color: #7AA2F7;">withAuth() 鉴权</span>
  <span style="color: #6C7086;"> → </span>
  <span style="color: #9ECE6A;">computeTaskRoi() 评分</span>
  <span style="color: #6C7086;"> → </span>
  <span style="color: #F9E2AF;">INSERT ON CONFLICT UPDATE</span>
</p>
```

**Trade-off**: You lose pill-shaped backgrounds around each step. The colored text provides sufficient visual distinction for call chains. If backgrounds are essential, use **explicit rows** (one `<div>` per row, no `flex-wrap`) with items that fit within one line.

### Rule 13: Use `<span>` for inline colored text — never heading tags (`<h1>`-`<h6>`) with `display: inline`

html2pptx.js processes heading tags (`<h1>`-`<h6>`) as standalone text elements AND as part of their parent context (list items, card content). This causes double-processing: the colored text appears both inline and as a separate element. The converter's `parseInlineFormatting` only recognizes `<b>`, `<strong>`, `<i>`, `<em>`, `<u>`, and `<span>` as inline formatting elements.

```html
<!-- WRONG: h3 inside li causes double-processing -->
<li><h3 style="font-size: 10pt; color: #863bff; display: inline;">任务管理</h3> — CRUD + 状态机</li>

<!-- WRONG: h2 inline next to p creates two separate text boxes -->
<div style="background: #313244; padding: 6pt 10pt;">
  <h2 style="font-size: 9pt; color: #863bff; display: inline;">TTRP</h2>
  <p style="font-size: 9pt; color: #CDD6F4; display: inline;">Task / Ticket / Requirement / Project</p>
</div>

<!-- CORRECT: span for inline colored text -->
<li><span style="font-size: 10pt; color: #863bff; font-weight: 700;">任务管理</span> — CRUD + 状态机</li>

<!-- CORRECT: single p with span for inline formatting -->
<div style="background: #313244; padding: 6pt 10pt;">
  <p style="font-size: 9pt; color: #CDD6F4;"><span style="color: #863bff; font-weight: 700;">TTRP</span> — Task / Ticket / Requirement / Project</p>
</div>
```

---

## Common Export Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `DIV element contains unwrapped text "XXX"` | Bare text in `<div>` | Wrap in `<p>` or `<h1>`-`<h6>` |
| `CSS gradients are not supported` | `linear-gradient` or `radial-gradient` | Replace with solid color or flex-based stripes |
| `Text element <p> has background` | `<p>` has `background` CSS | Move background to wrapping `<div>` |
| `Background images on DIV elements are not supported` | `background-image` on `<div>` | Replace with `<img>` tag |
| `HTML content overflows body by Xpt vertically` | Content exceeds 540pt height | Reduce content or shrink font sizes |
| `HTML dimensions don't match presentation layout` | Body size mismatch | Set body to `960pt × 540pt` |
| `flex-wrap` causing arrow/element wrapping in PPTX | PPTX has no flex layout | Use single `<p>` with `<span>` for call chains (Rule 12) |
| Heading tag with `display: inline` double-processed | `<h*>` not recognized as inline by converter | Use `<span>` for inline colored text (Rule 13) |
