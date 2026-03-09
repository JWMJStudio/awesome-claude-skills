# Awesome Claude Skills — Review Notes

> Reviewed from codespace for security and usefulness.
> Context: CCO building AI-first team — marketing chefs to private dining hosts, enterprise tools for creatives and chefs.

---

## Security Review

- CodeQL ran clean (JS/TS + Python) — zero alerts
- Manual scan: no `eval()`, `exec()`, hardcoded secrets, `.env` files, or suspicious network calls
- Shell scripts and `subprocess` usage reviewed — all legitimate (calling known CLI tools with list-form args)
- **832 composio-skills are thin Rube MCP wrappers** — no real logic, just instruction templates pointing to a third-party service. No security concern, but no real value either.
- 28 top-level curated skills are the actual useful content

---

## Repo Structure

```
awesome-claude-skills/
├── 28 curated skill directories (the real value)
│   ├── SKILL.md         — core instructions (<5k words)
│   ├── scripts/         — executable utilities
│   ├── references/      — docs loaded on demand
│   └── assets/          — templates, fonts, boilerplate
├── composio-skills/     — 832 Rube MCP wrappers (skip these)
├── connect-apps/        — Composio plugin for 500+ app integrations
├── document-skills/     — docx, pdf, pptx, xlsx manipulation
└── README.md
```

---

## Key Patterns Learned (for building our own skills)

### 1. Progressive Disclosure (most important pattern)
- **Tier 1**: YAML frontmatter — name + description (~100 words, triggers activation)
- **Tier 2**: SKILL.md body — core instructions (<5k words, loaded on trigger)
- **Tier 3**: references/ and scripts/ — loaded only when Claude needs them
- Assets (templates, fonts) are never loaded into context — only used for output

### 2. Workflow-Centric Instructions
Structure skills as step-by-step workflows, not feature lists:
- Each step has a clear input, action, and testable output
- Use decision trees for branching logic
- Provide concrete output examples, not abstract descriptions

### 3. Imperative Voice
- Use: "To find available chefs, filter Chefs Roster by Availability"
- Avoid: "You should filter the roster"

### 4. Explicit Reference Loading
Don't assume Claude reads everything. Direct it:
- "Load [ooxml.md] completely before editing any document"
- "Call RUBE_SEARCH_TOOLS first to get current schemas"

### 5. Workspace Context is Power
Best skills leverage the local environment:
- Filesystem structure (file-organizer)
- Git history (changelog-generator)
- Chat history (developer-growth-analysis)
- Run skills from directories where relevant data lives

### 6. Evaluation-Driven Design (from mcp-builder)
Write realistic test scenarios BEFORE finalizing the skill. Design for actual agent workflows, not just API coverage.

---

## Notion MCP Instructions Template

The official Notion MCP handles API calls. What it needs from us is a **workspace map**. Add this to CLAUDE.md or a skill:

```markdown
# Notion Workspace Guide

## Key Databases (with IDs)
- **Chefs Roster**: `<database-id>` — Name, Cuisine, Rate, Availability, Portfolio, Rating
- **Host Directory**: `<database-id>` — Name, Location, Event Types, Budget, Dietary Prefs
- **Bookings**: `<database-id>` — Chef, Host, Date, Menu, Status, Invoice
- **Menu Library**: `<database-id>` — Dish, Cuisine, Course, Dietary Tags, Chef

## Key Pages
- **Operations Hub**: `<page-id>`
- **Marketing Assets**: `<page-id>`
- **Client Onboarding**: `<page-id>`

## Terminology
- "Match" = chef-host pairing for an event
- "Tasting" = trial booking before host commits
- "Drop" = chef's availability window for the week

## Common Workflows
- Find available chefs: filter Chefs Roster → Availability = Open + Cuisine Type
- Create booking: add to Bookings → link Chef + Host → Status = "Pending"
- Update chef profile: find by name in Chefs Roster → update fields

## Relations
- Bookings → links to Chefs Roster and Host Directory
- Chefs Roster → rollup of Bookings count + avg Rating
```

---

## Skills Useful to Our Business

### Directly Useful — Copy and Adapt

