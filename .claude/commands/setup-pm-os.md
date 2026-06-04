You are guiding a new PM through onboarding their PM OS fork. Walk them through one step at a time. Don't dump all questions at once -- ask, get the answer, confirm, then move to the next step.

This command has two jobs:

1. Set up the repo spine: company identity, publishing config, and first product folder.
2. Use setup answers, website/search sources, volunteered sources, competitor/docs sources, and short user answers to seed as much useful company, product, strategy, and selected feature context as the PM wants.

Keep setup single-product. Additional product folders belong in `/add-context product`. Keep MCP connectors optional; public web, pasted docs, files, and direct answers are enough to complete setup.

## Step 0: Welcome

Greet the user and briefly explain what this command does:

> "I'll walk you through setting up your PM OS fork. We'll cover: (1) your company identity, (2) Atlassian/Confluence publishing config, (3) your first product folder, and (4) a source-driven context pass. I'll use your website, web search, setup answers, and any sources you share to fill company and product context, then offer optional strategy and feature-context seeding. You can skip almost any question and fill it later with `/add-context` or by editing `pm-os.config.yml`; you can also skip any source request and finish once you have enough to start projects. Sound good?"

Wait for confirmation before proceeding.

## Step 1: Detect Existing State

Check whether this is a fresh fork or a re-run. The blank product template lives at `templates/products/example-product/` (not in `context/products/`).

Read these signals:

- `pm-os.config.yml` -- if `atlassian.domain` is non-empty, treat as "already set up".
- `context/company/about-company.md` -- if line 1 is the literal `# About [Your Company]`, this is a fresh company file. If the line has been replaced, the company section has already been customized.
- `context/products/` -- list children, filtering out `README.md`. **No remaining children -> fresh fork.** Any subfolder present (whether `myapp/` or a stray `example-product/`) -> already-set-up or partial setup.
- `templates/products/example-product/` -- must exist. If missing, halt with: "Blank product template not found at `templates/products/example-product/`. The repo appears corrupted. Restore from your most recent backup and try again."

If any "already set up" signal fires, ask:

> "Looks like you've run setup before. Re-running can update `pm-os.config.yml` and propose context changes in `about-company.md`, your product `overview.md`, optional `strategy.md`, and any selected feature files. Config values are rewritten only after you confirm, and existing non-placeholder context is shown as a diff/merge before applying. Continue? (y/N)"

Default to bailing if the user doesn't confirm. If they confirm, keep config rewrites separate from context synthesis diffs.

Re-run write safety:

- If the target slot is still a placeholder, write directly.
- If the target slot already has real content, show the proposed diff or merge and ask before applying.
- If setup answers later conflict with web/search evidence, preserve the setup answer as sourced content until the PM confirms a change.
- Apply this diff/merge rule to every target: `about-company.md`, product `overview.md`, optional `strategy.md`, and selected feature files.

## Step 2: Company Identity

Ask in sequence (one question at a time, wait for each answer):

1. **Required** -- "What's the name of the company you're building this PM OS for?"
2. **Optional** -- "Year founded? (Press Enter to skip)"
3. **Optional** -- "Where's the company headquartered? (e.g. `Seattle, WA, USA` -- press Enter to skip)"
4. **Optional** -- "Roughly how many employees? (Press Enter to skip)"
5. **Optional but recommended** -- "Public website URL? (Press Enter to skip -- if you share it, I'll use it now to draft company and product context.)"

Treat every answer as a source labeled `Setup answer`.

Don't ask for a 1-3 paragraph company overview here. The source-driven synthesis pass can draft it from website/search sources or ask a focused follow-up if sources are thin.

## Step 3: Fill `context/company/about-company.md` Identity Fields

Apply targeted string replacements against the actual template. The template uses specific bracketed strings, not a generic `[Placeholder]` marker -- do exact matches.

Replacement table:

| Line | Original | Replace with | If user skipped |
|---|---|---|---|
| 1 | `# About [Your Company]` | `# About <company-name>` | required -- block until provided |
| 13 (Founded row) | `[Year]` | `<year> (_Source: Setup answer_)` | leave `[Year]` |
| 14 (Headquarters row) | `[City, Country]` | `<location> (_Source: Setup answer_)` | leave `[City, Country]` |
| 16 (Employees row) | `[Approximate count]` | `<count> (_Source: Setup answer_)` | leave `[Approximate count]` |

Lines 15 (Offices: `[List locations]`), 17 (Funding: `[Total raised or "Bootstrapped" / "Public"]`), 18 (Customers: `[Approximate count or notable logos]`) are filled later only when sourced.

