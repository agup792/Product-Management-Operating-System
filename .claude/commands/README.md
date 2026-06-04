# .claude/commands/

Project-level Claude Code skills. Each `.md` file in this folder defines one slash-command. Invoke them inside Claude Code with `/<command-name>` (e.g., `/start-project`).

Some commands also depend on shared prompt contracts under `docs/command-contracts/`. For example, `/setup-pm-os` and `/add-context` both read and apply `docs/command-contracts/context-synthesis.md` before synthesizing context. Shared contracts live in `docs/`, not in this slash-command folder.

## Skills shipped in this template

This template ships with **18 project-level skills**, organized into four groups:

### Setup

- `/setup-pm-os` — Interactive first-time setup. Walks you through company identity, Atlassian/Confluence config, your first product folder, source-driven company/product context, optional strategy enrichment, and selected feature-context seeding.
- `/add-context` — Add or enrich one context doc (company, product, or feature level). Accepts URLs, Confluence pages, pasted text, files, or just conversation; proactively asks for missing high-value sources and reports sampled/failed/deferred pages.

### Project lifecycle

- `/start-project <Project Name>` — Scaffold a new project workspace from templates.
- `/add-input` — File any input (feedback, transcript, meeting notes, etc.) into `inputs/` with proper frontmatter.
- `/import-inputs` — Scan `inputs/` for files relevant to a project and add them to `input-references.md`.
- `/publish-to-jira` — Link a PRD to a Jira issue. Uses `pm-os.config.yml` defaults; PRD frontmatter overrides per-PRD.
- `/publish-to-confluence` — Publish a PRD to Confluence. Uses `pm-os.config.yml` defaults; PRD frontmatter overrides per-PRD.
- `/update-on-confluence` — Update an existing Confluence page using the stored `page_id`.

### Authoring / discovery

- `/product-brainstorming` — Root-cause analysis, first principles, idea generation.
- `/solution-architect` — Define and evaluate solution approaches; produce solution design docs.
- `/prd-generator` — Generate lean, job-focused PRDs (asks 10-15 clarifying questions first).
- `/documentation-agent` — Convert PRDs into dev/design/QA docs.
- `/competitive-analysis` — Structured competitor research and capability comparison.

### Design / UX

- `/ux-expert` — Usability heuristics, design review, interaction analysis.
- `/design-review` — Compare designs against PRDs for completeness.
- `/screen-diagrams` — ASCII wireframes of product screens, maintained under `context/products/<product>/screen-diagrams.md`.
- `/wireframes` — ASCII wireframes and user flows for a specific project (writes `wireframes.md`).
- `/lovable-structured` — Generate structured Lovable prompts for prototyping.

## Note on Figma-dependent skills

`/design-review` and `/screen-diagrams` use the Figma MCP server if it's installed in your Claude Code environment. If Figma MCP isn't available, both skills also work from pasted screenshots or exported images — no skill changes needed.

## Customizing skills

Skills are plain markdown files. Open any `.md` here, edit the instructions, save. The next time you invoke the skill, Claude Code reads the updated version.

A few conventions if you fork or extend:

- The filename (minus `.md`) is the slash-command name.
- Shared command contracts belong under `docs/command-contracts/`; do not put non-command helper docs in this folder.
- Don't create a `.claude/README.md` — the only other things in `.claude/` are `settings.json` (small, self-evident) and `settings.local.json` (gitignored). This commands folder is where forkers spend their time.
- If you remove a skill, also remove references to it from `CLAUDE.md`, the top-level `README.md`, and any other skill that mentions it — otherwise agents will tell users to invoke a command that 404s.
