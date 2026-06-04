You are helping a PM add or enrich a context file. Accept any mix of inputs the user has -- URLs, Confluence pages, pasted text, attached files, or just conversation -- and synthesize into the target file. Preserve the existing section structure; never rewrite a file wholesale.

Default write scope: `/add-context` writes only the selected `target_file`. If gathered sources reveal related feature files, strategy docs, product overviews, or adjacent context files worth updating, propose them as follow-up actions and ask before creating or writing any additional file.

## Step 1: Pick Level + Target File

Ask:

> "What level of context are you adding?
>   1. **Company** — `about-company.md` or `strategy.md`
>   2. **Product** — `overview.md` or a feature doc for a specific product
>   3. **Feature** — a feature-level doc (creates a new file if needed)"

Branch on the answer:

### Company

Ask which file: `about-company.md` or `strategy.md`. Read the chosen file as the starting point.

### Product

List `context/products/` subdirectories (excluding `README.md`) and ask which product. If the user says "new product":

1. Ask for the product name; slugify per the canonical rule (lowercase, whitespace → hyphens, strip non-`[a-z0-9-]`, collapse multi-hyphens, trim).
2. **Copy from the blank template** at `templates/products/example-product/`:
   ```bash
   cp -r templates/products/example-product context/products/<slug>
   ```
   Never copy from another live product folder — that would inherit content from a different product. If the template doesn't exist, halt with: "Blank product template not found at `templates/products/example-product/`. The repo appears corrupted."
3. Ask: "Add or update `overview.md`, or create a new feature doc?" -- branch into the product overview flow or the feature flow.

For an existing product, ask: "Add or update `overview.md`, or create / update a feature doc?"

### Feature

Ask which product folder (list `context/products/` subdirs, excluding `README.md`). Then:

1. Ask "What's the feature name?" — slugify per the canonical rule.
2. Check whether `context/products/<product>/<feature-slug>.md` already exists:
   - **Exists:** ask "Overwrite, append to, or create with a disambiguated name (e.g., `<slug>-v2`)?" If "overwrite" or "append", proceed to Step 2 with that file as the target. If "disambiguate", change `<feature-slug>` to the new value and continue to the create path.
   - **Doesn't exist (create path):**
     ```bash
     cp templates/products/example-product/example-feature.md \
        context/products/<product>/<feature-slug>.md
     ```
     Always copy from the canonical template — never from a sibling feature in the same product folder.
3. **Initialize identity fields** before the synthesis loop (same template-aware pattern `/setup-pm-os` uses for `overview.md`):

   | Line | Original | Replace with |
   |---|---|---|
   | 1 | `# [Example Feature Name]` | `# <Feature Name>` (human-readable, not the slug) |
   | 3 (header blockquote) | `> This is an example feature context template. Copy + rename ...` | `> Feature context for **<Feature Name>** in the **<Product Name>** product. Linked from project \`input-references.md\` files.` |
   | 9 | `**Core Value**: [One sentence — the outcome the feature delivers.]` | Ask the user "In one sentence, what outcome does `<Feature Name>` deliver?" -- if they answer, replace `[One sentence — the outcome the feature delivers.]` with the answer. If they skip, leave the placeholder for Step 3's synthesis to fill. |

   Sections below "Feature Overview" (User Jobs, Key Workflows, Key Concepts / Data Model, Edge Cases / Constraints, Integrations / Dependencies, Related Features, Out of Scope, Related Documentation) stay as placeholders — too rich for an interactive initial-fill. Step 3 synthesizes from user-provided inputs.

4. Save the initialized file, then continue to Step 2.

## Step 2: Inspect Gaps + Gather Input Sources

After the target file is selected and read, inspect its placeholders and current content. Use `docs/command-contracts/context-synthesis.md` to form a gap inventory before asking for sources.

Show a short source nudge tailored to the target:

> "Based on this target file, the highest-value missing sources are: `<list>`. Share any now, skip this source for now and leave the gap in the checklist, or continue with what you already provided."