Confirm: "Updated `context/company/about-company.md` with setup-sourced identity fields. Next we'll configure publishing and create your first product, then use sources to draft richer context."

## Step 4: Atlassian + Publishing Setup

Ask in sequence:

### 4a. Atlassian site host

> "What's your Atlassian site host? (e.g. `yoursite.atlassian.net` -- press Enter to skip; publish commands will prompt until this is set.)"

Explanation if asked: "Look at the URL when you're logged into Jira or Confluence. Paste the full host including `.atlassian.net` -- it's the part between `https://` and the next `/` in URLs like `https://yoursite.atlassian.net/jira/...`."

If the PM presses Enter, leave `atlassian.domain` empty and skip normalization entirely.

Normalize the input before storing in config. Apply in order:

1. Strip leading `https://` or `http://`.
2. Strip any path / trailing slash (everything from the first `/` onward).
3. If the result doesn't end in `.atlassian.net`, ask: "Did you mean `<input>.atlassian.net`? (Y/n)" -- append the suffix on confirmation; loop on "no".
4. The result is the canonical form: `<slug>.atlassian.net`.

Examples:

- `https://yoursite.atlassian.net/jira/your-work` -> `yoursite.atlassian.net`
- `yoursite.atlassian.net/` -> `yoursite.atlassian.net`
- `yoursite` -> confirm -> `yoursite.atlassian.net`

### 4b. Default Jira project key

> "What's your default Jira project key? (e.g. `PROJ` -- press Enter to skip; publish commands will prompt until this is set.)"

Explanation if asked: "The prefix on Jira issue keys, e.g. `PROJ-123`. This will be the default project for all new PRDs. You can override per-PRD in PRD frontmatter (`jira.project_key`) if a specific PRD needs to land in a different project."

### 4c. Default Confluence space ID

> "What's your default Confluence space ID? (e.g. `123456`)"

Explanation if asked: "Open any page in your target Confluence space. The space ID is in the URL (`/spaces/<ID>/...`) or in Space settings -> Space details."

Allow skip -- note that `/publish-to-confluence` will prompt per-PRD until this is set.

### 4d. Optional default parent page

> "Optional: default Confluence parent page ID? (Press Enter to skip)"

Explanation if asked: "If set, new PRDs publish under this parent page in Confluence. If skipped, they publish at the top of the space. You can always override per-PRD."

## Step 5: Write `pm-os.config.yml`

Edit the existing `pm-os.config.yml` (already in the repo with empty defaults). Replace the four empty string values with what Step 4 collected. Keep all comments intact. Leave skipped values as empty strings.

On re-run, apply config changes only after the Step 1 confirmation. Do not bundle config rewrites with context synthesis diffs.

If all config values are set, confirm: "Wrote `pm-os.config.yml`. Publishing commands will use these defaults; you can edit by hand any time."

If any config values were skipped, confirm: "Wrote `pm-os.config.yml`. Publishing commands will use the defaults that are set. Left unset: `<fields>`. Publishing commands will prompt until those values are filled; you can also edit `pm-os.config.yml` by hand any time."

## Step 6: First Product

Ask: "What's the first product you'll be managing in this PM OS?"

### 6a. Slugify the response

Apply the canonical slug rules (used everywhere in the OS for product folder names):

1. Lowercase the entire string.
2. Replace each run of whitespace with a single hyphen.
3. Strip every character that isn't `[a-z0-9-]`.
4. Collapse runs of multiple hyphens to a single hyphen.
5. Trim leading/trailing hyphens.

Examples: `MyApp` -> `myapp`. `My Product!` -> `my-product`. `AI / ML Insights` -> `ai-ml-insights`.

Confirm: "I'll use `<slug>` as the folder name (e.g., `context/products/<slug>/`). OK?"

### 6b. Create the product folder

The canonical blank template lives at `templates/products/example-product/`. **Copy from there, never move.** This keeps the template pristine for future products created via `/add-context`.

1. **Verify the template exists:** if `templates/products/example-product/` is missing, bail with "Blank product template not found at `templates/products/example-product/`. The repo appears corrupted." Don't try to compensate.
2. **Check for slug collision:** if `context/products/<slug>/` already exists, ask: "A folder for `<slug>` already exists. Use a different name, or overwrite the existing folder?" Don't silently merge.
3. **Copy the template into place:**
   ```bash
   cp -r templates/products/example-product context/products/<slug>
   ```
   This copies both `overview.md` and `example-feature.md`. The template stays untouched.

### 6c. Fill `context/products/<slug>/overview.md` identity fields

Apply targeted replacements against the actual template (line numbers are approximate). Use exact placeholder matching where the template placeholder is shown; for the header blockquote, match by stable prefix or quote the full template line verbatim.

