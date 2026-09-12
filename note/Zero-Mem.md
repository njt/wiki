# Zero-Mem

A clean-room Rust implementation of Zero-Mem (Xiao et al., arXiv:2607.29377): a fully deterministic, zero-token agent memory system where every operation from ingestion through retrieval is a classical information retrieval pipeline — no LLM calls, no generated summaries, no embeddings for reasoning. Raw conversation turns are the immutable source of record; retrieval is structured search over them using an entity-context graph and temporal hierarchy. The only LLM call is the host agent's final answer generation, optionally calibrated against typed evidence candidates retrieved by the memory.

Why it matters: most agent memory systems ([[Mnemo]], [[Sawtooth Memory]], [[Claude-Mem]]) use LLMs for extraction, summarization, or retrieval — which means they consume tokens, incur latency, and inject nondeterminism into what should be a deterministic subsystem. Zero-Mem proves you can build a high-quality retrieval pipeline with zero ongoing LLM cost: heuristic NER, BM25, Personalized PageRank, and cosine similarity are the *only* primitives. For agent deployments where token budgets matter, this is a genuinely different trade-off point.

Tags: #tool #project #agents #memory #retrieval #deterministic

---

## Architecture

The system is a single Rust library (`crates/zeromem/src/lib.rs`) with 17 modules, ~2,700 lines total. Two public entry points — `ZeroMem::open()` for file-backed SQLite and `ZeroMem::open_in_memory()` for testing — plus three consumer surfaces: a `zm` CLI (`main.rs`, 83 lines), a PyO3 Python module (`zeromem-py/src/lib.rs`, 104 lines), and a Hermes Agent memory provider plugin (`hermes/zeromem/__init__.py`, 208 lines).

### Core pipeline (`lib.rs:172-236`)

The `query()` method runs a 7-stage deterministic pipeline:

1. **Profile** (`profile.rs`): Parse the query into subjects (canonical entities), keywords (content tokens minus stopwords), answer type (8-way classifier: Person/Time/Place/Number/List/Boolean/Entity/Open), temporal cues (date ranges, recency preference), session boundary, and exact-match phrases. Entirely regex-based, no ML.
2. **Route** (`route.rs`): Choose Relational (entity-heavy questions with no temporal cues) or Local (temporal, boundary-anchored, or aggregation questions). Both views always run; the route only sets the fusion weights (0.6/0.4 split, inverted per route).
3. **Graph view** (`graph_view.rs`): Seed entity activations from query subjects (exact match or dense alignment via cosine over entity embeddings), propagate through the bipartite entity-turn co-occurrence graph, then run Personalized PageRank (30 iterations, damping gamma=0.6) over the combined entity-turn graph with turn-turn adjacency edges. Exact phrase matches boost scores by 25%.
4. **Hierarchical view** (`hier_view.rs`): Coarse-to-fine beam search: score all episodes (best turn + avg), keep top 4; score windows within surviving episodes (centroid cosine + best turn), keep top 8; then score individual turns within surviving windows with a compatibility bonus (subject matches, temporal range hits, boundary matches, answer-type entity presence, phrase matches).
5. **Fuse** (`fuse.rs`): Each view's scores are independently min-max normalized over its own candidate pool, then linearly combined with route-specific weights. Absent-from-a-view scores count as zero — the normalized floor, not a penalty.
6. **Closure** (`closure.rs`): For each main evidence turn, add the best graph bridge (turn sharing an entity, scored by min shared weight) and local neighbors (adjacent in-session turns with wider span for short or anaphoric turns). Bridges and neighbors score at 0.5× the anchor's score.
7. **Calibrate** (`calibrate.rs`): Filter by boundary (session ordinal must match), remove superseded mains (older date/number turns when query prefers latest), sort by score, truncate to top-k (default 5), attach up to 2 supporting evidence items per main piece. Total output budget is 2× top_k.

### Supporting structures

