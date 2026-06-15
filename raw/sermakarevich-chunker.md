---
url: https://github.com/sermakarevich/chunker
title: "Chunker: Hierarchical Document Chunking with Multi-Level Summaries"
author: sermakarevich
date_fetched: 2026-06-15
date_published: 2025
---

# Full Analysis

## Repository Overview

Chunker is a Python CLI tool (~2,700 lines of code) that transforms documents into navigable knowledge trees. It processes a document into a hierarchy of self-sufficient chunks and multi-level summaries, producing linked markdown files and JSON that an AI model (or human) can explore through progressive disclosure.

Dependencies: langchain-ollama, pydantic, pymupdf (for PDF). Uses Ollama for local LLM inference. Python 3.12+. Built with hatchling.

## Architecture

### Pipeline Pattern

The core is `Pipeline` (pipeline.py:49) which orchestrates a sequential loop:

```python
while state.has_more_input:
    chunk = extractor.extract_next(state)     # Phase 1
    chunk = rewriter.rewrite(chunk, state)    # Rewrite
    state.chunks[chunk.id] = chunk
    sweeper.sweep(state)                      # Phase 2 aggregation
    checkpointer.save(state)                  # Atomic checkpoint
```

Two extractors exist in parallel: `ChunkExtractor` for text documents (nodes/chunking.py:21) and `PageChunkExtractor` for PDF documents (nodes/page_chunking.py:94). The pipeline selects which to use based on whether `state.pages` is present.

### Two-Phase Processing

**Phase 1 — Cursor-Driven Chunking**: A `CursorWindow` (splitter.py:43) advances through text by expanding one split unit at a time. After reaching `min_chunk_tokens`, an LLM completeness check asks whether the window ends at a natural topic boundary. The LLM returns `{complete: bool, boundary_phrase: str}`. If complete, the boundary phrase is validated by exact substring match in the remaining text; if not found, a single retry with explicit verbatim instructions is attempted before falling back to sentence boundary. The window expands until complete, `max_chunk_tokens` is reached, or `max_expansion_attempts` is exhausted — any of which trigger a force-split.

**Phase 2 — Bottom-Up Aggregation**: `AggregationSweeper.sweep()` (nodes/aggregation.py:87) is called after every chunk. It checks whether pending summaries at the current level exceed `summary_aggregation_threshold` (token count) or `summary_count_threshold` (item count). If triggered, it: (1) calls the LLM to group pending summaries into contiguous, thematically coherent clusters, (2) validates grouping constraints (min_group_size is hard, contiguity is hard), (3) synthesizes a block context from children, (4) promotes the block to the next level's pending summaries. This recurses up until only one block remains (root) or thresholds are no longer exceeded.

### Key Abstraction: LoadedDocument

`load_document()` (loaders.py:29) dispatches on file suffix: `.pdf` files are rendered to page images via PyMuPDF (each page gets a PNG at configurable DPI) and stored as `Page` objects with both text and image_path; other files are read as plain text. This creates a mode switch — the pipeline's behavior branches on `state.pages is not None` throughout.

### Context Injection

`ContextBuilder` (context.py:18) assembles context for every LLM call in priority order:
1. Immediate predecessor's context (resolves local references)
2. Latest summary from each higher level (topic framing)
3. Earlier chunks walking backwards (broader context)

A hard `context_budget_tokens` cap is enforced with all-or-nothing insertion: items that would exceed the budget are skipped entirely (no partial insertion), and the builder tries the next priority item. Later chunks benefit from the hierarchy built by earlier ones.

### State and Checkpointing

`PipelineState` (state.py:9) holds all mutable state: cursor position, chunks dict, blocks dict, pending summaries per level, counters. `Checkpointer` (checkpoint.py:10) uses mkstemp + atomic replace for crash-safe writes — the pattern of write-to-temp, then rename guards against partial writes. The pipeline checkpoints after every single chunk/block, enabling resume without reprocessing.

### Output

Two formats from `nodes/output.py`:
- **MarkdownRenderer**: writes a directory tree with `index.md` entry point, `content/L0/`, `content/L1/`, etc. Each node is a self-sufficient markdown file with wiki-links to parent and children. Links display the target node's summary as a label.
- **JsonExporter**: writes a single `hierarchy.json` with the complete tree including all original text, contexts, summaries, and bidirectional parent/child links.

### LLM Service Layer

`LLMService` (llm/service.py:69) wraps all model calls through a single interface. Uses `langchain_core`'s `with_structured_output()` to enforce JSON schema compliance via Pydantic models. Retries up to 3 times with exponential backoff (2^n seconds), appending error context to the conversation on retry. Structured log entries for every call.

Five LLM call types defined in llm/schemas.py:
- `CompletenessResult`: `{complete: bool, boundary_phrase: str|null}`
- `PageCompletenessResult`: `{complete: bool, split_after_page: int|null}`
- `RewriteResult`: `{context, summary, filename}`
- `GroupingResult`: `{groups: list[list[int]]}`
- `BlockContextResult`: `{context, summary, filename}`