| Line | Original | Replace with |
|---|---|---|
| 1 | `# [Example Product Name]` | `# <Product Name>` (human-readable, not the slug) |
| 3 (header blockquote) | Line starting `> This is an example product context template.` | `> Product context for **<Product Name>**. Edit sections as your understanding evolves.` |
| 9 | `**Core Value Proposition**: [One sentence — the outcome customers get.]` | Ask the user "In one sentence, what outcome do customers get from `<Product Name>`? (Press Enter to skip)" If they answer, replace `[One sentence — the outcome customers get.]` with `<answer> (_Source: Setup answer_)`. If they skip, leave the placeholder untouched for Step 8 synthesis or `/add-context`. |
| 11 | `**Tagline**: *[Your tagline]*` | Ask the user "One-line tagline for `<Product Name>`? (Press Enter to skip)" If they answer, replace `[Your tagline]` with the answer. Add `(_Source: Setup answer_)` after the italic tagline. If they skip, leave the placeholder untouched for Step 8 synthesis or `/add-context`. |

Preserve product value prop/tagline setup answers as setup-sourced content. If later synthesis finds conflicting or stronger website wording, propose a diff/merge instead of silently replacing the answer.

### 6d. Feature template note

The `cp -r` in Step 6b also copies `example-feature.md` into `context/products/<slug>/`. Tell the user: "The `example-feature.md` in your product folder is a per-feature template. You can copy and rename it for each feature you want to document, or we can seed selected feature files later in this setup flow."

## Step 7: Gather Initial Sources

Build a source list from:

- Setup answers from Steps 2 and 6.
- The website URL, if provided.
- Any additional links, pasted text, or files the PM volunteers now.

Ask:

> "Any sources I should use for company, product, strategy, competitor, or feature context? You can paste links, competitor names/sites, docs/help/manual URLs, strategy docs, text, or file paths now. You can also say `skip` and I'll work from the website and direct questions."

### 7a. Website crawl

If a website URL is available and web fetch tools are available, attempt up to 8 unique same-domain pages:

1. Homepage first.
2. Same-domain About, Product, Platform, Solution, and Pricing links discovered from homepage navigation.
3. Within each navigation category, attempt links in homepage DOM/order-of-appearance.
4. Fallback standard paths until 8 unique pages have been attempted: `/about`, `/about-us`, `/company`, `/products`, `/product`, `/platform`, `/solutions`.

Deduplicate by normalized URL. Report sampled, failed, and deferred pages. If a URL fails (404, auth-walled, robots, timeout), report it and continue.

If web fetch tools are unavailable, tell the PM: "Auto-fetch is unavailable in this environment, so I'll use pasted text/docs and direct answers for the context pass."

### 7b. Web search

If web search is available, search for:

- founding year
- headquarters
- headcount
- funding
- competitor names and official competitor URLs when needed

Use source tags like `(_Source: Web search - Crunchbase_)`.

If web search is unavailable, say so and continue with setup answers, fetched pages, pasted sources, and direct questions.

## Step 8: Synthesize Company + Product Context

Before synthesizing, read and apply `docs/command-contracts/context-synthesis.md` with each target file, setup answers, gathered website/search sources, current file contents, and any loop answers.

Initial targets:

1. `context/company/about-company.md`
2. `context/products/<slug>/overview.md`

Synthesis boundaries:

- DRAFT company overview, mission, explicit category/tagline/value-proposition language, explicitly stated differentiators, product portfolio, product value prop, and product capability context when sources support them.
- ASK or SKIP inferred positioning wedge, why-we-win claims, strategic interpretation, ICP, green flags, and red flags.
- Keep Key Facts source-tagged. If setup answers conflict with web/search values, surface the conflict and ask the PM which value to keep.
- Preserve existing non-placeholder prose unless the PM accepts a proposed diff.

After the first synthesis pass, show a compact filled-vs-empty status for both target files with sources.

## Step 9: Gap-Driven Source Loop

Use the gap inventory from the shared contract. Process nudges highest-value-first:

1. Product capabilities/help-docs gaps.
2. ICP/positioning ASK gaps.
3. Competitor/competitive-landscape gaps.
4. Strategy docs or guided strategy Q&A.
5. Funding/About/mission cleanup gaps.

Every setup nudge must include this out: "or skip and fill later with `/add-context`."

When the PM provides a new source mid-loop, ingest it, re-synthesize affected sections, and refresh the gap inventory.

### 9a. Product capabilities and help docs

If product capabilities or feature context remain thin, ask:

> "Share your product's help docs, manual, or docs-site URL and I'll sample the top capability pages, or skip and fill later with `/add-context`."

If a docs/help/manual URL is provided:

