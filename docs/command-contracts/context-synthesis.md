# Context Synthesis Contract

Shared prompt contract for commands that synthesize PM OS context files. Before synthesizing, read and apply this contract with:

- `target_file`
- current target file contents
- gathered `sources[]`
- any user answers
- command-specific scope constraints

Commands that use this contract must still own their surrounding flow: target selection, source gathering, crawl sequencing, user prompts, user review, saving, and next-step suggestions.

## Source Records

Track each source with a short stable label, type, and status:

- `Homepage` - the company's homepage.
- `About page` - about/company pages.
- `Product page` - product/platform/solution pages.
- `Docs page - <title>` - help-docs/manual/docs-site pages.
- `Competitor - <name> homepage` - competitor homepage.
- `Competitor - <name> product page` - competitor product/platform/solution page.
- `Web search - <source name>` - third-party search result source.
- `Setup answer` - a value the PM gave during `/setup-pm-os`.
- `Pasted text`, `File - <name>`, `Confluence - <page title>` - user-provided materials.

Use source labels in generated text. Do not invent source names.

## Placeholder Recognition

- Treat any bracketed string `[...]` as a placeholder slot, including descriptive placeholders like `[Year]`, `[City, Country]`, `[Your tagline]`, `[Pillar 1]`, and table rows where cells are bracketed.
- A section can be partially placeholder and partially real content. Fill only placeholder slots automatically.
- Real, non-placeholder content must never be silently replaced. Propose an addition or merge and show the diff before saving.
- Preserve the target file's existing section order, heading levels, table shape, and internal links.
- Never rewrite a file wholesale.

## Verdicts

Apply one verdict per section, paragraph, table row, or placeholder slot:

| Verdict | Use When | Action |
|---|---|---|
| DRAFT | A source explicitly states the content and no unresolved conflict exists. | Fill the slot and add a source tag. |
| ASK | Content is partial, inferred, judgment-heavy, or contradicted by another source. | Ask a targeted question seeded with what was found. |
| SKIP | No source supports the content, or the user skips the question. | Leave the placeholder and include it in the gap inventory. |

Never fabricate. A plausible inference is ASK, not DRAFT.

## Source Tags

Use one stable inline format:

- Paragraphs: `(_Source: Homepage_)`
- Table cells: `Value text (_Source: Web search - Crunchbase_)`
- Setup answers: `Value text (_Source: Setup answer_)`

For tables, put source tags in the existing value/description/notes cell. Do not add a new source column unless the target template already has one.

## Conflict Handling

If sources disagree, surface the conflict instead of silently choosing.

Examples:

- Founded: setup answer says `2019`, search result says `2020`.
- Employees: homepage says `500+`, search result says `312`.
- Funding: one source says bootstrapped, another names a funding round.

Show the conflicting values with source labels and ask the PM which to keep. If the PM already provided a value during setup, preserve it as `(_Source: Setup answer_)` until they confirm a change.

## Source Confidence Rules

- DRAFT only what the source explicitly states.
- ASK for plausible interpretations, positioning, "why we win", "our advantage", "our counter", weaknesses, ICP fit, strategy, or any claim that depends on internal judgment.
- Competitor pages can support competitor name, category, public capabilities, public strengths, and competitor-stated positioning. They cannot support our counter-positioning or advantage without PM input.
- Help-doc pages can support product capabilities, workflows, concepts, edge cases, integrations, and selected feature context when the page explicitly describes them.
- Strategy documents or PM answers can support strategy sections. Public web pages rarely support strategic bets, risks, rejected paths, or multi-year goals unless they state them directly.

## Tables

- Fill table rows one at a time.
- Do not delete unfilled rows; leave placeholders intact.
- If there are more sourced items than existing placeholder rows, propose adding rows and show them before saving.
- If a row has mixed confidence, draft the supported cells and route unsupported cells to ASK or SKIP.

## Review Output

Before saving, show:

- Proposed updates grouped by section.
- Source labels used.
- Conflicts needing resolution.
- Questions to ask now.
- Gaps that will remain placeholders.

After saving, report:

- Updated file path.
- Sections filled.
- Sections still empty.
- Suggested next command for the remaining gaps.

## Crawl Report Schema

When a command crawls websites, competitor pages, docs sites, or Confluence pages, show a compact report before synthesis:

- Sampled: source labels and URLs/pages actually used.
- Failed: URL/page plus failure reason when known.
- Deferred: pages skipped because of caps, low relevance, auth, or because they belong to a follow-up target.

The command controls crawl order and caps; this contract only defines the reporting shape.

## Gap Inventory

Emit a gap inventory after every synthesis pass. For each gap, include:

- Target file and section.
- Missing field or placeholder.
- Recommended action: ASK, NUDGE, RESEARCH, or SKIP.
- Best next source or exact question.
- Resume command, such as `/add-context company`, `/add-context product`, or `/add-context feature`.

## Command Scope

This contract is reusable policy. Each command supplies the allowed targets and write scope:

- `/setup-pm-os` can orchestrate multiple setup targets while still asking before touching existing non-placeholder content.
- `/add-context` writes only the selected `target_file` by default; related files are suggestions unless the user starts a separate target flow.

Use setup answers as first-class sources tagged `(_Source: Setup answer_)`. Use command-specific deferral wording: setup nudges point to `/add-context`; `/add-context` nudges leave skipped gaps in that run's checklist.
