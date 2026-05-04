---
name: codebase-to-deck
description: "Analyze any codebase and generate a role-tailored, editable PPTX presentation that introduces the product behind the code. Supports 7 preset personas — Developer (研发), Product Manager (产品经理), Product Operations (产品运营), Sales (销售), End User (终端用户), Partner/Reseller (代理商/合作伙伴), Business Development (商务) — plus custom roles with auto-inferred focus areas. All roles include a universal Content Quality Loop: an independent blind reviewer subagent (no codebase context) scores content on role-specific dimensions — 画面感 (vividness) for business roles, 层次感 (layered depth) for technical roles — and iterates 4-5 rounds to optimize clarity. Extracts architecture, features, user flows, tech stack, and business value from running code as primary truth, with competitive intelligence from multi-round web research. Produces presentation-quality slides via an HTML-to-PPTX pipeline for fully editable output. Trigger this skill whenever the user wants to: create a presentation/PPTX/PPT/deck from a codebase or project, explain a software product to specific audiences, generate product knowledge slides for different roles, turn code into slides, prepare a pitch/intro/onboarding deck based on source code, or any request involving codebase-to-presentation conversion. Also triggers for Chinese phrases like 代码库转PPT, 为XX角色生成演示, 从项目生成PPTX, 代码库生成演示文稿, 帮我准备给XX的演示, 这个项目是做什么的PPT. Do NOT trigger for simple code explanation or documentation generation that doesn't involve slide decks or presentations."
---

# codebase-to-deck

Analyze a codebase → extract product knowledge → generate a role-tailored, editable PPTX presentation.

## Core Principle

**Code is the primary truth.** Running code and configuration files override README, docs, and marketing materials. When they conflict, Developer and PM decks flag the discrepancy; all other roles silently use code truth.

**Language default**: The deck language matches the user's prompt language. If the user writes in Chinese, the deck is in Chinese. If in English, the deck is in English. Only override when the user explicitly specifies a different language.

---

## Workflow (7 Phases, 4 Checkpoints)

### Phase 1: Role Selection & Content Confirmation

