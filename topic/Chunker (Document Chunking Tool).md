# Chunker (Document Chunking Tool)

A Python CLI (~2,700 LOC) that transforms documents into navigable knowledge trees — self-sufficient chunks and multi-level summaries linked into a hierarchy. Uses local Ollama LLMs to find semantic boundaries instead of fixed-size splitting. Outputs Obsidian-compatible markdown and JSON for progressive disclosure: start at root summaries, drill down to specific passages without ever loading the full document.

---

## Architecture

The system follows a **pipeline pattern** with two distinct phases orchestrated through a single sequential loop.

**Entry point**: `chunker.cli:main` → `Command.run()` or `Command.resume()` dispatches to `Pipeline.run_document()` ([[pipeline.py:72|pipeline.py:72]]).

### Phase 1: Semantic Chunking (cursor-driven)

A `CursorWindow` advances through the text one split unit (sentence, paragraph, or word) at a time. After reaching `min_chunk_tokens`, an LLM **completeness check** asks whether the window ends at a natural topic boundary. The loop:

1. Expand the window by one split unit
2. Ask the LLM: "does this window end at a complete thought?"
3. If complete: validate the `boundary_phrase` by exact substring match in remaining text. Retry once if not found, then fallback to sentence boundary.
4. If not complete: expand and repeat (up to `max_expansion_attempts`)
5. Safety valves: `max_chunk_tokens` or exhausted attempts → force-split at last sentence boundary, mark `forced_split=true`

Two parallel extractors handle different input modes:
- **`ChunkExtractor`** ([[nodes/chunking.py:21|nodes/chunking.py:21]]) — text documents, char-level cursor
- **`PageChunkExtractor`** ([[nodes/page_chunking.py:94|nodes/page_chunking.py:94]]) — PDF documents, page-level cursor with text-only completeness checks (no vision tokens wasted on boundaries)

PDF input uses PyMuPDF to render pages to PNG at configurable DPI. The vision model is only invoked during the rewrite step (not during chunking), saving tokens.

### Phase 2: Bottom-Up Hierarchical Aggregation

`AggregationSweeper.sweep()` ([[nodes/aggregation.py:87|nodes/aggregation.py:87]]) is called after every chunk. It checks two thresholds:
- **Token threshold**: cumulative token count of pending summaries > `summary_aggregation_threshold`
- **Count threshold**: number of pending summaries > `summary_count_threshold`

When triggered, it:
1. Asks the LLM to group pending summaries into **contiguous, thematically coherent clusters**
2. Validates hard constraints (contiguity, `min_group_size` ≥ 2) and soft constraints (`max_group_size` as a hint)
3. Falls back to even-sized grouping after 2 consecutive hard failures
4. For each group: synthesizes a chunk-sized context + summary → creates a `SummaryBlock`
5. Promotes blocks to the next level's pending summaries, recurses upward

The recursion stops when only one block remains (root) or pending summaries fall below both thresholds.

### Context Injection

`ContextBuilder` ([[context.py:18|context.py:18]]) assembles metadata for every LLM call in strict priority order:
1. **Immediate predecessor's context** — resolves local references and pronouns
2. **Latest summary from each higher level** — topic framing from the growing hierarchy
3. **Earlier chunks walking backwards** — broader document context

A hard `context_budget_tokens` cap is enforced with **all-or-nothing insertion**: items that would exceed the budget are skipped entirely (no sentence fragments). Later chunks get richer context as the hierarchy fills in.

### State and Checkpointing

`PipelineState` ([[state.py:9|state.py:9]]) holds all mutable state as a single dataclass with JSON serialization. `Checkpointer` ([[checkpoint.py:10|checkpoint.py:10]]) uses **mkstemp + atomic rename** for crash-safe writes — no partial checkpoints can exist. The pipeline checkpoints after every chunk and block, enabling resume without reprocessing.

### Output

Two formats from [[nodes/output.py|nodes/output.py]]:
- **`MarkdownRenderer`**: writes `index.md` + `content/L0/`, `content/L1/`, etc. with wiki-links and summary labels. Designed for Obsidian or any wiki-link viewer.
- **`JsonExporter`**: single `hierarchy.json` with complete tree: all chunks, blocks, bidirectional links, original text, contexts, summaries.

## Key Techniques

### LLM-mediated semantic boundaries
The core technique: instead of splitting at every N tokens, the LLM identifies where topics naturally end. The LLM returns a **verbatim boundary phrase** that must match character-for-character in the source text. One retry with explicit "copy EXACTLY" instructions, then sentence-boundary fallback. No fuzzy matching — the retry addresses the root cause (model paraphrasing) more directly.

