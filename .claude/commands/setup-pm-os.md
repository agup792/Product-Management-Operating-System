You are guiding a new PM through onboarding their PM OS fork. Walk them through one step at a time. Don't dump all questions at once -- ask, get the answer, confirm, then move to the next step.

This command has two jobs:

1. Set up the repo spine: source-backed company identity, first product folder, and optional publishing config.
2. Use the company website first, then web search, volunteered sources, competitor/docs sources, and short user answers to seed as much useful company, product, strategy, and selected feature context as the PM wants.

Default to automation before manual questions. Ask the PM for factual company/product fields only after public sources are unavailable, thin, or conflicting. Keep setup single-product. Additional product folders belong in `/add-context product`. Keep MCP connectors optional; public web, pasted docs, files, and direct answers are enough to complete setup.

## Step 0: Welcome

Greet the user and briefly explain what this command does:

> "I'll walk you through setting up your PM OS fork. We'll start with your company website so I can infer company facts and product candidates before asking you to type anything manually. Then we'll confirm the company context, pick your first product from source-backed candidates, draft company/product context, optionally seed strategy or feature docs, and only then offer Atlassian/Confluence publishing defaults. You can skip almost any question and fill it later with `/add-context` or by editing `pm-os.config.yml`. Sound good?"

Wait for confirmation before proceeding.

## Step 1: Detect Existing State

Check whether this is a fresh fork or a re-run. The blank product template lives at `templates/products/example-product/` (not in `context/products/`).

Read these signals:

- `pm-os.config.yml` -- if `atlassian.domain` is non-empty, treat publishing config as already set.
- `context/company/about-company.md` -- if line 1 is the literal `# About [Your Company]`, this is a fresh company file. If the line has been replaced, the company section has already been customized.
- `context/products/` -- list children, filtering out `README.md`. **No remaining children -> fresh fork.** Any subfolder present (whether `myapp/` or a stray `example-product/`) -> already-set-up or partial setup.
- `templates/products/example-product/` -- must exist. If missing, halt with: "Blank product template not found at `templates/products/example-product/`. The repo appears corrupted. Restore from your most recent backup and try again."

If any "already set up" signal fires, ask:

> "Looks like you've run setup before. Re-running can propose context changes in `about-company.md`, a product `overview.md`, optional `strategy.md`, selected feature files, and optionally update `pm-os.config.yml`. Existing non-placeholder context is shown as a diff/merge before applying, and config values are rewritten only after you confirm. Continue? (y/N)"

Default to bailing if the user doesn't confirm. If they confirm, keep config rewrites separate from context synthesis diffs.

Re-run write safety:

- If the target slot is still a placeholder, write directly after the review step.
- If the target slot already has real content, show the proposed diff or merge and ask before applying.
- If user-confirmed setup answers later conflict with web/search evidence, preserve the setup answer as sourced content until the PM confirms a change.
- Apply this diff/merge rule to every target: `about-company.md`, product `overview.md`, optional `strategy.md`, selected feature files, and `pm-os.config.yml`.

## Step 2: Gather Website + Initial Sources First

Ask:

> "What's the public website for the company? Paste the homepage URL if you have it. If there is no public website, say `skip` and I'll fall back to direct questions."

If the PM provides a URL:

1. Normalize it to an absolute URL with scheme. If the user gives `example.com`, treat it as `https://example.com`.
2. Add it as source label `Homepage`.
3. Do not ask for company name, founded year, headquarters, employee count, product name, value prop, or tagline yet.

If the PM skips the URL:

- Say: "No problem. Without a website, I can still set up the repo from short answers and any docs/text you share."
- Continue with volunteered sources and fallback questions, but clearly mark manually provided facts as `Setup answer`.

Ask once for additional sources before crawling:

> "Any other sources I should use now? You can paste product pages, help docs, press/about pages, competitor names/sites, strategy docs, text, or file paths. You can also say `skip` and I'll start from the website."

Classify each volunteered item:

- URL -> fetch/crawl using the relevant rules below.
- Confluence URL or page ID -> use the Atlassian MCP if connected; if unavailable, report that and continue.
- Pasted text -> keep as source label `Pasted text`.
- File path -> read it and label as `File - <name>`.
- Competitor name without URL -> reserve it for Step 8 competitor handling.

## Step 3: Crawl + Search Before Asking Factual Questions

Build the initial source list from the website URL, volunteered sources, and any direct answers already provided. Track each source with the labels from `docs/command-contracts/context-synthesis.md`.

### 3a. Website crawl

If a website URL is available and web fetch tools are available, attempt up to 10 unique same-domain pages:

1. Homepage first.
2. Same-domain About, Company, Product, Products, Platform, Solutions, Customers, Pricing, Resources, and Docs/help links discovered from homepage navigation.
3. Within each navigation category, attempt links in homepage DOM/order-of-appearance.
4. Fallback standard paths until 10 unique pages have been attempted: `/about`, `/about-us`, `/company`, `/products`, `/product`, `/platform`, `/solutions`, `/customers`, `/pricing`.

Deduplicate by normalized URL. Report sampled, failed, and deferred pages. If a URL fails (404, auth-walled, robots, timeout), report it and continue.

If web fetch tools are unavailable, tell the PM: "Auto-fetch is unavailable in this environment, so I'll use pasted text/docs and direct answers for the context pass."

### 3b. Web search

If web search is available, search for:

- company legal/display name when the homepage is ambiguous
- founding year
- headquarters
- headcount
- funding
- customer count or notable public customer proof
- product names/product portfolio
- competitor names and official competitor URLs when needed

Prefer official company pages for current company/product claims. Use third-party search results for facts commonly maintained off-site, such as headcount, funding, and founding year, and label them like `(_Source: Web search - Crunchbase_)`.

If web search is unavailable, say so and continue with fetched pages, volunteered sources, and direct questions.

### 3c. Checkpoint report

Before asking the PM to confirm anything, show a compact report:

- Sampled: source labels and URLs/pages actually used.
- Failed: URL/page plus failure reason when known.
- Deferred: pages skipped because of caps, low relevance, auth, or follow-up scope.
- Found facts: company name, founding year, headquarters, employees/headcount, funding, customers, products/product candidates, and source labels.
- Gaps/conflicts: fields that remain missing or disagree across sources.

This report is the first resume checkpoint if the run times out.

### 3d. Resume checkpoints

After each major phase, print a compact checkpoint line so a later setup run can continue from real progress instead of starting over:

- After Step 3: detected sources, sampled pages, failed/deferred pages, found facts, and source gaps.
- After Step 5: confirmed company facts and the `about-company.md` fields written or left as placeholders.
- After Step 6: selected product display name, selected slug, candidate source, and whether the product folder was created.
- After Step 7: files written, sections filled, source labels used, and remaining company/product gaps.
- After Step 8: accepted nudges, skipped nudges, selected feature files, and remaining gap inventory.
- After Step 9: publishing defaults configured or explicitly skipped.

On a later invocation, inspect already-written files and these visible checkpoint facts from the conversation, then continue from the latest completed phase instead of starting over.

## Step 4: Confirm Company Identity + Key Facts

Use the gathered sources to propose company facts before writing `context/company/about-company.md`.

Show a short review table:

| Field | Proposed value | Source | Confidence |
|---|---|---|---|
| Company name | `<value or missing>` | `<source>` | high/medium/low/conflict |
| Founded | `<value or missing>` | `<source>` | high/medium/low/conflict |
| Headquarters | `<value or missing>` | `<source>` | high/medium/low/conflict |
| Employees | `<value or missing>` | `<source>` | high/medium/low/conflict |
| Funding | `<value or missing>` | `<source>` | high/medium/low/conflict |
| Customers | `<value or missing>` | `<source>` | high/medium/low/conflict |

Then ask one targeted confirmation:

> "I'll use these sourced company facts where confidence is high. Anything wrong or missing that you want to correct now? You can answer with corrections, or say `looks good`."

