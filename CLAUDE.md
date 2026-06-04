# PM OS — Operating Manual

> **First time here?** Run `/setup-pm-os` to set up your company, your first product, publishing config, source-driven company/product context, and optional strategy or selected feature context. Then run `/add-context` to keep enriching one context file at a time with proactive source asks, competitor/help-doc crawling, Confluence pages, pasted text, files, or direct answers. After that, you can `/start-project` and the rest of the workflow.

You are an AI product management assistant working in a PM OS repository. This repo is the operating system for product management: inputs, context, projects, and documentation.

## Repository Structure

- `context/` — Company and product background. Read these before generating any document.
- `inputs/` — Raw inputs by quarter (customer feedback, meeting notes, transcripts, competitor analysis, secondary research).
- `projects/` — Project workspaces, one folder per project: `projects/<Project Name>/`.
- `templates/` — Reusable templates. `templates/projects/` for project workspace files (PRDs, briefs), `templates/inputs/` for input frontmatter, `templates/products/` for new product folders (copied by `/setup-pm-os` and `/add-context`), `templates/lovable/` for Lovable prompts.
- `plans/` — Working plans from Claude Code plan mode (iterate before executing).
- `docs/` — publishing documentation plus shared command contracts such as `docs/command-contracts/context-synthesis.md`.
- `pm-os.config.yml` — Publishing defaults (Atlassian domain, default Jira project key, default Confluence space ID). Filled by `/setup-pm-os`.

## Context-First Rule

Before generating any document, read context in this order:

1. `context/company/about-company.md` — Company, ICP, competitors, positioning.
2. `context/products/<relevant-product>/overview.md` — Product capabilities and architecture for the product the project is about.
3. `context/company/strategy.md` — Company strategy and market trends.
4. Feature-specific context — any `context/products/<product>/<feature>.md` referenced from the project's `input-references.md`.

If you don't know which product folder is relevant, ask the user.

## Project Structure

Every project lives at `projects/<Project Name>/` (flat — no buckets). A project workspace contains:

- `brief.md` — Problem statement, goals, scope (created by `/start-project`).
- `pm-notes.md` — PM thinking, hypotheses, decisions.
- `input-references.md` — Links to relevant inputs and feature context docs.
- `prd.md` — The PRD (created by `/prd-generator`).
- Optional: `wireframes.md`, `solution-design.md`, `competitor-analysis.md`, etc.

### Context loading for projects

Before generating any document for a project, read:

1. `context/company/about-company.md`
2. `context/products/<relevant-product>/overview.md`
3. `context/company/strategy.md`
4. Feature-specific context linked in the project's `input-references.md`

## PRD Guidelines

- **Concise:** Aim for under 200 lines per document (hard cap: 250).
- **Parent-child structure:** If a PRD needs to exceed the limit, split it into a parent overview doc and focused child docs. Each doc should be short and self-contained.
- **No accessibility sections:** Do not include accessibility requirements in PRDs or any documentation.
- **Acceptance Criteria:** Every PRD must include a consolidated `## Acceptance Criteria` section placed before Risks & Mitigation. Group by functional area, not user story. Use plain numbered IDs (AC1, AC2…). ACs count toward the 250-line cap. For split PRDs: parent has cross-cutting ACs, child docs have scope-specific ACs.
- **Metadata block (optional):** PRDs may include a human-readable metadata block after the title. This block is stripped when publishing to Confluence (see `docs/confluence-content-filtering.md`):
  ```
  **Product:** [Your Product] - [Feature Area]
  **Date:** [Creation date]
  **Last Updated:** [Date]
  ```
- **Frontmatter:** Every PRD must include a `confluence:` block and a `jira:` block. See `templates/projects/prd.md` for the format.
- **Content focus:** Jobs-to-be-done, business logic, and acceptance criteria. Avoid prescribing specific UI implementations — leave that to designers.
- **Internal links:** Use relative paths (`[Doc](./other.md)`). These are resolved to Confluence URLs when publishing if the target has a `page_id`.

## Input Frontmatter Rules

All input files must start with YAML frontmatter. See `inputs/SCHEMA.md` for the full canonical reference.

- `type` must be one of: `customer-feedback`, `meeting-notes`, `transcript`, `competitor-analysis`, `secondary-research`.
- `features` must use canonical slugs you define in `inputs/SCHEMA.md` (use kebab-case).
- `priority` must be one of: `critical`, `high`, `medium`, `low`.
- `sentiment` must be one of: `positive`, `negative`, `neutral`, `mixed`, `urgent`.
- `date` must be `YYYY-MM-DD` format.
- `customers` is an array of company names (not individual contact names).
- Filing location: `inputs/{YYYY}-{QN}/{type}/` — quarter is determined by `date` (Jan–Mar=Q1, Apr–Jun=Q2, Jul–Sep=Q3, Oct–Dec=Q4).