### Self-sufficient chunk rewriting
`ChunkRewriter` ([[nodes/rewriting.py:9|nodes/rewriting.py:9]]) calls the LLM with the chunk's original text, injected context, and optional page images (PDF mode). The prompt ([[llm/prompt_templates/rewrite.txt|rewrite.txt]]) instructs: resolve all pronouns, clarify implied references, preserve ALL specific facts/numbers/names, remove academic citation markers, and for PDFs — transcribe tables into markdown, state chart values concretely.

### Contiguous grouping with progressive validation
During aggregation, groups must be contiguous runs of the ordered summary list. The `GroupValidator` ([[nodes/aggregation.py:25|aggregation.py:25]]) enforces:
- **Hard**: contiguity (flattened groups must match ordered list), `min_group_size` (rejects groups of 1 as degenerate)
- **Soft**: `max_group_size` exceeded triggers one re-prompt, then accepts the result

### Context budget as priority-ordered gate
Context assembly walks a priority list and inserts items whole-or-not-at-all. This prevents the common failure mode where a partially-filled context window contains sentence fragments that mislead the LLM. Items are `ContextItem` dataclasses with pre-computed token counts.

### Atomic checkpointing
`Checkpointer.save()` writes to a temp file via `mkstemp`, then atomically renames over the target. If the process crashes mid-write, the temp file is garbage. No `.tmp` cruft, no corrupted checkpoints.

### Vision-model PDF pipeline
PDF documents skip `pdftotext` entirely. Pages are rendered to PNG, page-level completeness checks use text-only extracts, and the vision model is invoked only during rewrite — where images are passed as `data:image/png;base64` URLs in a multi-part message. This means visuals (tables, charts, figures) are read by the model and their data lands in the context.

## Design Decisions

| Decision | Rationale | Trade-off |
|----------|-----------|-----------|
| **LLM for chunk boundaries** not fixed-size | Topics don't align to token counts | Higher cost per chunk; safety valves prevent runaway |
| **Verbatim exact-match validation** not fuzzy | Retry addresses paraphrasing root cause; fuzzy would accept wrong text | Requires one extra LLM call on mismatch |
| **Sequential Phase 1** not parallel | Cursor must advance through text in order | Linear wall-clock time; no parallelism gains |
| **Contiguous grouping** not arbitrary clustering | Preserves document narrative order | Less flexible than topic-based clustering |
| **Hard min_group_size=2** | Prevents degenerate single-child blocks | Short documents may hit grouping fallback more often |
| **Ollama-only** | Local-first, no cloud dependency | No GPT/Claude support; requires local GPU |
| **Single-model profile hardcoding** | Simple, sufficient for known models | New models require manual profile addition |
| **Atomic checkpoint via rename** | Prevents corrupted checkpoints from partial writes | Extra I/O; negligible overhead |
| **Vision model at rewrite only** | Saves tokens vs sending images for completeness checks | Page boundaries use text-only heuristics |

## Comparison Notes

- **vs [[RAPTOR]]**: Both build hierarchies bottom-up, but Chunker adds cursor-driven semantic chunking (Phase 1) and self-sufficient context rewriting at every node. RAPTOR focuses on embedding/retrieval; Chunker on navigation/reading.
- **vs GraphRAG**: Complementary. GraphRAG extracts entities/relationships for graph retrieval. Chunker produces navigable trees for progressive disclosure. Could feed Chunker's output as input to GraphRAG's entity extraction.
- **vs LangChain RecursiveCharacterTextSplitter**: Chunker uses LLM-mediated boundaries vs separator-based splitting. 2-3 LLM calls per chunk vs zero, but chunks are complete thoughts.
- **vs [[markitdown]]**: markitdown converts office docs to markdown. Chunker takes that markdown (or any text) and builds a knowledge tree on top.
- **vs [[MiMo Code]]'s memory extraction**: Both use hierarchical summarization to compress documents for agent consumption. MiMo applies it to codebase understanding; Chunker to any document.
- **vs [[DeepWiki]]'s instant codebase wikis**: DeepWiki answers questions about codebases. Chunker creates navigable document trees you browse — the artifact is the structure, not the answer.

## Tags

#tool #document-processing #llm #rag #chunking #knowledge-management #obsidian

---
*Sources: [[summary/sermakarevich-chunker]]*
*Last updated: 2026-06-15*