- Fetch the docs homepage/index plus up to 10 top-level capability pages.
- Prefer pages that appear in primary docs navigation and describe user-facing capabilities/workflows.
- Report sampled, failed, and deferred pages before synthesis.
- DRAFT product capability content from explicit docs content.

### 9b. ICP and positioning

Ask targeted questions seeded with site-implied content. Keep questions batched by file. Route judgment-heavy claims to ASK unless directly stated in a source.

### 9c. Competitors and competitive landscape

If competitor context is empty or thin, ask:

> "Paste your main competitors' websites, or just their names and I'll search for official sites. I can draft public competitor facts and ask you for your counter-positioning, or skip and fill later with `/add-context`."

On receiving competitor URLs or names:

- Process up to 5 competitors in provided order.
- For each competitor, fetch homepage plus up to 3 product/platform/solution pages.
- Report sampled, failed, and deferred pages.
- DRAFT competitor name, category, and public strengths/capabilities from competitor pages.
- ASK for "how we compare", weaknesses, our counter, our advantage, and positioning. Do not infer those from competitor pages alone.
- Competitor data may populate company competitors, product competitive landscape, and strategy competitive landscape, but show proposed rows before saving, especially when adding rows beyond existing placeholders.

### 9d. Optional strategy enrichment

After the ready-to-start threshold is met, ask:

> "You have enough to start projects. Want to add strategy context now, seed feature docs, or finish setup?"

Ready-to-start threshold:

- company overview filled
- ICP filled
- product value prop filled
- product key capabilities filled

If the PM chooses strategy:

- Use provided strategy docs first.
- If no strategy docs exist, ask only:
  1. "What's the key strategic bet you're making?"
  2. "What are the top 1-2 risks?"
  3. "Any short mitigation for those risks?"
- Leave unsupported strategy sections as placeholders.

If the PM skips strategy, treat setup as complete-ready, leave placeholders, and list `/add-context company` as the resume command.

### 9e. Optional feature-context seeding

Offer feature-context seeding after product overview/help-doc synthesis, or when the PM names features.

Candidate criteria:

- The PM explicitly named the feature, or the docs page title/nav item names a product capability.
- The source content describes a distinct user-facing workflow or capability.
- Generic docs pages such as "Getting Started", "Admin", or "FAQ" are not candidates unless they also meet the distinct-capability test.

If more than 5 candidates exist, rank:

1. PM-named features.
2. Docs navigation order.
3. Source frequency across sampled pages.

Show up to 5 candidates with source labels/pages and ask which ones to seed. Default to none unless selected.

For each selected feature:

1. Slugify using the canonical product/feature slug rules.
2. If `context/products/<slug>/<feature-slug>.md` exists, ask whether to overwrite, append, or create a disambiguated slug; never silently replace it.
3. If the file doesn't exist, copy `templates/products/example-product/example-feature.md` into place.
4. Initialize identity fields using the same template-aware flow as `/add-context feature`.
5. Run the shared synthesis contract on the selected feature file with gathered sources.
6. Propose updating the product `overview.md` Related Documentation section with links to the new feature files. If the section still contains the template `example-feature.md` link, propose replacing it; otherwise propose appending links. Do not edit links without confirmation.

## Step 10: Final Checklist

Print a concise final checklist with the actual slug and actual remaining gaps:

```
Setup complete. You're ready to start projects when company overview, ICP, product value prop, and key capabilities are filled.

Ready to start projects:
  - Yes/No, with the missing threshold item if No.

Filled by /setup-pm-os:
  - context/company/about-company.md
      List filled sections and source labels.
  - context/products/<slug>/overview.md
      List filled sections and source labels.
  - context/company/strategy.md
      Optional; list filled sections if strategy enrichment ran.
  - context/products/<slug>/<feature-slug>.md
      Optional; list any selected feature files.
  - pm-os.config.yml
      Atlassian domain, default Jira project key, default Confluence space ID
      (and optional default parent page).

Optional context still available:
  - List skipped strategy sections, feature candidates not selected, competitor judgments needing PM input, and any remaining placeholders.
  - For each gap, include the resume command:
      /add-context company
      /add-context product
      /add-context feature

Suggested next steps:
  1. /start-project <Project Name>
  2. /add-context company    # optional strategy and deeper company context
  3. /add-context product    # optional deeper product/competitive detail
  4. /add-context feature    # optional feature-level detail
```

Skipped strategy is a normal completion state, not a setup failure.

If the PM declined all nudges or chose `skip remaining`, still save accepted sourced content and print this checklist.

## Step 11: Wrap

Skill ends. Do not automatically chain to `/add-context` -- that's the user's call.