Prompt templates are external text files (llm/prompt_templates/*.txt) loaded via format strings — separating prompt engineering from code.

## Code Structure

| File | Lines | Purpose |
|------|-------|---------|
| pipeline.py | 169 | Top-level orchestration, resume logic |
| nodes/chunking.py | 131 | Text chunk extraction with completeness loop |
| nodes/page_chunking.py | 175 | PDF page-window chunk extraction |
| nodes/aggregation.py | 263 | Bottom-up summary grouping and synthesis |
| nodes/rewriting.py | 28 | Chunk rewrite via context injection |
| nodes/output.py | 192 | JSON export + Obsidian-compatible markdown |
| llm/service.py | 223 | LLM client with retry and structured output |
| llm/prompts.py | 55 | Prompt template loading and formatting |
| llm/schemas.py | 29 | Pydantic models for LLM responses |
| context.py | 109 | Priority-ordered context assembly |
| state.py | 98 | Mutable pipeline state + serialization |
| config.py | 74 | Configuration with model profiles |
| splitter.py | 103 | Text splitting strategies + CursorWindow |
| loaders.py | 78 | Text and PDF document loading |
| models.py | 132 | Chunk, SummaryBlock, Page data classes |
| checkpoint.py | 39 | Atomic checkpoint save/load |
| metrics.py | 116 | Step-level timing instrumentation |
| cli.py | 147 | argparse CLI: run + resume subcommands |
| tests/integration/test_pipeline_e2e.py | 535 | Full pipeline tests with mock LLM |

## Design Decisions

1. **LLM-mediated boundaries over fixed-size splitting**: The core bet is that paying for LLM calls during chunking produces higher-quality chunks than token-count splitting. The LLM finds semantic boundaries — places where a topic naturally ends. Safety valves (max tokens, max attempts) prevent runaway costs.

2. **Verbatim boundary phrase validation with exact match, not fuzzy**: When the LLM says "split here at this phrase," the system requires the phrase to be found character-for-character in the source text. One retry with explicit verbatim instructions. Then fallback to sentence boundary. No edit distance, no Levenshtein — the retry addresses the root cause (model paraphrasing) more directly than approximate matching would.

3. **Sequential Phase 1, batch Phase 2**: Chunking must be sequential because the cursor advances through the document. But aggregation is triggered per-chunk and can fire at multiple levels in a single `sweep()` call. Early chunks get less hierarchical context (the hierarchy hasn't been built yet); later chunks get rich multi-level summaries injected.

4. **Contiguous grouping constraint**: During aggregation, the LLM is constrained to contiguous runs of summaries — no reordering, no skipping. This preserves the document's narrative order. The fallback is even-sized grouping, not arbitrary clustering.

5. **Hard min_group_size of 2**: A group of 1 is rejected because it produces a higher-level block semantically identical to its single child — adding a level without adding information. This prevents degenerate hierarchies.

6. **Vision model for PDF, no pre-extraction**: PDFs skip `pdftotext` entirely. Pages are rendered to images and a vision model reads tables, charts, and figures directly. The text extraction from PyMuPDF is used only for page-level completeness checks (not for content). This preserves visual data that text extraction discards.

7. **Single-model architecture**: One Ollama model instance handles all LLM calls (chunking, rewriting, grouping, synthesis). For PDFs, a separate `--vision-model` can be specified (promoted to the effective model during PDF runs). No multi-model routing or tiering.

8. **Atomic checkpointing via mkstemp + rename**: Rather than write directly to the checkpoint file, data is written to a temp file then atomically renamed. This prevents corruption from partial writes if the process crashes mid-write.

## Comparison Notes

- **vs LangChain's RecursiveCharacterTextSplitter**: Chunker uses LLM-mediated semantic boundaries instead of separator-based splitting. The trade-off: higher cost (LLM calls per chunk) but chunks that are complete thoughts rather than position-based slices.

- **vs RAPTOR (recursive summarization)**: Both build hierarchies bottom-up, but Chunker adds the cursor-driven Phase 1 (extractive chunking with semantic completeness checks) and connects chunks into a navigable tree with self-sufficient contexts at every node. RAPTOR focuses on the embedding/retrieval side; Chunker focuses on the navigation/reading side.

- **vs GraphRAG**: GraphRAG extracts entities and relationships for graph-based retrieval. Chunker produces a document tree for progressive disclosure. Complementary approaches — GraphRAG for "who/what is related," Chunker for "what does the document say, in order."

- **vs simple markdown-to-wiki converters**: Chunker uses the LLM to understand document structure, not just parse headings. It creates self-sufficient contexts at every level rather than just splitting on markdown headers.

## Innovation Points

1. **Cursor-window expansion loop with LLM gate**: Rather than chunk-then-check, the window grows one unit at a time and the LLM acts as a gate: "is this complete yet?" This is more token-efficient than sending potential boundaries to the LLM for ranking.

2. **Priority-ordered context injection with all-or-nothing insertion**: Context items are strictly ordered by relevance, and partial insertion is forbidden. This prevents context from being filled with sentence fragments that confuse more than they help.

3. **PDF mode as full alternative path**: Rather than a separate preprocessing step, PDF handling is integrated as a parallel extractor and a vision rewrite step. The same pipeline, aggregation, and output code runs for both modes.

4. **Self-sufficient context at every tree node**: Chunks are rewritten to resolve pronouns and make implied subjects explicit. Summary blocks synthesize children into chunk-sized contexts. Every node is independently readable.

## Limitations

- **Sequential bottleneck**: Phase 1 is inherently sequential (cursor-driven). For very long documents, this means wall-clock time scales linearly with document length.
- **Ollama-only**: Hard dependency on Ollama running locally. No cloud LLM provider support.
- **No embedding integration**: Produces a knowledge tree but doesn't generate embeddings for retrieval. The output is designed for human navigation or agentic tree traversal, not vector search.
- **Single-model profile hardcoding**: Model profiles (token factors) are a small hardcoded dict in config.py rather than auto-detected.
- **No incremental updates**: Document changes require full reprocessing. The checkpoint format is designed for partial reprocessing in a future version but doesn't implement it.
