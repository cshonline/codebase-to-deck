# codebase-to-deck

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that analyzes any codebase and generates a role-tailored, editable PPTX presentation.

Point it at a code repository, tell it who the audience is, and it produces a complete slide deck — from code analysis through competitive research to polished HTML slides with optional PPTX export.

## What it does

1. **Codebase Analysis** — Reads source code to understand features, architecture, tech stack, business logic, and brand identity
2. **Competitive Research** — Web search for market context (when the audience needs it)
3. **Content Generation** — Role-specific content following audience-first principles
4. **Content Quality Loop** — Blind reviewer subagent scores and iterates slides for clarity
5. **HTML Slide Generation** — Constrained HTML that converts cleanly to PPTX
6. **PPTX Export** — Optional export via [huashu-design](https://github.com/anthropics/huashu-design) integration, always outputs viewable HTML

## Supported Roles

7 preset audience personas, plus custom roles with auto-inferred focus areas:

| Preset | Target Audience |
|--------|----------------|
| Developer | Engineering team |
| Product Manager | PM / product leadership |
| Product Operations | Ops / growth team |
| Sales | Sales team / prospects |
| End User | Terminal users |
| Partner/Reseller | Channel partners |
| Business Development | BD / partnership team |

## Installation

```bash
# Clone this repo into your Claude Code skills directory
git clone https://github.com/cshonline/codebase-to-deck.git ~/.claude/skills/codebase-to-deck
```

The skill will be automatically available in Claude Code. Trigger it by asking to generate a presentation from a codebase.

## Usage

```
# In Claude Code, navigate to your codebase and say:
"帮我把这个项目做成一份给销售团队看的 PPT"

# Or in English:
"Create a presentation for the sales team from this codebase"
```

## PPTX Export (Optional)

For editable PowerPoint output, also install [huashu-design](https://github.com/anthropics/huashu-design):

```bash
git clone https://github.com/anthropics/huashu-design.git ~/.agents/skills/huashu-design
cd ~/.agents/skills/huashu-design && npm install
```

Without huashu-design, the skill generates HTML slides with a built-in viewer (keyboard navigation, print-to-PDF support).

## Project Structure

```
codebase-to-deck/
├── SKILL.md                        # Main skill instructions (7-phase workflow)
├── pitfalls.md                     # Development log of issues encountered
├── references/
│   ├── pptx-constraints.md         # 14 rules for HTML→PPTX compatibility
│   └── role-templates.md           # Content templates for each role
└── assets/
    └── deck_index.html             # Bundled HTML slide viewer
```

## Key Design Decisions

- **Code is primary truth** — Running code overrides README/docs when they conflict
- **Audience-first content** — Business roles get vividness (scenarios, stories); technical roles get layered depth (why→what→how→detail)
- **14 PPTX constraint rules** — HTML slides are constrained at generation time to ensure clean PPTX conversion (font families, layout patterns, text width)
- **Flow layout, not absolute positioning** — Content blocks use flex/margin flow layout to prevent overlap when text wraps
- **Explicit font families** — Uses `PingFang SC` / `Microsoft YaHei` instead of `system-ui` to ensure consistent rendering between HTML preview and PPTX output

## Acknowledgments

This skill builds on the excellent work of:

- **[huashu-design](https://github.com/anthropics/huashu-design)** — The HTML-to-PPTX conversion pipeline (`html2pptx.js`) that makes editable PowerPoint export possible. codebase-to-deck's `references/pptx-constraints.md` was derived from studying huashu-design's validation rules, text width compensation (2% single-line expansion), and font extraction behavior. The PPTX export in Phase 7 directly invokes huashu-design's `export_deck_pptx.mjs` script.

## License

MIT
