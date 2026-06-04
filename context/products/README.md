# context/products/

Per-product context — one folder per product you ship.

## Folder layout

```
context/products/
├── my-app/
│   ├── overview.md        # required
│   ├── billing.md         # feature-level, optional
│   └── search.md          # feature-level, optional
└── another-product/
    └── overview.md
```

- **Folder names: kebab-case**, lowercase. Use `my-app/`, not `MyApp/` or `My App/`. Agents resolve product folders from project `input-references.md` links; a consistent naming convention keeps those links predictable.
- **Every product folder must have `overview.md`.** It's the entry point — capabilities, target users, architecture, key concepts.
- **Add feature-level docs (`<feature>.md`) as needed** when a single feature has enough surface area to warrant its own file. Don't create files speculatively; create them when an agent or you would benefit from having that context isolated.

## Getting started

Don't create product folders here by hand — let the skills do it so the blank template is copied consistently:

- **First product:** run `/setup-pm-os` (asks for product name, copies the template, drafts sourced `overview.md`, and can optionally seed selected feature files from PM-named features or qualifying help-doc pages).
- **Additional products:** run `/add-context` and pick "new product" (same flow, no first-time setup steps). `/add-context product` writes one selected target by default and proposes adjacent feature files as follow-ups unless you opt in.

The blank product template lives at `templates/products/example-product/`. Both skills copy from there into `context/products/<your-slug>/` so each new product starts from a pristine template, never inheriting another product's content.

If you want to peek at the template before running the skills, browse `templates/products/example-product/overview.md` and `example-feature.md`.

When setup offers feature candidates, it shows up to five source-labeled options and creates none unless you select them. Existing feature slugs require overwrite/append/disambiguate confirmation.

## Linking from projects

Project workspaces reference product context via `input-references.md`:

```markdown
## Feature Context
- [Product Overview](../../context/products/my-app/overview.md)
- [Billing](../../context/products/my-app/billing.md)
```

Skills like `/prd-generator` and `/competitive-analysis` resolve the relevant product folder from those links rather than guessing.
