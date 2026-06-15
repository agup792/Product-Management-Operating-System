# context/company/

Cross-product context about your company. Agents read these on most tasks.

## Files alongside this README

- **`about-company.md`** — Company overview: what you do, who you sell to (ICP), competitors, positioning, product portfolio. This is the single most important context file in the repo. Almost every agent reads it first.
- **`strategy.md`** — Strategy: market trends you're betting on, multi-year goals, strategic priorities, risks. Used when deciding *what's worth building*, not just *how to build it*.

Both files ship as templates with `[Placeholder]` markers. `/setup-pm-os` drafts `about-company.md` during onboarding and can optionally enrich `strategy.md` after the repo is ready to start projects. `/add-context company` can continue either file later with URLs, Confluence pages, competitor sources, pasted docs, files, or direct answers.

Fill the remaining placeholders before doing serious AI work. If `about-company.md` says "[Company name]" everywhere, the PRDs and brainstorming sessions will be correspondingly vague.

## Adding more

Most repos only need these two files. If you find yourself wanting a third (e.g., `pricing.md`, `personas.md`, `brand-voice.md`), feel free to add it — but ask first whether it really applies across all products. If it only applies to one product, it belongs under `context/products/<product>/`.
