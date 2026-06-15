# PM OS — Product Management Operating System Template

An open-source repository template for running your product management practice with AI agents. Fork it, fill in your company and product context, and let Claude Code generate PRDs, link them to Jira, and publish them to Confluence — using your customer inputs as evidence.

## What This Is

PM OS gives you a structured workspace with three layers:

1. **Context** — Background on your company and products that AI agents read before generating anything.
2. **Inputs** — Raw customer feedback, meeting notes, transcripts, and research, tagged with YAML frontmatter so you can find what's relevant per project.
3. **Projects** — Per-project workspaces (`brief.md`, `pm-notes.md`, `input-references.md`, `prd.md`) generated and updated through Claude Code skills.

## Prerequisites

- Claude Code (or another Claude-compatible client) installed and authenticated.
- An Atlassian account with API access — only required if you plan to publish to Confluence or link to Jira.
- A forked copy of this repo.

## Quickstart — One-Time Setup

```bash
git clone <your-fork-url> pm-os
cd pm-os
```

Open the repo in Claude Code and run:

1. **`/setup-pm-os`** — Website-first interactive walkthrough. Starts from your company website and volunteered sources, uses web crawl/search to infer company facts and product candidates, asks you to confirm gaps or conflicts, then drafts company/product context. It can optionally enrich `strategy.md`, seed selected feature files, and configure Atlassian/Confluence publishing defaults at the end. Updates context files after review, copies the product template into `context/products/<your-product>/`, writes `pm-os.config.yml` only if you opt into publishing defaults, and prints a gap checklist.
2. **`/add-context company`** — Flesh out `about-company.md` (Mission, ICP, Positioning, Differentiators, Competitors) and `strategy.md` (Market Trends, Strategic Bets, Risks). Accepts URLs, Confluence pages, pasted text, files, or conversation; proactively asks for high-value missing sources and reports sampled/failed/deferred pages.
3. **`/add-context product`** — Flesh out your product overview (Strategy/Positioning, Unique Approach pillars, Key Capabilities, Target Users, ICP, Product Principles, Key Metrics, Competitive Landscape, Differentiation) from product pages, docs/help sites, competitor sources, pasted material, or direct answers.

You're now ready to run projects. Setup is a one-time exercise — refresh context with `/add-context` whenever something material changes.

## Running a Project

Once setup is done, every project follows the same loop:

```
/start-project ─▶ /add-input + /import-inputs ─▶ /product-brainstorming ─▶ /solution-architect ─▶ /prd-generator
                                                                                                      │
                                                              /publish-to-jira  ◀──┴──▶  /publish-to-confluence
```

1. **`/start-project <Project Name>`** — Scaffolds `projects/<Project Name>/` with `brief.md`, `pm-notes.md`, and `input-references.md`. Capture the problem statement, goals, and scope in `brief.md` before moving on.
2. **`/add-input`** and **`/import-inputs`** — Use `/add-input` to file any new raw input (customer feedback, transcript, meeting notes, Slack thread, competitor teardown) into `inputs/` with proper frontmatter. Then use `/import-inputs` to scan `inputs/` for everything relevant to this project and link it into `input-references.md`.
3. **`/product-brainstorming`** — Explore the problem with first-principles thinking, root-cause analysis, and divergent idea generation. Captures conclusions into `pm-notes.md`.
4. **`/solution-architect`** — Define and evaluate solution approaches; produce a `solution-design.md` covering trade-offs, dependencies, and the recommended approach.
5. **`/prd-generator`** — Generate a lean, job-focused PRD. It asks 10–15 clarifying questions before writing, then produces `prd.md` with acceptance criteria, business logic, and frontmatter blocks for Confluence + Jira.
6. **`/publish-to-confluence`** — Publish the PRD to Confluence using defaults from `pm-os.config.yml`. Stores the resulting `page_id` back into PRD frontmatter so future updates target the same page. Use `/update-on-confluence` afterward for revisions.
7. **`/publish-to-jira`** — Link the PRD to a Jira issue (Epic, Task, or Story). Stores `issue_key` and `issue_url` back into PRD frontmatter.

> Steps 6 and 7 are independent — run them in either order, or skip whichever you don't need. Publishing defaults live in `pm-os.config.yml`; per-PRD overrides go in the PRD frontmatter.

