# Attemory

The first production-grade retrieval engine that replaces vector similarity with model attention as the core retrieval primitive. Rather than embedding documents and searching by cosine distance, Attemory indexes raw text into KV cache state and runs a local Qwen3.5 model that attends over memory and query simultaneously — the same mechanism an LLM uses to reason over context. This is a fundamentally different category from vector search, BM25, or late-interaction retrieval, and its benchmark results (SOTA-class on LongMemEval-M at million-token scale, 43.8% token reduction for Claude Code on SWE-QA) suggest the approach has real leverage.

---

## Architecture

Attemory is a two-tier system: a **C++ core engine** (`attemory-core` SDK, built on llama.cpp/ggml) and a **Python client layer** that wraps it in a Pythonic HTTP API plus CLI tools (`attemory-server`, `atcode`, `attemory-mcp`).

### C++ Core (`src/context/`)

The core is organized around a single `AttemoryContext` class (`src/context/context.h`) that delegates everything to `SessionManager` (`src/context/session/session_manager.cpp`). The architecture has five key subsystems:

**Session management.** Each retrieval session owns a `Session` struct (`src/context/session/session_state.h:36-43`) containing a `SessionStore` (persistent facts), a `SegmentPlan` (how memories are split across context windows), and KV cache metadata. Sessions are loaded lazily at startup from disk (`persistent::scan_sessions`) and can be created, restored, indexed, searched, and deleted.

**Segment planning** (`src/context/session/segment_planner.h`). Because model context windows are finite, Attemory splits large sessions into segments that each fit within 85% of the model's context limit (`kSegmentSoftLimitPercent = 85`). The planner estimates token counts for each memory using the retrieval model's tokenizer, then greedily packs memories into segments, auto-splitting when a segment would overflow. Manual segment boundaries (`next_segment`) let callers control grouping. The key constant is `kMaxSegmentsPerSession = 100`.

**KV cache management** (`src/context/kv/segment_kv_manager.h`, `src/context/kv/resident_kv_store.h`). This is the heart of the system. `SegmentKVManager` maintains a resident store (`ResidentKVStore`) of segment KV caches in memory, with an LRU-like eviction policy keyed on a configurable byte budget. Each segment goes through a lifecycle: *Missing* → *Resident* (in memory) → *DiskOnly* (persisted snapshot). The manager supports incremental indexing (stage a prefix, build from the last checkpoint), activation for search (load a segment's KV into the model's active context), and syncing search results back to persistent state.

**Search** (`src/context/search/search_runner.cpp`, `search_service.cpp`). Search works in two modes. For indexed sessions (`run_cached`), each segment's pre-built KV cache is activated into the model, the query tokens are appended, and `atmcore::run_search_with_query_tokens` runs a single forward pass. The model's attention weights produce ranked memory references. For oneshot/ephemeral search (`run_ephemeral`), KV state is built from scratch per query (`atmcore::build_active_kv_ephemeral`), then searched. Results are merged across segments and optionally limited per-segment (`top_k`). The `SearchQueryTokens` structure separates context tokens from query tokens, with the system prompt (the "system" memory) providing the framing.

**Persistence** (`src/persistent/`). Session metadata is stored as JSON files on disk; KV caches are binary blobs managed by `kv_cache_persistence.cpp`. The model cache key (`model_cache_key.h`) ensures different model versions get separate cache namespaces.

### Python Layer (`python/attemory/`)

The Python side is thin by design. `AttemoryClient` (`client.py`) is an HTTP client that wraps every API endpoint with typed models (`models.py`). `server.py` is a launcher that validates CLI args, resolves the native runtime binary, and execs it. Repository search lives in `code/` — `project.py` handles chunking source files (30-line chunks with blank-line-window boundary alignment), file discovery (90+ language suffixes), TOML config, and incremental index tracking via SHA-256 chunk hashes. The `atcode` CLI (`code/cli.py`) chains `init`, `scan`, `index`, and `search` subcommands.

### Dependency Graph

- **attemory-core** (closed-source SDK): provides `atmcore::Runtime`, model loading, tokenization, KV context building, and the actual attention-based retrieval call
- **llama.cpp / ggml**: the inference backend (acknowledged in README)
- **Qwen3.5**: the retrieval model family used for attention-based search
- **cpp-httplib** (vendored): HTTP server and client
- **nlohmann/json** (vendored): JSON parsing
- **Python**: `requests` for HTTP, `pytest` for testing; optional `mcp` extra

## Key Techniques

### Attention as retrieval primitive

The central innovation. Every other retrieval system (vector DBs, BM25, ColBERT, SPLADE) reduces documents to a compressed representation and searches by comparing those representations. Attemory instead loads the raw indexed text into a model's KV cache and runs a forward pass with the query appended. The model's attention weights over the memory tokens become the relevance signal. This means retrieval quality is bounded by the model's *understanding* of the text, not by how well an embedding captures it — a qualitative difference, not a quantitative one.

### Token-aware segment planning

The segment planner (`segment_planner.cpp`) doesn't just count characters or words — it runs the actual retrieval model's tokenizer to estimate KV cache token counts for each memory. It uses a two-pass estimation: a fast estimate (`estimate_new_segment_token_count`) for the common case, and a slower exact estimate (`estimate_segment_token_count_with_extra_memory`) when the fast estimate is near the limit. The append estimate formula includes a safety margin of 4 tokens (`kAppendTokenEstimateSafetyMargin`) to prevent borderline overflows. This is careful engineering, not hand-waving.