| Skill | What It Does | Our Use Case |
|-------|-------------|--------------|
| **lead-research-assistant/** | Identifies leads, scores them, generates outreach | Find private dining hosts, qualify by budget/location/event type |
| **content-research-writer/** | Collaborative blog/newsletter writing with research + citations | Chef profiles, menu stories, host-facing content, SEO articles |
| **canvas-design/** | Creates PDFs and visual art with design philosophy | Chef portfolios, tasting menus, pitch decks, event lookbooks |
| **invoice-organizer/** | Extracts info from invoices, renames, organizes, generates CSV | Chef billing, event expense tracking, tax prep |
| **competitive-ads-extractor/** | Scrapes Facebook/LinkedIn ads, analyzes messaging patterns | Study competitor private dining platforms, catering services |
| **brand-guidelines/** | Applies consistent colors + typography | All chef and host facing materials |
| **internal-comms/** | Templates for 3P updates, newsletters, status reports | Team updates, investor comms, weekly ops reports |
| **tailored-resume-generator/** | Analyzes job description, generates tailored resume | Adapt for chef profile generation tailored to host preferences |

### Build Tools — Use to Create Our Own

| Skill | What It Does | Our Use Case |
|-------|-------------|--------------|
| **mcp-builder/** | Guide for creating MCP servers | Build custom MCPs (booking system, chef-host matching, menu API) |
| **skill-creator/** | Guide for packaging Claude skills | Package our workflows as reusable skills for the team |
| **template-skill/** | Starter template | Base for any new skill |
| **artifacts-builder/** | React/Tailwind single-HTML artifacts | Interactive chef menus, booking forms, dashboards |

### Document & Data Tools

| Skill | What It Does | Our Use Case |
|-------|-------------|--------------|
| **document-skills/pptx/** | Create/edit PowerPoint slides | Investor decks, event proposals, chef showcases |
| **document-skills/docx/** | Create/edit Word docs with tracked changes | Contracts, NDAs, partnership agreements |
| **document-skills/xlsx/** | Spreadsheets with formulas and charts | Financial models, pricing sheets, event budgets |
| **document-skills/pdf/** | Extract, merge, annotate PDFs | Process signed contracts, health permits, insurance docs |

### Operations & Team

| Skill | What It Does | Our Use Case |
|-------|-------------|--------------|
| **meeting-insights-analyzer/** | Analyzes transcripts for behavioral patterns | Team dynamics, client call reviews, sales coaching |
| **file-organizer/** | Intelligent file/folder organization | Keep chef assets, contracts, photos organized as we scale |
| **changelog-generator/** | User-facing release notes from git history | Product updates for our platform tools |

### Marketing & Growth

| Skill | What It Does | Our Use Case |
|-------|-------------|--------------|
| **domain-name-brainstormer/** | Generates names + checks availability | Brand naming for sub-brands, event series, chef collectives |
| **image-enhancer/** | Upscales and sharpens images | Chef food photography for marketing materials |
| **slack-gif-creator/** | Animated GIFs for Slack | Internal team culture, celebrations |

---

## Skills NOT Useful (Skip These)

- **composio-skills/** (all 832) — Rube MCP wrappers with no real logic
- **connect-apps/** / **connect-apps-plugin/** — Only useful if you want Composio as middleman (we use direct MCPs)
- **langsmith-fetch/** — LangChain debugging, not relevant
- **developer-growth-analysis/** — Developer-specific analytics
- **video-downloader/** — Downloads YouTube videos, not our workflow
- **raffle-winner-picker/** — Niche utility
- **twitter-algorithm-optimizer/** — Twitter/X specific, may revisit if we market there
- **webapp-testing/** — QA testing tool

---

## Action Items

- [ ] Copy useful skills to personal repo for customization
- [ ] Build Notion workspace map (database IDs, page IDs, terminology)
- [ ] Use mcp-builder/ to plan custom MCPs (booking system, chef-host matching)
- [ ] Use skill-creator/ to package our first internal skill (chef profile generator?)
- [ ] Adapt lead-research-assistant/ for host prospecting workflow
- [ ] Adapt invoice-organizer/ for chef billing pipeline
- [ ] Review canvas-design/ for chef portfolio template