Rules:

- If company name is still missing, ask: "What's the company name for this PM OS?"
- If a high-value fact is missing, do not ask all missing fields individually. Ask only for blocking or high-confidence corrections. Leave unsupported fields as placeholders and include them in the gap inventory.
- If sources conflict, show the conflicting values with source labels and ask which one to keep.
- Treat PM corrections as `Setup answer`.
- Do not ask for a 1-3 paragraph company overview here. The synthesis pass can draft it from sources or ask a focused follow-up if sources are thin.

## Step 5: Fill `context/company/about-company.md` Identity Fields

Apply targeted string replacements against the actual template after Step 4 confirmation. The template uses specific bracketed strings, not a generic `[Placeholder]` marker -- do exact matches.

Replacement table:

| Line | Original | Replace with | If missing/skipped |
|---|---|---|---|
| 1 | `# About [Your Company]` | `# About <company-name>` | ask; company name is required |
| 13 (Founded row) | `[Year]` | `<year> (_Source: <source label>_)` | leave `[Year]` |
| 14 (Headquarters row) | `[City, Country]` | `<location> (_Source: <source label>_)` | leave `[City, Country]` |
| 16 (Employees row) | `[Approximate count]` | `<count> (_Source: <source label>_)` | leave `[Approximate count]` |
| 17 (Funding row) | `[Total raised or "Bootstrapped" / "Public"]` | `<funding> (_Source: <source label>_)` | leave placeholder |
| 18 (Customers row) | `[Approximate count or notable logos]` | `<customers> (_Source: <source label>_)` | leave placeholder |

Lines 15 (Offices: `[List locations]`) and any richer prose sections are filled later only when sourced by the synthesis contract.

Confirm: "Updated `context/company/about-company.md` with confirmed sourced identity fields. Next I'll use the website/source crawl to suggest your first product."

## Step 6: Select First Product From Source-Backed Candidates

Do not begin by asking "What's the first product?" if the website/source crawl found product candidates. Use source-backed discovery first.

### 6a. Candidate extraction

Extract product candidates from:

- homepage/product navigation
- product, platform, solution, and pricing pages
- product portfolio tables/sections
- docs/help navigation when it clearly maps to user-facing products
- PM-provided product URLs or names

Candidate criteria:

- A candidate is a distinct product, platform, module, or branded offering a PM might manage.
- Generic pages such as "Solutions", "Resources", "Company", "Customers", "Pricing", "Blog", "FAQ", and "Contact" are not product candidates unless the page content names a distinct user-facing offering.
- Prefer official source names over inferred names.

Rank up to 8 candidates:

1. PM-provided product names or product URLs.
2. Official product navigation order.
3. Product/platform pages with explicit product descriptions.
4. Source frequency across sampled pages.

For each candidate, show:

- display name
- suggested folder slug
- one-line source-backed description, if available
- source label/URL
- confidence

Ask:

> "Which product should be the first PM OS product folder? Pick one candidate, give me a different product name/URL, or say `skip` to create it later with `/add-context product`."

If no candidates were found, ask:

> "I couldn't find a clear product candidate from the sources. What's the first product you'll be managing in this PM OS? You can also skip and create it later with `/add-context product`."

### 6b. Slugify the selected product

Apply the canonical slug rules (used everywhere in the OS for product folder names):

1. Lowercase the entire string.
2. Replace each run of whitespace with a single hyphen.
3. Strip every character that isn't `[a-z0-9-]`.
4. Collapse runs of multiple hyphens to a single hyphen.
5. Trim leading/trailing hyphens.

Examples: `MyApp` -> `myapp`. `My Product!` -> `my-product`. `AI / ML Insights` -> `ai-ml-insights`.

If the selected product came from a source-backed candidate, show the slug and ask only if the slug is ambiguous or surprising:

> "I'll use `<slug>` as the folder name for `<Product Name>` (for example, `context/products/<slug>/`). OK?"

