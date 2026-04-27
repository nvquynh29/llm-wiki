# LLM Wiki

An agent skill for maintaining a personal knowledge base with coding agents, built on [Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern.

This repo can be installed as a skill for Claude Code, OpenCode, Cursor, and other supported agents.

## Install

Uses the [skills](https://www.npmjs.com/package/skills) CLI, which supports Claude Code, OpenCode, Cursor, Codex, and 41+ agents.

### Claude Code

#### Recommended: plugin
From within Claude Code, first add the marketplace:
```
/plugin marketplace add nvquynh29/llm-wiki
```

Then install the plugin:
```
/plugin install llm-wiki@llm-wiki
```

#### Via skills CLI

Install to a single project:
```bash
npx skills add nvquynh29/llm-wiki -a claude-code -s llm-wiki
```

Install globally:
```bash
npx skills add nvquynh29/llm-wiki -a claude-code -s llm-wiki -g
```

Install non-interactively:
```bash
npx skills add nvquynh29/llm-wiki -a claude-code -s llm-wiki -y
```

### OpenCode

#### Via skills CLI

Install to a single project:
```bash
npx skills add nvquynh29/llm-wiki -a opencode -s llm-wiki
```

Install globally:
```bash
npx skills add nvquynh29/llm-wiki -a opencode -s llm-wiki -g
```


### Cursor

This repository includes a committed Cursor project rule (`.cursor/rules/llm-wiki.mdc`) with `alwaysApply: true`. Open the folder in Cursor and the guidelines apply automatically.

#### Via skills CLI

```bash
npx skills add nvquynh29/llm-wiki -a cursor -s llm-wiki
```

### Per-project (any agent)

New project:
```bash
curl -o AGENTS.md https://raw.githubusercontent.com/nvquynh29/llm-wiki/main/AGENTS.md
```

Existing project (append):
```bash
echo "" >> AGENTS.md
curl https://raw.githubusercontent.com/nvquynh29/llm-wiki/main/AGENTS.md >> AGENTS.md
```

## Purpose

Build and maintain a persistent, compounding knowledge base where:
Build and maintain a persistent, compounding knowledge base where:
- Claude handles the bookkeeping (ingest, cross-referencing, linting)
- The human directs, curates, and asks questions
- Knowledge grows richer over time through compounding

## Folder Structure

```
raw/          -- source documents (immutable -- never modify these)
wiki/         -- markdown pages maintained by Claude
wiki/index.md -- table of contents for the entire wiki
wiki/log.md   -- append-only record of all operations
```

---

## How It Works
The following sections describe the full workflow and conventions that Claude follows when maintaining the wiki.

### The Full Workflow
#### 1. Initialization
Ensures the wiki can run safely and repeatedly:
1. Check for `raw/` and `wiki/`
2. Create missing files only: `wiki/index.md`, `wiki/log.md`
3. Never overwrite existing content

#### 2. Ingest
When you add a new source to `raw/` and ask Claude to ingest it:
1. Read the full source document
2. Discuss key takeaways before writing anything
3. Create a summary page in `wiki/` named after the source
4. Create or update concept pages for each major idea
5. Add wiki-links ([[page-name]]) to connect related pages
6. Update `wiki/index.md` with new pages and descriptions
7. Prepend an entry to `wiki/log.md` with the date, source name, and what changed

A single source may touch 10-15 wiki pages. That is normal.

#### 3. Query
When you ask a question:
1. Claude reads `wiki/index.md` to find relevant pages
2. Read those pages and synthesize an answer
3. Cite specific wiki pages in the response
4. If the answer is valuable, offer to save it as a new wiki page

Good answers should be filed back into the wiki so they compound over time.

#### 4. Lint
Regular health checks to keep the wiki consistent:
- Check for contradictions between pages
- Find orphan pages (no inbound links)
- Identify concepts that lack their own page
- Flag claims that may be outdated
- Verify all pages follow the correct format

### Page Format
Every wiki page follows this structure:

```markdown
# Page Title

**Summary**: One to two sentences describing this page.

**Sources**: List of raw source files this page draws from.

**Last updated**: Date of most recent update.

---

Main content with [[wiki-links]] to related concepts.

## Related pages

- [[related-concept-1]]
- [[related-concept-2]]
```

### Citation Rules

- Every factual claim references its source file
- Use the format (source: filename.pdf) after the claim
- If two sources disagree, note the contradiction explicitly
- If a claim has no source, mark it as needing verification

### How to Know It's Working

- Wiki pages are consistently interlinked
- New ingested sources update related pages
- Orphan pages are identified and resolved
- `wiki/index.md` and `wiki/log.md` always stay in sync with changes

### Customization

These guidelines are designed to be merged with project-specific instructions. Add them to your existing `CLAUDE.md` or create a new one.

## License

MIT