## Commands Reference

Run inside Claude Code (`/<command-name>`). For full operating rules — context loading order, frontmatter schemas, PRD guidelines — see [`CLAUDE.md`](./CLAUDE.md).

### Core workflow

| Command | What it does |
|---------|--------------|
| `/setup-pm-os` | Website-first setup: source-backed company facts, first product candidates, company/product context, optional strategy, selected feature seeding, and optional publishing defaults |
| `/add-context` | Add or enrich one company / product / feature context doc with proactive source asks and bounded crawl reporting |
| `/start-project <name>` | Scaffold a new project workspace from templates |
| `/add-input` | File a raw input into `inputs/` with proper frontmatter |
| `/import-inputs` | Pull relevant existing inputs into a project's `input-references.md` |
| `/product-brainstorming` | Root-cause analysis, first principles, idea generation |
| `/solution-architect` | Define and evaluate solution approaches |
| `/prd-generator` | Generate a lean, job-focused PRD (10–15 clarifying questions first) |
| `/publish-to-confluence` | Publish a PRD to Confluence; stores `page_id` in frontmatter |
| `/publish-to-jira` | Link a PRD to a Jira issue; stores `issue_key` / `issue_url` |

### Side quests

| Command | What it does |
|---------|--------------|
| `/competitive-analysis` | Structured competitor research and capability comparison |
| `/documentation-agent` | Convert PRDs into dev / design / QA docs |
| `/ux-expert` | Usability heuristics, design review, interaction analysis |
| `/design-review` | Compare designs against PRDs for completeness |
| `/screen-diagrams` | ASCII wireframes of product screens, kept in `context/products/` |
| `/wireframes` | ASCII wireframes and user flows for a specific project |
| `/lovable-structured` | Generate structured Lovable prompts for prototyping |
| `/update-on-confluence` | Update an existing Confluence page |

## Repository Structure

```
├── context/                     # Company & product knowledge base (AI reads these)
│   ├── company/
│   │   ├── about-company.md     # Company overview, ICP, competitors
│   │   └── strategy.md          # Strategy, market trends, strategic bets
│   └── products/                # One folder per real product (created by /setup-pm-os or /add-context)
│
├── inputs/                      # Raw inputs by quarter
│   ├── SCHEMA.md                # Canonical frontmatter values
│   └── YYYY-QN/
│       ├── customer-feedback/
│       ├── meeting-notes/
│       ├── transcripts/
│       ├── competitor-analysis/
│       └── secondary-research/
│
├── projects/                    # One folder per project: <Project Name>/
│   └── <Project Name>/
│       ├── brief.md             # Problem, goals, scope
│       ├── pm-notes.md          # PM thinking
│       ├── input-references.md  # Links to relevant inputs
│       └── prd.md               # Generated PRD
│
├── templates/                   # Reusable templates
│   ├── projects/                # Project workspace files (PRDs, briefs)
│   ├── inputs/                  # Per-input-type templates
│   ├── products/                # Blank product folder template
│   │   └── example-product/
│   │       ├── overview.md
│   │       └── example-feature.md
│   └── lovable/                 # Lovable prompt scaffolding
│
├── pm-os.config.yml             # Publishing defaults (Atlassian domain, Jira key, Confluence space)
├── plans/                       # Working plans from plan mode
├── docs/                        # publishing docs plus shared command contracts
│   └── command-contracts/        # docs/command-contracts/ shared prompt contracts used by slash commands
└── .claude/commands/            # Project-level Claude Code skills
```

## Learn More

- [`CLAUDE.md`](./CLAUDE.md) — Operating manual for AI agents working in this repo.
- [`docs/confluence-publishing.md`](./docs/confluence-publishing.md) — Confluence publishing guide.
- [`docs/confluence-content-filtering.md`](./docs/confluence-content-filtering.md) — How PRD content is filtered before publishing.
- [`docs/command-contracts/context-synthesis.md`](./docs/command-contracts/context-synthesis.md) — Shared synthesis contract used by `/setup-pm-os` and `/add-context`.
- [`inputs/SCHEMA.md`](./inputs/SCHEMA.md) — Canonical frontmatter values for inputs.