If the PM provides a different product name, confirm the slug the same way. Treat PM-provided names as `Setup answer`.

### 6c. Create the product folder

The canonical blank template lives at `templates/products/example-product/`. **Copy from there, never move.** This keeps the template pristine for future products created via `/add-context`.

1. **Verify the template exists:** if `templates/products/example-product/` is missing, bail with "Blank product template not found at `templates/products/example-product/`. The repo appears corrupted." Don't try to compensate.
2. **Check for slug collision:** if `context/products/<slug>/` already exists, ask: "A folder for `<slug>` already exists. Use a different name, or overwrite the existing folder?" Don't silently merge.
3. **Copy the template into place:**
   ```bash
   cp -r templates/products/example-product context/products/<slug>
   ```
   This copies both `overview.md` and `example-feature.md`. The template stays untouched.

### 6d. Fill `context/products/<slug>/overview.md` identity fields

Apply targeted replacements against the actual template (line numbers are approximate). Use exact placeholder matching where the template placeholder is shown; for the header blockquote, match by stable prefix or quote the full template line verbatim.

| Line | Original | Replace with |
|---|---|---|
| 1 | `# [Example Product Name]` | `# <Product Name>` (human-readable, not the slug) |
| 3 (header blockquote) | Line starting `> This is an example product context template.` | `> Product context for **<Product Name>**. Edit sections as your understanding evolves.` |

Do not ask for product value proposition or tagline before synthesis. First attempt to draft those fields from source-backed product pages. If they remain unsupported after synthesis, leave placeholders or ask one targeted follow-up in the gap loop.

Preserve product value prop/tagline setup answers as setup-sourced content if the PM volunteered them. If later synthesis finds conflicting or stronger website wording, propose a diff/merge instead of silently replacing the answer.

### 6e. Feature template note

The `cp -r` in Step 6c also copies `example-feature.md` into `context/products/<slug>/`. Tell the user: "The `example-feature.md` in your product folder is a per-feature template. You can copy and rename it for each feature you want to document, or we can seed selected feature files later in this setup flow."

## Step 7: Synthesize Company + Product Context

Before synthesizing, read and apply `docs/command-contracts/context-synthesis.md` with each target file, setup answers, gathered website/search sources, current file contents, and any loop answers.

Initial targets:

1. `context/company/about-company.md`
2. `context/products/<slug>/overview.md`

Synthesis boundaries:

- DRAFT company overview, mission, explicit category/tagline/value-proposition language, explicitly stated differentiators, product portfolio, product value prop, product tagline, and product capability context when sources support them.
- ASK or SKIP inferred positioning wedge, why-we-win claims, strategic interpretation, ICP, green flags, red flags, product principles, and internal metrics.
- Keep Key Facts source-tagged. If setup answers conflict with web/search values, surface the conflict and ask the PM which value to keep.
- Preserve existing non-placeholder prose unless the PM accepts a proposed diff.

Before saving synthesized prose, show the shared contract review output:

- Proposed updates grouped by section.
- Source labels used.
- Conflicts needing resolution.
- Questions to ask now.
- Gaps that will remain placeholders.

After saving accepted updates, show a compact filled-vs-empty status for both target files with sources.

## Step 8: Gap-Driven Source Loop

Use the gap inventory from the shared contract. Process nudges highest-value-first:

1. Product capabilities/help-docs gaps.
2. ICP/positioning ASK gaps.
3. Competitor/competitive-landscape gaps.
4. Strategy docs or guided strategy Q&A.
5. Funding/About/mission cleanup gaps.

Every setup nudge must include this out: "or skip and fill later with `/add-context`."

When the PM provides a new source mid-loop, ingest it, re-synthesize affected sections, and refresh the gap inventory.

### 8a. Product capabilities and help docs

If product capabilities or feature context remain thin, ask:

> "Share your product's help docs, manual, or docs-site URL and I'll sample the top capability pages, or skip and fill later with `/add-context`."

If a docs/help/manual URL is provided:

