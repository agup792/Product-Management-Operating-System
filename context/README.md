# context/

Persistent background that AI agents read before generating anything in this repo. If a fact about your company, products, or strategy lives anywhere in the workflow, it lives here.

## Subfolders

- **`company/`** — Cross-product context: who you are, who you sell to, what your strategic priorities are. Files here are read on almost every task.
- **`products/`** — Per-product context, one folder per product. Each product folder contains an `overview.md` plus optional feature-level files.

## Context-First Rule

Before generating any document, agents load context in this order (see `CLAUDE.md`):

1. `context/company/about-company.md` — Company, ICP, competitors, positioning.
2. `context/products/<relevant-product>/overview.md` — The product the project is about.
3. `context/company/strategy.md` — Strategy and market trends.
4. Feature-specific files (`context/products/<product>/<feature>.md`) — referenced from the project's `input-references.md`.

If an agent doesn't know which product folder is relevant, it asks the user.

## context/ vs inputs/

- **context/** is curated, stable, and authored by you. It's the "current truth" about your company and products.
- **`inputs/`** is raw, dated, and accumulates over time (customer calls, meeting notes, competitor screenshots). Inputs are evidence. Context is the synthesis you do over inputs.

When something you'd write in an input has stabilized enough to be reused across many projects — promote it into `context/`.

## Filling this in

The repo ships with placeholder files. Run `/setup-pm-os` first: it fills the repo spine, drafts sourced company/product context, and can optionally enrich strategy or seed selected feature files. Use `/add-context` afterward to enrich one file at a time; it can ask for missing high-value sources, crawl bounded website/docs/competitor pages, and report sampled/failed/deferred pages.

Replace remaining `[Placeholder]` markers in `company/about-company.md`, `company/strategy.md`, product overviews, and feature files before doing serious AI work -- otherwise agent outputs will be generic.