1. Ask: "这份演示文稿的目标受众是谁？"
2. Match against 7 preset roles (read `references/role-templates.md` for each role's content template):
   - Developer (研发)
   - Product Manager (产品经理)
   - Product Operations (产品运营)
   - Sales (销售)
   - End User (终端用户)
   - Partner/Reseller (代理商/合作伙伴, includes 渠道商)
   - Business Development (商务)
3. If the user names a **custom role** not in presets:
   - Infer 5-8 focus areas based on that role's relationship to the product
   - Present the inferred list for user confirmation/addition
4. If the user names **multiple roles**, ask whether to produce separate decks or prioritize one.

**Present the selected role's content template** (or custom focus areas) to the user. Ask them to confirm, add, or remove content areas.

> **Checkpoint 1**: Wait for user to confirm role + content areas before touching any code.

### Phase 2: Codebase Analysis

The goal: understand what the product does, how it's built, and what value it provides — from the code. Run these in sequence (each informs the next):

**Step 1 — Project type & tech stack**
Read `package.json` / `requirements.txt` / `go.mod` / `Cargo.toml` / `pom.xml` / `*.csproj` / equivalent. Identify: language, framework, key dependencies, build system.

**Step 2 — Project structure**
Map top-level directories and their purposes. Find entry points (`main.*`, `index.*`, `app.*`, `cmd/`, `src/`, `lib/`).

**Step 3 — Features & routes**
Extract from: route definitions, API endpoints (REST/GraphQL/gRPC), CLI commands, event handlers, UI screens/pages. Map each feature to its code module directory.

**Step 4 — Business logic**
Read core service/controller/handler files. Identify: special business rules, validation logic, calculation algorithms, workflow state machines, permission models.

**Step 5 — Configuration & flags**
Check: config files, `.env.example`, feature flags, version/edition indicators (free/pro/enterprise tiers), multi-tenant patterns, billing/pricing config.

**Step 6 — Non-functional qualities**
Look for: performance patterns (caching, indexing, pagination), security measures (auth, encryption, CORS), observability (logging, monitoring, analytics/tracking SDK), error handling patterns.

**Step 7 — Theme extraction (icon/favicon/color analysis)**
Search for and analyze all brand assets:
- **Icons & favicons**: Find `favicon.ico`, `favicon.svg`, `apple-touch-icon.*`, `logo.*`, `icon.*`, `public/icons/*`, manifest icons. Read SVG files to extract colors. For raster images (PNG/JPG/ICO), use the image analysis tool to identify dominant colors and brand palette.
- **CSS/SCSS/Tailwind colors**: Extract from `tailwind.config.*`, CSS custom properties (`:root { --primary: ... }`), theme files. These are secondary to the icon/favicon colors.
- **Brand palette derivation**: From the icon/favicon analysis, derive a 3-5 color palette: primary (dominant), secondary (accent), neutral (text/background), highlight (CTAs/emphasis). This palette drives all slide styling.
- **SVG preprocessing for PPTX**: SVG files containing filter effects (`feGaussianBlur`, `feFlood`, `feBlend`, `feDropShadow`, etc.) will not render correctly in PPTX — they produce visible borders/artifacts. After identifying brand assets, check SVGs for filter complexity:
  1. If the SVG is simple (just `<path>`, `<circle>`, `<rect>`, `<polygon>` with fills/strokes), copy as-is to `images/` and reference via `<img>`.
  2. If the SVG contains filter effects, extract the main visual paths/shapes (skip `<mask>`, `<filter>`, `<defs>` effect layers) and generate a simplified transparent PNG using `sharp`. Save to `images/` as `favicon.png` (or equivalent) and reference the PNG in slides instead of the SVG.
  3. Command to convert: `sharp(Buffer.from(simplifiedSvg)).resize(192, null, { fit: 'contain', background: { r: 0, g: 0, b: 0, alpha: 0 } }).png().toFile('images/favicon.png')`
- **If no brand assets found**: Fall back to role-appropriate default palettes (see Phase 5).

**Step 8 — Industry domain identification**
From the tech stack, dependencies, business logic, and variable/model naming, determine what industry or domain this product operates in (e.g., e-commerce, fintech, healthcare, EdTech, DevOps, logistics). Identify domain-specific concepts, regulatory context, and industry norms embedded in the code. If the domain is ambiguous, search the web for the product name or key terms to clarify.

**Step 9 — Domain glossary extraction**
From code, comments, config keys, model/entity names, and API parameters, extract domain-specific terms and acronyms. For each term: the code context where it appears and a plain-language explanation. Include abbreviations and their expansions. This becomes the glossary section of the deck.

**Step 10 — Doc cross-reference**
Read README, CHANGELOG, `docs/` if present. Compare claims against actual implementation. Record any discrepancies — these are important for Developer and PM roles.

**Output**: A structured analysis held in conversation context (not a file), covering all the above. This feeds directly into content generation and theme extraction.

### Phase 3: Competitive Research

**When required**: Product Operations, Sales, Partner/Reseller, End User, Business Development roles always need competitive context. Developer and PM roles only if explicitly requested.

**Search protocol** (4-5 rounds minimum):
1. **Product category** — search the general product category + "alternatives" / "competitors" / "竞品"
2. **Feature comparison** — search for specific feature comparisons within the category
3. **User reviews & sentiment** — search for positive and negative reviews, 口碑, pain points
4. **Pricing & packaging** — search for pricing models, tiers, free vs paid features
5. **Partnership models** (Partner/BD only) — search for how similar products do partnerships, integrations, reselling

**Competitor validation** — before listing something as a competitor, verify:
- Same target user segment? (CRM for dentists ≠ CRM for real estate)
- Same core problem? (project management ≠ time tracking, despite overlap)
- Same market tier? (enterprise ≠ SMB tools)

Failed validation → classify as "adjacent tool", not competitor.

**Honesty rule**: If searches yield no useful results for a dimension, state "未能获取到相关竞品信息" in the output. Never fabricate competitor data.

### Phase 4: Content Generation

Read `references/role-templates.md` for the selected role's content template. Generate slide content based on:
- Codebase analysis (Phase 2)
- Competitive research (Phase 3, if applicable)
- Role-specific requirements (from templates)

**Content must be audience-first**: Every slide should be organized from the target audience's cognitive journey, not from the product's feature list. Before writing each slide, ask: "If I were [target role] reading this for the first time, what would I need to see next to build understanding?"

**Role-specific content principles**:

| Role category | Content principle | What it means |
|--------------|-------------------|---------------|
| Business roles (Sales, BD, Operations, Partner, End User) | **画面感** (vividness) | Use concrete scenarios, stories, and before/after contrasts. The reader should be able to *picture* the product in use. Replace abstract descriptions with "imagine you're a sales manager who needs to..." framing. |
| Technical roles (Developer, PM) | **层次感** (layered depth) | Structure follows: **Why** (business context / problem) → **What** (architecture / feature overview) → **核心逻辑** (how it actually works — call chains, data flow, state machines) → **Detail** (file paths, config options, edge cases). Never jump to Detail without establishing the Why. |

**Role-specific generation rules**:

| Role | Special generation rules |
|------|------------------------|
| Developer | Map each feature to its code module. Trace entry points (routes/commands/events) through to service layer — show call chains. Inventory technical debt and optimization opportunities. |
| PM | End the deck with 3-4 feasible product direction suggestions, each with 2-3 concrete features. Directions should differ in strategy (deepen core vs. expand vertical vs. platform play vs. new market). |
| Operations | Map user behavior touchpoints from code. Identify A/B test capabilities, feature flags, and analytics events in the codebase. |
| Sales | Create a "Feature → Customer Value" conversion table. Identify pricing/tier signals from code (feature gates, edition checks). |
| End User | Write in user-facing language (no code jargon). Build a "can do / can't do" boundary list from actual feature implementation. |
| Partner | Support competitive claims with code-level evidence (e.g., "supports SSO via SAML/OIDC" backed by the auth module). Assess integration difficulty from API surface and deployment config. |
| BD | Include competitor partnership model comparison from web research. Flag any gaps in this product's partnership infrastructure. |

**Industry background & glossary rules** (apply to all roles):
- **Industry background**: Generated from Phase 2 Steps 8-9. Depth matches the role — Developers get technical context (protocols, standards, compliance requirements); Sales/Partner/BD get market context (market size, trends, regulatory landscape); End Users get practical context (why this industry has this problem, how this product fits).
- **Glossary**: Generated from Phase 2 Step 9. Terminology is role-adapted — Developers see technical terms (API names, protocol acronyms, design pattern names); non-technical roles see domain concepts explained in plain language. Always include the product's own terminology (feature names, internal concepts surfaced to users).

**Common slide structure** (adapt per role):

For **non-End-User roles** (Developer, PM, Operations, Sales, Partner, BD, and custom roles that are NOT the end consumer):
1. Cover — product name + tagline + target audience
2. **Industry background & user pain points** — domain context, market landscape, user pain points that create the problem this product solves. Establish *why the problem matters* before presenting the solution.
3. Product positioning — what problem, for whom, why this solution (now the audience has context to appreciate the positioning)
4. Core value proposition — the "why should I care" for this role
5. Role-specific content (5-12 slides, see role templates)
6. Glossary — domain and product terminology reference (1-2 slides, role-adapted)
7. References — all cited sources with links (see References section below)
8. Summary / Call to action

For **End User role**:
1. Cover — product name + tagline
2. Product positioning — what problem, for whom, why this solution (users already know their own pain)
3. Core value proposition — what you can do with this product
4. Feature walkthrough (progressive: most-used features first)
5. "Can do / Can't do" boundary
6. Glossary — plain-language feature names and concepts
7. Summary / Next steps

**Why industry-first for non-End-User roles**: Roles like Sales, BD, Partner, and even Developer/PM need to understand the *problem space* before they can appreciate the *solution*. Without industry context first, product positioning lacks anchor — the audience has no frame of reference for "why should I care about this product's features". End Users already live in the problem space; they don't need the industry explained to them.

**References & citations**:
Every factual claim from external sources (competitive research, market data, pricing info) must have an inline citation in the slide content. Example: `[1]`, `[2]`, etc. The final References slide lists all sources:
```
[1] Competitor X Feature Comparison — https://...
[2] Market Size Report 2026 — https://...
[3] README — local: /path/to/README.md
```
For code-derived facts, cite the source file: `Source: app/api/tasks/route.js:45-78`. Code sources don't need the References slide entry — just inline file references.

**UI screenshots** (for Sales, Operations, End User, Partner, and BD roles):
When slides need to show product usage flows or interface screenshots:
1. Try to start the dev server: `npm run dev` (or equivalent from codebase analysis)
2. Wait for the server to be ready, then take screenshots of key UI pages using Playwright or the system screenshot tool
3. Save screenshots to `images/` directory and reference via `<img>` tags in slides
4. If the server cannot be started (missing .env, database, external deps), or screenshots cannot be captured: use a placeholder image — a gray rectangle with centered text describing what should be there: "Screenshot: [页面名称/功能描述] — 请手动替换此占位图". Use the same 960×540pt body context for consistent sizing.

### Phase 5: Content Quality Loop (4-5 Iterations)

This phase runs for ALL roles. It ensures content is optimized for the target audience through blind evaluation by an independent subagent.

**Why**: Content generated from codebase analysis is accurate but often not audience-friendly. The blind reviewer catches jargon, unclear structure, missing context, and pacing issues that the author (who has codebase context) cannot see.

**Loop procedure** (4-5 rounds):

```
Round 1: Generate initial HTML slides (Phase 4 output → Phase 6 HTML rules applied)
    ↓
Round 1 Review: Spawn blind reviewer subagent
    ↓
Round 2: Optimize based on review feedback
    ↓
Round 2 Review: Spawn blind reviewer subagent
    ↓
... repeat for 4-5 rounds total ...
    ↓
Final Selection: Pick the version with highest overall score
```

**Blind reviewer subagent specification**:

The blind reviewer is a **separate agent** (spawned via Agent tool) that:
- Has **NO codebase context** — it only sees the generated HTML slides
- Simulates being the **target role** (auto-inferred from deck content and tone)
- Reads every slide as if encountering the product for the first time
- Scores on role-specific dimensions (see below)
- Provides specific, actionable improvement feedback per slide

**Role-specific evaluation dimensions**:

**Business roles** (Sales, BD, Operations, Partner, End User) — evaluate 画面感 (vividness):

| Dimension | What it checks | Min |
|-----------|---------------|-----|
| Product comprehension | Can I explain what this product does in one sentence? | 8 |
| Scenario vividness | Can I *picture* someone using this? Do scenarios feel real, not abstract? | 8 |
| Value clarity | Can I articulate 2-3 specific benefits with concrete outcomes? | 8 |
| Language accessibility | Is the language MY language (not developer jargon)? Can I repeat this to a colleague? | 8 |
| Boundary awareness | Do I know what this product does NOT do? | 7 |
| Actionability | Do I know what to do next after reading? | 8 |

**Technical roles** (Developer, PM) — evaluate 层次感 (layered depth):

| Dimension | What it checks | Min |
|-----------|---------------|-----|
| Problem context (Why) | Do I understand WHY this system exists and what problem it solves? | 8 |
| Architecture overview (What) | Can I describe the high-level architecture and module boundaries? | 8 |
| Core logic understanding | Do I understand the key algorithms, state machines, and data flows? | 8 |
| Detail depth | Are the file paths, call chains, and code references specific enough to be useful? | 8 |
| Doc/code accuracy | Are discrepancies between docs and code identified and explained? | 7 |
| Actionability | Do I know where to start working on this codebase? | 8 |

**Subagent prompt template** (for spawning the blind reviewer):

```
You are simulating a [target role] with ZERO prior knowledge of this product.
You have NEVER seen this codebase. You only have the slides below.

Read each HTML slide file in order. After reading all slides, score the
presentation on the following dimensions (0-10 each):

[Insert the appropriate dimension table above based on role category]

For each dimension:
1. Score 0-10
2. Quote specific slide content that helped or confused you
3. If score < minimum, explain exactly what's missing and what would improve it

Output format:
{
  "round": N,
  "target_role": "...",
  "dimensions": [
    {"name": "...", "score": N, "min": N, "passed": true/false,
     "evidence": "...", "improvement_suggestion": "..."}
  ],
  "overall_score": N,
  "worst_slides": ["slide_file.html: reason"],
  "best_slides": ["slide_file.html: reason"]
}
```

**Optimization rules per round**:
- Only rewrite slides that the reviewer flagged as worst
- Do NOT change slides the reviewer rated as best (they're working)
- Focus on the reviewer's specific improvement suggestions
- Track scores per round in `content-quality-scores.json`
- After round 4-5, select the version with the highest `overall_score`
- If scores plateau (no improvement between rounds), stop early

**Output**: The winning version's HTML slides proceed to Phase 6. The scorecard is saved to `content-quality-scores.json` alongside the slides.

> **Note**: The blind reviewer subagent is spawned by the main agent using the Agent tool. It does NOT have access to the codebase — it only reads the HTML slide files in the output directory.

### Phase 6: HTML Slide Generation

Each slide becomes an independent HTML file. Read `references/pptx-constraints.md` for the full constraint rules and HTML template.

**Critical**: These constraints are physical limitations of the PPTX format, not stylistic preferences. Violating them causes export failure. Even if PPTX export is not available, follow these constraints — they produce cleaner HTML slides and keep the option open for future PPTX conversion.

Key constraints:
- Body: `width: 960pt; height: 540pt` (matches `LAYOUT_WIDE`)
- All text must be inside `<p>` or `<h1>`-`<h6>` tags (never bare text in `<div>`)
- No CSS gradients — solid colors only
- Backgrounds, borders, and shadows go on `<div>` elements, never on `<p>` or `<h*>` tags
- Images use `<img>` tags, never CSS `background-image`
- No manual bullet symbols (`-`, `*`, `•`) or numbered patterns (`1.`, `2.`) in `<p>` tags — use `<ul>/<li>` or `<ol>/<li>` instead
- No inline SVG elements — they cause `className.includes is not a function` errors in the export script. Save SVG files to `images/` and reference via `<img>` tags
- Anti-overflow: leave 30pt bottom margin, no more than 4-5 content blocks per slide, use 10-11pt for dense content. PPTX doesn't clip overflow like HTML does. If content feels tight, split into two slides.
- **Layout rule (Rule 11)**: Use ONE `position: absolute` wrapper for the content area, then flow layout inside (flex, margin, padding). Do NOT use `position: absolute` with hardcoded `top` for individual content blocks — Chinese text wraps unpredictably and causes overlap. See `references/pptx-constraints.md` Rule 11 for the correct pattern.
- **Call chains / flow diagrams (Rule 12)**: Do NOT use `display: flex; flex-wrap: wrap` for call chains or flow diagrams. PPTX has no flex layout — each flex child becomes an independently positioned element, and PPTX's wider text rendering shifts wrap points. Use a single `<p>` with `<span>` elements for inline color coding instead. Each chain becomes one text element in PPTX.
- **Inline colored text (Rule 13)**: Use `<span>` for inline colored text with `font-weight: 700` — never `<h1>`-`<h6>` with `display: inline`. The export script processes heading tags as standalone text elements, causing double-rendering. This applies inside `<li>` items AND as inline siblings in glossary/term cards.
- **Font family (Rule 14)**: Use explicit font names — `"PingFang SC", "Microsoft YaHei", Arial, sans-serif`. Never use generic families (`system-ui`, `-apple-system`) because the export script takes `fontFamily.split(',')[0]` and passes it to PowerPoint, which doesn't recognize generic names and falls back to Calibri/SimSun. This font mismatch (PingFang SC in HTML vs Calibri in PPTX) causes width differences far exceeding what Rule 10 compensates for.
- Text width: PPTX renders text ~5-10% wider than HTML. Use 85-90% of visual container width for body text, **75-80% for Chinese titles/headers** (Chinese characters are wider and titles wrap first). Text that fills edge-to-edge in HTML WILL wrap in PPTX.

**Style decisions**:
- **Primary source**: Use the brand palette derived from icon/favicon analysis (Phase 2 Step 7). This is the authentic brand representation and takes precedence over role defaults.
- **Fallback**: If no brand assets were found, choose a palette appropriate for the role:
  - **Developer**: Dark theme, code-editor aesthetics (dark bg #1E1E2E, syntax-highlighting accents #7AA2F7 #9ECE6A)
  - **PM**: Clean blue/white (#0A66C2 accent, white bg), data-driven feel
  - **Sales/BD**: Bold, confident (deep blue #1A3A5C + gold/orange #E8A838 accents)
  - **End User**: Warm, friendly (light bg #FAFAFA, soft accents #4A90D9 #67B7DC)
  - **Partner**: Professional warmth (navy #1B2A4A + teal #2D9CDB)
  - **Operations**: Data-viz friendly (white bg #FFFFFF, chart colors #4E79A7 #F28E2B #E15759)

**Output structure**:
```
<workspace>/codebase-deck-<role>/
├── index.html           (HTML slide viewer — always generated)
├── slides/
│   ├── 01-cover.html
│   ├── 02-positioning.html
│   ├── 03-value-proposition.html
│   ├── ...
│   └── NN-summary.html
└── images/              (any <img> referenced assets)
```

**Generating index.html**: Copy `assets/deck_index.html` to the deck root as `index.html`, then edit the `DECK_MANIFEST` array to list all generated slides with their file paths and labels. The viewer provides keyboard navigation (arrow keys, space, number keys), auto-scaling to browser window, slide counter, and print-to-PDF support. Canvas size is 960×540 (matching the slide HTML body dimensions).

> **Checkpoint 2**: After generating HTML slides and index.html, tell the user the count and optionally preview one. The index.html can be opened in any browser for immediate review. Wait for feedback before proceeding to Phase 7.

### Phase 7: Export (PPTX if available, HTML always)

This phase has two paths depending on whether the huashu-design skill is installed.

**Step 1 — Detect huashu-design export capability**

Check if the PPTX export script exists by searching common skill installation paths:

```bash
# Check these paths in order:
find ~/.agents/skills/huashu-design/scripts/export_deck_pptx.mjs \
      ~/.claude/skills/huashu-design/scripts/export_deck_pptx.mjs \
      -type f 2>/dev/null
```

**Path A — huashu-design is installed** (export script found):

Run the PPTX export:

```bash
node <path-to>/export_deck_pptx.mjs \
  --slides <workspace>/codebase-deck-<role>/slides \
  --out <output-path>/<product-name>-<role>.pptx
```

Prerequisites: `npm install pptxgenjs playwright sharp` (if not already installed).

If any slide fails export, check the error against the constraint rules in `references/pptx-constraints.md` and fix the HTML. Common fixes:
- Bare text in div → wrap in `<p>`
- CSS gradient → replace with solid color
- Background on `<p>` → move to wrapping `<div>`

Deliver both the .pptx file AND the index.html. Tell the user to open the PPTX and verify text is editable (double-click any text).

**Path B — huashu-design is NOT installed** (export script not found):

The deck is already complete as HTML (index.html + slides/). Tell the user:

> "PPTX export is not available because the huashu-design skill is not installed. Your deck is ready as an HTML presentation — open index.html in a browser. You can navigate with arrow keys and print to PDF.
>
> To get editable PPTX output, install the huashu-design skill:
> 1. Open Claude Code settings (type /help for guidance)
> 2. Add the huashu-design skill
> 3. Re-run this skill — it will auto-detect and export PPTX
>
> Want me to help install it now?"

If the user says yes to installation, guide them through it or attempt to install automatically if possible.

**Why HTML is always generated**: The HTML output is the primary artifact — it's portable, requires no special software, supports keyboard navigation and print-to-PDF. PPTX is a convenience export for teams that need to edit slides in PowerPoint. Both outputs serve different use cases, so both should be available when possible.

---

## Fact Priority Rules

Priority order for **product naming and positioning**:
1. **UI truth (highest)** — product names, labels, titles visible in HTML/UI templates, page titles, navigation labels, user-facing strings in the frontend code
2. **Code truth** — running code, config files, database schemas, route definitions, API contracts
3. **Doc truth** — README, CHANGELOG, docs/, code comments
4. **External truth** — web search results for competitor/market data

When multiple sources name the product differently, use the UI truth. If the UI says "TTRP工作台" but README says "Tobee TRP System", the deck should use "TTRP工作台" (with "Tobee TRP System" as an alias if relevant).

Conflict handling for **technical facts** (features, architecture, tech stack):
- **Developer & PM roles**: Flag every discrepancy with both the doc claim and the actual implementation. Example: "README claims PostgreSQL support, but code only has MySQL adapters (see `src/db/mysql.ts`). Current implementation: MySQL only."
- **All other roles**: Use code truth silently. External audiences should not see internal inconsistencies.

---

## Custom Role Handling

When the user specifies a role not in the 7 presets:

1. **Infer the role's relationship** to the product:
   - Internal stakeholder? (leadership, finance, legal)
   - External partner? (integrator, consultant, auditor)
   - End consumer? (specific demographic, industry vertical)
2. **Generate 5-8 focus areas** based on what that role needs to know about the product.
3. **Present to user**: "我推测[角色名]可能关注以下方面，请确认或补充：[list]"
4. After confirmation, proceed through Phases 2-6 using the custom focus areas.

---

## Bundle Reference

| File | Purpose | When to read |
|------|---------|-------------|
| `references/role-templates.md` | Detailed content templates for all 7 roles | Phase 1 (role confirmation) and Phase 4 (content generation) |
| `references/pptx-constraints.md` | PPTX HTML 4-constraint rules, template, common errors | Phase 6 (HTML slide generation) |
| `assets/deck_index.html` | HTML slide viewer template (fallback when huashu-design unavailable) | Phase 6 (generate index.html) |

**Optional external dependency**: If the huashu-design skill is installed, Phase 7 uses its `export_deck_pptx.mjs` script for PPTX export. The script is discovered at runtime via `find` — no hardcoded paths.