Common target-specific nudges:

- `about-company.md`: company website/About page, funding links/status, competitor names/sites, customer/ICP notes.
- `strategy.md`: strategy docs/decks, market analysis, strategic bet/risk answers, competitor sites.
- Product `overview.md`: product pages, help docs/manual/docs site, competitor sites, target-user notes.
- Feature file: feature help-doc pages, product docs, release notes, customer feedback, internal specs.

Ask:

> "What inputs do you have for me to learn from? You can mix any of these:
>   - **URLs** -- paste any (company website, blog posts, press releases, public docs, help docs, competitor sites). I'll fetch them.
>   - **Confluence pages** -- paste Confluence URLs or page IDs. I'll fetch them via the Atlassian MCP if connected.
>   - **Pasted text** -- paste large chunks directly into the conversation.
>   - **Files** -- attach docs (PDF, markdown, images) to the conversation.
>   - **Just talk to me** -- I can ask you questions and write what you tell me.
>
> Paste / mention everything you've got, and tell me when you're done."

Loop until the user signals they're done. For each item, classify and fetch:

- **URL** → use `WebFetch` to retrieve and summarize the relevant content.
- **Confluence URL or page ID** → use `mcp__claude_ai_Atlassian__getConfluencePage` (or `searchConfluenceUsingCql` if only a partial URL is provided).
- **Pasted text** → keep in working memory.
- **File path** → use `Read` (handles markdown / text / PDF / images via Claude Code's built-in support).

If a URL fails (404, auth-walled, robots), report it and continue with the rest. Don't bail on the whole session for one bad URL.

### 2a. Crawl and reporting rules

Use bounded crawling when the source type warrants it:

- Company/product website: sample relevant same-domain homepage/About/Product/Platform/Solution pages, capped to the pages needed for the selected target.
- Competitor sources: process up to 5 competitors in provided order; for each, fetch homepage plus up to 3 product/platform/solution pages.
- Help docs/manual/docs site: fetch docs homepage/index plus up to 10 top-level capability pages.
- Confluence: fetch the user-provided page; use search only when the user provided a partial title or ID and MCP is connected.

Always report sampled, failed, and deferred pages before synthesis. If Atlassian MCP is unavailable, say so and continue with other sources; do not block the session.

If sources reveal adjacent files worth updating, list them as suggestions only. Examples: "These docs also expose feature candidates: `<list>`. Want to run `/add-context feature` after this file?" or "This strategy deck could also update `strategy.md`; want to handle that as a separate target?"

## Step 3: Synthesize Into the Target File

Before synthesizing, read and apply `docs/command-contracts/context-synthesis.md` with `target_file`, gathered `sources[]`, current file contents, and any user answers.

Use the shared contract for placeholder recognition, source tags, DRAFT / ASK / SKIP decisions, conflict handling, table row handling, gap inventory, and no-silent-overwrite behavior.

Competitor confidence rule: DRAFT competitor name/category/public strengths from competitor pages. ASK for our counter, our advantage, weaknesses, and positioning unless the PM explicitly provided those claims.

## Step 4: Show + Iterate

Use the shared contract's review output. Show the proposed updates section by section, including source labels, conflicts, ASK items, and gaps that will remain placeholders. Ask: "Look good, or want to revise this section?"

For any section the user wants to revise, switch to conversational mode (Socratic questions) and refine.

## Step 5: Save + Suggest Next

Save the file. Print:

> "Updated `<file path>`.
>
> Sections still empty: `<list>`.
>
> Want to keep filling, or are we done for now?"

If "done", suggest the next file the user should work on:

- After `about-company.md` → suggest `/add-context company` on `strategy.md`.
- After `strategy.md` → suggest `/add-context product` on the user's primary product overview.
- After a product overview → suggest `/add-context feature` for any feature the user is actively PMing.
- If sources exposed related files during this run, list them as opt-in follow-ups. Do not write them from the current run unless the user explicitly selected them as the target.
