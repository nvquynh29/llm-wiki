---
name: llm-wiki
description: "Personal knowledge base maintained by Claude Code. Use for building or maintaining a LLM-powered knowledge base. Triggers include ingesting sources into a wiki, querying wiki knowledge, linting wiki quality, 'add to wiki', 'what do I know about', or any mention of 'LLM wiki' or 'My wiki'."
argument-hint: "init <name> | ingest <path|url> | compile [<path>] | query <question> | lint | remove <name>"
license: MIT
---

# LLM Wiki


Build and maintain a persistent, compounding knowledge base using LLMs.


## File Structure


```
raw/          -- source documents (immutable -- never modify these)
wiki/         -- markdown pages maintained by Claude
wiki/index.md -- table of contents for the entire wiki
wiki/log.md   -- append-only record of all operations
```


## Global Rules


- **File name format**: Kebab case, example: `here-is-a-sample-file-name`
- **Logging**: When updating `wiki/log.md`, always run this CLI command `date +"%Y-%m-%d %H:%M:%S"` to get the exact timestamp for log entries.
- **Atomic Operations**: All ingestion, queries, and maintenance tasks must be complete and self-contained.
- **Never modify anything in the `raw/` folder**
- **Always update `wiki/index.md` and `wiki/log.md` after changes**
- **Keep page names lowercase with hyphens (e.g. `machine-learning.md`)**
- **Write in clear, plain language**
- **When uncertain about how to categorize something, ask the user**


---


## 1. Initialization


Goal: Ensure the wiki can run safely and repeatedly without overwriting existing work.


### Steps


1. Check for `raw/` and `wiki/`.
2. Create missing items only:
   - `raw/`
   - `wiki/`
   - `wiki/index.md` (if missing)
   - `wiki/log.md` (if missing)
3. Never overwrite existing content.
4. If Query/Lint is requested but wiki structure is missing, respond: `Run an ingest first to initialize the wiki.`


### Required Output


- A valid workspace with immutable source area (`raw/`) and compiled knowledge area (`wiki/`).


---


## 2. Ingest Workflow


Goal: Convert one source into compounding wiki knowledge, not just a one-off summary.


### A. Pre-Write Review (required)


1. Read the full source.
2. Extract key takeaways, entities, and potential contradictions.
3. Discuss key takeaways with the user before writing wiki updates.


### B. Compile into Wiki (wiki/)


1. Decide action per concept:
   - **Merge** if concept already exists.
   - **Create new page** if concept is new.
   - **Cross-link** if concept spans multiple topics.
2. For each material claim:
   - Add source attribution.
   - If contradicted by newer or stronger evidence, mark as contradicted or superseded.
3. Apply cascade updates to related pages that are materially affected.
4. Refresh each touched page's `Last updated` field.


### C. Finalize


1. Update `wiki/index.md` for all created and updated pages.
2. Run `date +"%Y-%m-%d %H:%M:%S"` and prepend to `wiki/log.md`:
   - `## [<timestamp>] ingest | <primary article title>`
   - Optional `- Updated: <related page>` lines for cascade updates.
3. A single source may touch 10-15 wiki pages. That is normal.


### Page Format


Every wiki page should follow this structure:


```markdown
# Page Title


**Summary**: One to two sentences describing this page.


**Sources**: List of raw source files this page draws from. Use hyperlinks to the files in the `raw/` folder.


**Last updated**: Date of most recent update.


---


Main content goes here. Use clear headings and short paragraphs.


Link to related concepts using [[wiki-links]] throughout the text.


## Related pages


- [[related-concept-1]]
- [[related-concept-2]]
```


### Citation Rules


- Every factual claim should reference its source file
- Use the format (source: filename.pdf) after the claim
- If two sources disagree, note the contradiction explicitly
- If a claim has no source, mark it as needing verification


### Required Output


- Wiki pages created/updated with citations and cross-links.
- `wiki/index.md` and `wiki/log.md` updated in the same operation.


---


## 3. Query Workflow


Goal: Answer from compiled wiki knowledge and optionally crystallize valuable answers back into the wiki.


### A. Retrieve


1. Read `wiki/index.md` first to locate candidate pages.
2. Read the most relevant pages fully.
3. If coverage is weak, expand to related linked pages before answering.


### B. Synthesize


1. Answer using wiki content first (not model priors).
2. Cite supporting wiki pages with project-root-relative links.
3. If claims conflict, present both sides and indicate which appears stronger based on recency and support.


### C. Crystallize (only when user asks to save/archive)


1. Create a new wiki page for the answer (do not merge into source pages).
2. Add citations to wiki pages used.
3. Update `wiki/index.md` with an `[Archived]` summary marker.
4. Run `date +"%Y-%m-%d %H:%M:%S"` and prepend to `wiki/log.md`:
   - `## [<timestamp>] query | Archived: <page title>`


### Required Output


- Default: response with citations only (no file writes).
- Archive mode: new page + index update + log update.


---


## 4. Lint Workflow


Regular health checks to keep the wiki consistent and high-quality.


### Deterministic (Auto-fix)


- **Index consistency**: Ensure `wiki/index.md` matches the actual file structure.
  - *Do:* Add entries for new files; mark removed files as `[MISSING]`.
- **Internal links**: Validate all wiki-internal markdown links.
  - *Do:* Search `wiki/` for valid targets if a link is broken; fix path if found.
- **Raw references**: Validate links to `raw/` files.
  - *Do:* Ensure `raw/` files exist for every link in an article's `Sources` metadata.


### Heuristic (Report Only)


- **Factual contradictions**: Cross-compare claims. Report if two pages disagree.
- **Stale/Superseded claims**: Check if newer sources (or later pages) imply older claims are outdated.
- **Orphaned pages**: Identify files with no inbound links from any other wiki page.
