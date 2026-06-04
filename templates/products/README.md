# templates/products/

Blank product context template. Copied by `/setup-pm-os` (first product) and `/add-context` (new product) into `context/products/<slug>/`. The feature template is also copied when setup or `/add-context feature` seeds a selected feature file.

## What lives here

- `example-product/overview.md` — the canonical product overview template (Product Overview, Strategy/Positioning, Unique Approach pillars, Key Capabilities, Target Users tables, ICP, Product Principles, Key Metrics, Competitive Landscape, Differentiation).
- `example-product/example-feature.md` — the canonical feature-context template (Feature Overview, User Jobs, Key Workflows, Data Model, Edge Cases, Integrations, Related Features, Out of Scope).

## Editing rules

- **Don't edit content inside `example-product/`** unless you want to change the structure shipped to all future copies. The skills always copy from here, so any edit propagates to every new product folder created from this point forward.
- **Customizing the structure:** if your fork wants every product to have an extra section (say, a "Roadmap" block), edit `templates/products/example-product/overview.md` to add it. New products picked up after that change inherit the new structure.
- **Feature files:** keep `example-feature.md` generic. Setup and `/add-context feature` initialize identity fields after copying, then synthesize sourced sections from selected inputs.

## Why not in `context/products/`?

Earlier versions kept the example under `context/products/example-product/` and renamed it to the user's product slug on first setup — which deleted the only blank template the repo shipped with. Subsequent "new product" runs in `/add-context` would have had to copy from a sibling product folder, inheriting that sibling's filled content. Moving the template to `templates/` keeps it pristine for the lifetime of the fork and matches the convention used by `templates/projects/`, `templates/inputs/`, and `templates/lovable/`.
