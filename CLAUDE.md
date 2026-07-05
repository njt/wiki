# Wiki Schema

This is an LLM-maintained wiki inside Nat's Obsidian vault. The LLM writes and maintains all wiki pages. Nat sources material, asks questions, and reads the output.

## Structure

```
Wiki/
  CLAUDE.md       — this file (schema, conventions, workflows)
  index.md        — catalog of all topic pages with one-line summaries
  log.md          — append-only record of ingests, queries, and maintenance
  raw/<slug>.md      — VERBATIM source text (the archive; frontmatter + original body)
  summary/<slug>.md  — a concise précis of each source
  topic/<Title>.md   — analysis & synthesis; may draw on several sources
```

Three tiers per source, sharing one slug for raw/ and summary/:

- `raw/<slug>.md` — the **verbatim** fetched source (or a repo's README). The pipeline writes it; the model never rewrites it (only keeps or deletes it). Its `url:` frontmatter line is what duplicate detection reads.
- `summary/<slug>.md` — the précis.
- `topic/<Title>.md` — the analysis page. Cross-linked with `[[wikilinks]]`, which resolve by name regardless of folder.

`index.md` and `log.md` stay at the wiki root.

## Synthesis on ingest (compounding, not just filing)

After writing the new topic page, integrate the source into the existing wiki: find the 2–4 most-related existing `topic/` pages and make **minimal, additive** edits — weave in the new finding in a sentence or two plus a `[[backlink]]` to the new page. Never rewrite, reorder, or delete another page's content; only add. Skip when nothing is genuinely related. This keeps the wiki a compounding artifact, not a pile of disconnected pages.

## Page Format

Every wiki page starts with:

```markdown
# Page Title

One-paragraph summary of what this page covers.

---

(body)

---
*Sources: [[raw/filename]], [[raw/other]]*
*Last updated: YYYY-MM-DD*
```

Use `[[wikilinks]]` for cross-references. Use tags sparingly: `#person`, `#concept`, `#tool`, `#project`, `#comparison`.

## Workflows

### Ingest

When Nat drops a source into `raw/` or pastes content:

1. Read the source material fully
2. Create or update wiki pages for key entities, concepts, and takeaways
3. Update `index.md` with any new pages
4. Append an entry to `log.md`
5. Check existing pages for information that should be revised or cross-linked

### Query

When Nat asks a question against the wiki:

1. Read `index.md` to find relevant pages
2. Read those pages
3. Synthesize an answer with `[[wikilinks]]` to sources
4. If the answer is substantial and reusable, offer to save it as a new wiki page

### Lint

Periodic maintenance (do when asked, or suggest when the wiki grows):

1. Find orphan pages (not linked from any other page or index)
2. Flag stale claims (sources older than 6 months on fast-moving topics)
3. Check for contradictions between pages
4. Identify missing cross-references
5. Verify index.md is complete and accurate

## Relationship to the Rest of the Vault

This wiki is one section of a larger Obsidian vault. Other folders exist (`Agent Journals/`, `AI/`, `Band/`, `Ontempo/`, `Spain 2026/`). Wiki pages may link to content in those folders using `[[../folder/page]]` but the LLM only writes to `Wiki/`. Those other folders are Nat's space.

`Agent Journals/` are a natural source for wiki synthesis — session insights, thread summaries, and weekly digests can all feed the wiki.

## Principles

- Pages should be readable on a phone (short paragraphs, clear headers)
- Prefer updating an existing page over creating a new one
- Every factual claim should trace back to a source in `raw/` or a linked reference
- When in doubt, ask Nat rather than guessing