- Fetch the docs homepage/index plus up to 10 top-level capability pages.
- Prefer pages that appear in primary docs navigation and describe user-facing capabilities/workflows.
- Report sampled, failed, and deferred pages before synthesis.
- DRAFT product capability content from explicit docs content.

### 8b. ICP and positioning

Ask targeted questions seeded with site-implied content. Keep questions batched by file. Route judgment-heavy claims to ASK unless directly stated in a source.

### 8c. Competitors and competitive landscape

If competitor context is empty or thin, ask:

> "Paste your main competitors' websites, or just their names and I'll search for official sites. I can draft public competitor facts and ask you for your counter-positioning, or skip and fill later with `/add-context`."

On receiving competitor URLs or names:

- Process up to 5 competitors in provided order.
- For each competitor, fetch homepage plus up to 3 product/platform/solution pages.
- Report sampled, failed, and deferred pages.
- DRAFT competitor name, category, and public strengths/capabilities from competitor pages.
- ASK for "how we compare", weaknesses, our counter, our advantage, and positioning. Do not infer those from competitor pages alone.
- Competitor data may populate company competitors, product competitive landscape, and strategy competitive landscape, but show proposed rows before saving, especially when adding rows beyond existing placeholders.

### 8d. Optional strategy enrichment

After the ready-to-start threshold is met, ask:

> "You have enough to start projects. Want to add strategy context now, seed feature docs, configure publishing defaults, or finish setup?"

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

### 8e. Optional feature-context seeding

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

## Step 9: Optional Publishing Defaults

Publishing config is not part of the critical first-context path. Offer it after company/product context is usable or after the PM chooses to finish context setup.

Ask:

> "Optional: do you want to configure Atlassian/Jira/Confluence publishing defaults now? You can skip and publish commands will prompt later."

If the PM skips, do not write `pm-os.config.yml`; report that publishing defaults remain empty.

If the PM opts in, ask in sequence:

### 9a. Atlassian site host

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

### 9b. Default Jira project key

> "What's your default Jira project key? (e.g. `PROJ` -- press Enter to skip; publish commands will prompt until this is set.)"

Explanation if asked: "The prefix on Jira issue keys, e.g. `PROJ-123`. This will be the default project for all new PRDs. You can override per-PRD in PRD frontmatter (`jira.project_key`) if a specific PRD needs to land in a different project."

### 9c. Default Confluence space ID

> "What's your default Confluence space ID? (e.g. `123456` -- press Enter to skip.)"

Explanation if asked: "Open any page in your target Confluence space. The space ID is in the URL (`/spaces/<ID>/...`) or in Space settings -> Space details."

Allow skip -- note that `/publish-to-confluence` will prompt per-PRD until this is set.

### 9d. Optional default parent page

> "Optional: default Confluence parent page ID? (Press Enter to skip)"

Explanation if asked: "If set, new PRDs publish under this parent page in Confluence. If skipped, they publish at the top of the space. You can always override per-PRD."

### 9e. Write `pm-os.config.yml`

Edit the existing `pm-os.config.yml` (already in the repo with empty defaults). Replace the four empty string values with what Step 9 collected. Keep all comments intact. Leave skipped values as empty strings.

On re-run, apply config changes only after the Step 1 confirmation. Do not bundle config rewrites with context synthesis diffs.

If all config values are set, confirm: "Wrote `pm-os.config.yml`. Publishing commands will use these defaults; you can edit by hand any time."

If any config values were skipped, confirm: "Wrote `pm-os.config.yml`. Publishing commands will use the defaults that are set. Left unset: `<fields>`. Publishing commands will prompt until those values are filled; you can also edit `pm-os.config.yml` by hand any time."

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
      Optional; list configured publishing defaults if Step 9 ran, or "skipped".

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

Skipped strategy and skipped publishing defaults are normal completion states, not setup failures.

If the PM declined all nudges or chose `skip remaining`, still save accepted sourced content and print this checklist.

## Step 11: Wrap

Skill ends. Do not automatically chain to `/add-context` -- that's the user's call.