### Incremental KV cache building with disk persistence

Rather than rebuilding KV state from scratch for every search, Attemory supports incremental indexing: after each `add_memory`, it stages an incremental prefix (`stage_incremental_prefix`), letting subsequent index operations build from the last checkpoint. The `kv_persist` option writes segment KV caches to disk so later searches can restore them without recomputing. This is essential for the repository search use case — index a codebase once, search many times.

### Resident KV budget with LRU eviction

The `ResidentKVStore` (`resident_kv_store.h:36-69`) maintains an in-memory cache of segment KV state with a configurable byte budget. When adding a new segment would exceed the budget, the least-recently-used segments are evicted (with disk snapshots as fallback). The `evict_to_budget` method (private) tracks an `access_clock_` counter for LRU ordering. This is a classic caching pattern applied to model state rather than text data.

### Code chunking with language awareness

The repository search (`project.py:510-543`) chunks source files into 30-line windows, but with a blank-line-aware boundary heuristic: when a chunk boundary falls within a `blank_line_window` (default 10 lines) of a blank line, the boundary snaps to that blank line. This keeps functions and logical blocks together rather than splitting mid-routine. Language detection covers 90+ file extensions and 14 special filenames (Dockerfile, CMakeLists.txt, etc.).

## Design Decisions

**Local model requirement vs. API simplicity.** Attemory requires running a local inference model (Qwen3.5), which means GPU access and several GB of VRAM. This is a significant adoption hurdle compared to embedding APIs that need only a CPU. The trade-off is retrieval quality that embedding similarity can't match — and the benchmarks suggest it's a real quality gap, not a marginal improvement.

**C++ core + Python wrapper.** The heavy lifting (model inference, KV cache management, session persistence) lives in C++/llama.cpp, while the user-facing API is Python. This is pragmatic: Python for developer ergonomics, C++ for performance. But it means two build systems, platform-specific runtime wheels (6 CUDA variants + CPU + Metal), and a non-trivial install surface. The setup complexity is the project's biggest weakness — you need the right `attemory-runtime-*` wheel for your platform and CUDA version.

**Segment model vs. streaming.** Attemory splits memory into discrete segments (max 100) rather than using a sliding window or continuous stream. This simplifies KV cache management (each segment is independently buildable, indexable, and evictable) but means cross-segment relationships can only be captured by the merge step (`merge_segment_ranked_results`), not by the model's attention directly. For retrieval tasks where evidence spans segments, this is a genuine limitation.

**Attention-based retrieval as a new category.** Turnbull's [[Three Kinds of Agentic Search]] taxonomy (retrieval-centric, harness-centric, model-centric) doesn't have a slot for "the model reads the raw text directly." Attemory is retrieval-centric in that the search engine leads, but it's also model-centric in that the model's own attention mechanism — not an external ranking function — determines relevance. It's closer to "late interaction" models like ColBERT but goes further: no embeddings at all, just the model's forward-pass attention weights.

**KV cache as the universal interface.** Rather than building a custom index structure (inverted index, HNSW graph, IVF clusters), Attemory treats model KV state as the index. This means retrieval quality automatically improves when the underlying model improves — swap Qwen3.5 for a better retrieval model and everything gets better without changing the pipeline. It also means retrieval failures are model failures, not index failures, which is a cleaner failure mode to debug.

**Closed-source core SDK.** The `attemory-core` SDK is closed-source; only the session management, search orchestration, and Python wrapper are open. This means the actual attention-based retrieval implementation, the model loading, and the tokenizer integration are opaque. The C++ layer (`src/context/`) is orchestration code — it calls into `atmcore::*` functions but doesn't contain the retrieval algorithm itself. This is a significant limitation for understanding or modifying the system's core behavior.

## Comparison Notes

Unlike **vector databases** (pgvector, Chroma, Pinecone) which compress text into fixed-dimension embeddings and search by distance, Attemory performs no dimensionality reduction — the model attends over the actual tokenized text. The quality difference is most visible at scale: on LongMemEval-M (1.5M tokens, 500 sessions), Attemory achieves 92.55% message recall where most embedding-based systems degrade sharply.

Unlike **late-interaction retrieval** (ColBERT, SPLADE) which keeps per-token embeddings but still compares them via similarity scoring, Attemory uses the model's forward-pass attention weights directly. There's no separate scoring function — the attention mechanism IS the scorer.

Unlike **BM25 and learned sparse retrieval**, which match tokens, Attemory captures semantic relationships through attention patterns that span tokens. A query about "error handling in async code" can retrieve a chunk about "Promise.catch patterns" even when they share no keywords, because the model attended over both.

Unlike **RAG pipelines with rerankers** (the standard production setup), Attemory collapses retrieval and relevance scoring into a single model forward pass. No separate embedding model, no vector database, no cross-encoder reranker — just model attention. The SWE-QA result (43.8% fewer tokens) shows this matters in practice: when retrieval produces better-ranked results, the downstream agent does less wasteful exploration.

The most direct comparison is with **KV-cache-based long-context approaches** (like the KV cache sharing patterns in [[Recent Developments in LLM Architectures]]), but Attemory uses KV cache for *retrieval* rather than *generation* — it's a different use of the same mechanism.

---

*Sources: [[raw/attemory]], [[summary/attemory]]*
*Last updated: 2026-08-06*
*Tags: #tool #project #memory #retrieval #agents #search #kv-cache*