**Entity-context graph** (`graph.rs`, `ner.rs`): Heuristic NER extracts six entity kinds (Named, Date, Number, Quote, URL, Email) via regex patterns + capitalized-span detection with connector-word joining ("Museum of Modern Art"). The graph stores entities with canonical names, postings lists (entity→turns), and turn-entity lists (turn→entities). Edge weights are co-occurrence counts normalized per turn: w(d,e) = c(e,d) / Σ c(e',d). No inferred relations — only observed co-occurrence.

**Temporal hierarchy** (`hierarchy.rs`): Turns → windows (default 4 turns each, online centroid averaging) → episodes (split on session change, time gap >6h, or adjacent-window centroid cosine <0.35). The hierarchy is built online during ingest — `push_turn()` decides in real time whether to extend the current window or start a new one.

**Embedders** (`embed.rs`): Two implementations behind the `Embedder` trait. `FastEmbedder` wraps bge-small-en-v1.5 (384-dim) via fastembed-rs/ONNX — downloads the model on first use. `HashEmbedder` is a signed feature hasher over token + character trigram features with FNV-1a hashing, 256 dimensions by default — used in tests and as the offline fallback. Both L2-normalize their output vectors.

**Store** (`store.rs`): SQLite with two tables — `turns` (id, session_id, session_turn, speaker, text, ts) and `embeddings` (kind, key, vec as f32 LE bytes). Embedding vectors are cached in SQLite; on open, if the embedder ID has changed, cached vectors are cleared. Graph, hierarchy, and BM25 index rebuild entirely from turns on open — the store cannot drift from the indexes.

### Hermes Agent integration (`hermes/zeromem/__init__.py`)

A Python plugin implementing Hermes's `MemoryProvider` interface. Key design decisions:
- **Background writer thread**: `sync_turn()` pushes (session_id, speaker, text) tuples onto a `Queue`; a daemon thread drains them into the Rust core. No blocking on the agent's main loop.
- **Prefetch with background thread**: `queue_prefetch()` spawns a daemon thread per query so the agent doesn't wait for retrieval before responding. The prefetched result is cached and returned by `prefetch()`.
- **Current-session filtering**: `_recall_block()` strips out turns from the current session (already in context), returning only cross-session evidence.
- **Two tools**: `zeromem_recall` and `zeromem_stats`, exposed via `get_tool_schemas()` and handled in `handle_tool_call()`.
- **Read-only subagent/cron mode**: `initialize()` checks `agent_context` — non-primary contexts skip the writer thread and never call `sync_turn()`.

## Key Techniques

**Heuristic NER as the only extraction engine** (`ner.rs:110-198`): No spaCy, no ONNX NER model, no LLM. Six regex patterns handle structured types (URLs, emails, dates, numbers, quotes). Capitalized-word spans with connector joining ("of", "the", "de", "van", etc.) handle names. Sentence-start noise filtering removes capitalized stopwords and discourse markers that aren't entities. This is the biggest deviation from the paper (which uses spaCy) and it works surprisingly well — the integration tests pass on a 19-turn corpus with multi-session, multi-speaker scenarios.

**Entity alignment as cosine search** (`graph_view.rs:42-67`): When query subjects don't match known entities by name, the system falls back to dense alignment — it computes the query embedding and finds the entity with the highest cosine similarity above `align_threshold` (0.55). This bridges the gap between what the user said ("Carrie's roommate") and what the graph knows ("carrie"). The alignment threshold means the system refuses to force a match rather than hallucinate one.

**Co-occurrence-only graph edges** (`graph.rs:61-68`): Unlike [[Mnemo]] or [[Context Graphs]], there are no relation types, no inferred connections, no typed edges. The graph only knows that two entities appeared in the same turn and how often. This is deliberately impoverished — it avoids the hallucination risk of inferring relationships — and relies on PPR's propagation to surface indirect connections.

**Online hierarchy construction** (`hierarchy.rs:33-93`): Episodes and windows are built incrementally as turns arrive, with a running centroid that's a weighted moving average. The decision to split an episode compares the just-completed window's centroid against the previous window's in the same episode — if cosine similarity drops below 0.35, a new episode starts. This is a lightweight topic-drift detector with zero extra inference cost.

**Boundary as hard constraint** (`calibrate.rs:67-76`, `hier_view.rs:111-119`): Session boundaries aren't just a signal boost — they're enforced. Evidence from the wrong session when a boundary is specified gets a -0.9 compatibility penalty (effectively excluded). And in calibration, `violates_boundary()` filters before scoring. This is stricter than most agent memory systems, which treat session boundaries as soft ranking signals.

**Superseded-value detection** (`calibrate.rs:80-110`): When a query asks for a latest date or number, older turns with conflicting values are dropped. The system compares pairs of turns that both contain the right entity kind (Date or Number) — if a newer turn has values disjoint from an older turn's values, the older turn is superseded. This handles "When is the opening at the latest?" correctly without needing an LLM to understand state transitions.

## Design Decisions

**Optimized for determinism and cost, not semantic richness.** The system makes no LLM calls during ingest, profile building, retrieval, or calibration. Every decision is a regex, a cosine comparison, a PPR iteration, or a BM25 score. The trade-off: the system can't understand that "Lychee" is a dog or that "Carrie's roommate" refers to "Panat" — it relies on co-occurrence to surface these connections. For agent memory where the LLM will read the retrieved evidence and do the understanding, this is the right split of responsibilities.

**Immutable turns, derived indexes.** The SQLite store is append-only for turns. The entity graph, temporal hierarchy, and BM25 index are rebuilt from turns on every open. This eliminates index drift (the store is always the source of truth) at the cost of startup time proportional to the number of turns. For personal agent memory with thousands (not millions) of turns, this is a good trade.

**Swappable embedders and NER behind traits.** The `Embedder` trait (`embed.rs:3-8`) and `EntityExtractor` trait (`ner.rs:45-47`) make the two ML-adjacent components pluggable. The default embedder selection (`lib.rs:256-269`) tries fastembed first, falls back to hash embedding if ONNX fails. The NER defaults to `HeuristicNer` but could be replaced with a proper spaCy/ONNX model via the trait — the paper's approach was always "anything non-generative qualifies."

**What was sacrificed:**
- **Entity disambiguation**: Name-only dedup, no coreference resolution. "Carrie" and "she" are different unless heuristic NER catches both
- **Relation types**: The graph knows entities co-occurred; it doesn't know how. This limits relational queries that depend on typed edges
- **Scalability**: PPR over the full graph (entities + turns) is O(iterations × |E| × |T|) per query. Fine for thousands of turns; would need approximate methods at scale
- **Cross-lingual support**: Tokenization is English-centric (apostrophe handling, stopword list). The NER patterns are English-only

## Comparison Notes

Unlike **[[Mnemo]]**, which uses an LLM for entity/relation extraction and stores a typed knowledge graph, Zero-Mem is entirely deterministic — its entity graph has no relation types, only co-occurrence weights. Mnemo builds a richer semantic structure but pays per-turn token costs; Zero-Mem pays zero tokens but can't answer relational questions that depend on understanding *how* entities relate.

Unlike **[[Sawtooth Memory]]**, which runs LLM-powered compression on a background worker, Zero-Mem does no summarization at all. Raw turns are the only artifacts. Sawtooth's L1.5 entity ledger preserves exact deterministic values; Zero-Mem achieves the same guarantee by simply never generating summaries that could hallucinate them away.

Unlike **[[Context Graphs]]**, which proposes typed edges capturing *why* decisions were made, Zero-Mem's graph is deliberately impoverished — co-occurrence only. The argument is the same from opposite directions: Context Graphs says typed edges add priceless reasoning context; Zero-Mem says untyped edges avoid hallucinated structure and still propagate useful signal via PPR.

Unlike **[[Agent Memory and Context]]**'s landscape of LLM-dependent memory tools ([[Claude-Mem]], [[CodeMira]], [[mira-OSS]]), Zero-Mem is in a category of its own: zero-token, fully deterministic retrieval. The closest cousin conceptually is [[Just Brute Force Your Embeddings]], which makes the analogous argument for vector search — measure the simple thing before reaching for the complex one.

The retrieval architecture (dual-view with fusion) is closest to hybrid retrieval systems like **[[QMD]]** and the ranking-fusion patterns in **[[Cerebras Knowledge Base Architecture]]**, but Zero-Mem adds the entity-context graph as a third retrieval axis alongside dense and lexical.

---

*Sources: [[raw/zeromem]], [[summary/zeromem]]*
*Last updated: 2026-08-06*