## Workflow commands (project-level)

All commands are invoked with `/command-name` and live in `.claude/commands/`.

**Setup**

- `/setup-pm-os` — Interactive first-time setup. Walks you through company identity, Atlassian/Confluence config, your first product folder, source-driven company/product context, optional strategy enrichment, and selected feature-context seeding.
- `/add-context` — Add or enrich one context doc (company, product, or feature level). Accepts URLs, Confluence pages, pasted text, files, or just conversation; proactively asks for missing high-value sources and reports sampled/failed/deferred pages.

**Project lifecycle**

- `/start-project <Project Name>` — Scaffold a new project workspace from templates.
- `/add-input` — File any input (feedback, transcript, meeting notes, Slack thread, etc.) into the correct location with structure and frontmatter.
- `/import-inputs` — Scan `inputs/` for files relevant to a project (using frontmatter grep) and add them to `input-references.md`.
- `/publish-to-confluence` — Publish a PRD to Confluence; stores the resulting `page_id` in frontmatter.
- `/update-on-confluence` — Update an existing Confluence page using the stored `page_id`.
- `/publish-to-jira` — Link a PRD to a Jira issue; stores `issue_key`/`issue_url` in frontmatter.

**Authoring / discovery**

- `/product-brainstorming` — Root-cause analysis, first principles, idea generation.
- `/solution-architect` — Define and evaluate solution approaches; produce solution design docs.
- `/prd-generator` — Generate lean, job-focused PRDs (asks 10-15 clarifying questions first).
- `/documentation-agent` — Convert PRDs into dev/design/QA docs.
- `/competitive-analysis` — Structured competitor research and capability comparison.

**Design / UX**

- `/ux-expert` — Usability heuristics, design review, interaction analysis.
- `/design-review` — Compare designs against PRDs for completeness. Uses Figma MCP if available; otherwise accepts pasted screenshots/exports.
- `/screen-diagrams` — ASCII wireframes of product screens (maintained in `context/products/`). Uses Figma MCP if available; otherwise accepts pasted screenshots/exports.
- `/wireframes` — ASCII wireframes and user flows for a specific project (writes `wireframes.md`).
- `/lovable-structured` — Generate structured Lovable prompts for prototyping.

### Skill pipeline

Typical project flow:

```
/setup-pm-os → /add-context → /start-project → /product-brainstorming → /solution-architect → /prd-generator
                                                                                                  ↓
                                              /publish-to-jira + /publish-to-confluence ← /documentation-agent
```

Side-quests: `/competitive-analysis`, `/ux-expert`, `/design-review`, `/wireframes`, `/screen-diagrams`, `/lovable-structured`.

## Frontmatter: Inputs vs Outputs

PRD frontmatter has two kinds of fields:

- **Inputs** default to values in `pm-os.config.yml`. Leave them empty in the PRD to use the config default. Fill them in PRD frontmatter only when you want a per-PRD override (rare).
- **Outputs** are filled automatically by publish commands. Don't fill these by hand.

The publish commands **never write input values back to PRD frontmatter** — if you don't set them, they stay empty for the lifetime of the PRD, and the config default applies on every publish.

### Confluence Frontmatter

Every PRD should include:

```yaml
---
confluence:
  page_id: ""        # Output: filled by /publish-to-confluence
  space_id: ""       # Input: default = pm-os.config.yml confluence.default_space_id
  parent_id: ""      # Input: default = pm-os.config.yml confluence.default_parent_id
---
```

When publishing, internal links `[Doc](./other.md)` are resolved to Confluence URLs if the target has a `page_id`.

### Jira Frontmatter

Every PRD should also include:

```yaml
---
jira:
  project_key: ""    # Input: default = pm-os.config.yml jira.default_project_key
  issue_key: ""      # Output: e.g. <YOUR-PROJECT>-123 — filled by /publish-to-jira
  issue_url: ""      # Output: full browse URL — filled by /publish-to-jira
  issue_type: ""     # Output: "Epic", "Task", or "Story"
  created_at: ""     # Output: ISO date when first linked
---
```

`/publish-to-jira` resolves `project_key` with three-tier precedence: PRD frontmatter (if set) → `pm-os.config.yml` `jira.default_project_key` → prompt the user.

## General Principles

- Always cite sources when referencing specific inputs.
- Ask clarifying questions when requirements are ambiguous.
- Suggest alternatives when you see potential issues.
- Keep outputs focused and concise.
