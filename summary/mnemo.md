---
url: https://github.com/zaydmulani09/mnemo
title: mnemo — Local-first AI memory layer for any LLM
author: zaydmulani09
date_fetched: 2026-06-04
date_published: 2025
topics:
  - agent-memory-and-context
  - agent-architecture
---

# mnemo — Full Repo Analysis

A local-first AI memory layer sidecar: extracts entities and relationships from conversations via LLM, builds a persistent knowledge graph in SQLite + petgraph, and retrieves relevant context in sub-50ms for injection into future LLM prompts. Works with Ollama (free, local), OpenAI, Anthropic, or any OpenAI-compatible API.

## Project Stats

- **Language**: Rust (core + API + CLI + benchmarks) + Python SDK
- **Size**: ~122 Rust tests, ~21 Python tests, ~2,500 LoC Rust
- **Dependencies**: tokio, axum, sqlx, petgraph, reqwest, clap, serde
- **Crates**: mnemo-core (lib), mnemo-api (bin), mnemo-cli (bin), mnemo-bench (bin)

## Architecture

### Crate Structure

The workspace is strictly layered: `mnemo-api`, `mnemo-cli`, and `mnemo-bench` all depend on `mnemo-core`. `mnemo-core` has no knowledge of the HTTP layer.

| Crate | Type | Role |
|-------|------|------|
| mnemo-core | lib | All business logic — models, DB, graph, extraction, retrieval, providers |
| mnemo-api | bin | Thin Axum HTTP handler layer, wires AppState, delegates to core |
| mnemo-cli | bin | Blocking reqwest CLI with spinner UX, colored output, 10 subcommands |
| mnemo-bench | bin | 12 benchmark suites with colored ASCII output, optional JSON export |

### Data Flow

**Ingestion (POST /ingest)**:
1. Handler deserializes `IngestRequest`, constructs `MemoryChunk` with fresh UUID + UTC timestamp
2. Chunk is persisted to SQLite BEFORE extraction — no data loss on LLM failure
3. `Extractor::extract_with_fallback()` sends chunk to LLM with structured JSON schema prompt; returns `ExtractionResult` (entities + relations). Falls back to empty result on failure
4. Handler acquires write lock on `SharedGraph` (`Arc<RwLock<KnowledgeGraph>>`)
5. `graph.ingest()`: resolves/creates entities (dedup by name), links chunks to entities via join table, upserts relations, updates in-memory petgraph DiGraph
6. Returns `IngestResponse` with chunk_id, entities_extracted, relations_extracted, processing_time_ms

**Retrieval (POST /retrieve)** — 6-stage pipeline:
1. **Chunk search**: `LIKE %text%` query on memory_chunks + session-filtered chunks, merged by UUID
2. **Entity search**: `LIKE %text%` query on entities + entities from top-10 chunks, merged by UUID
3. **Graph expansion**: BFS over petgraph DiGraph from top-scored entities up to `graph_depth` hops; expanded entities get 0.5× score penalty
4. **Relation filter**: Only relations where both endpoints are in accumulated entity set
5. **Score + truncation**: Sort by score desc, filter below `min_confidence`, cap at `max_chunks`/`max_entities`
6. **Context assembly**: `build_context_prompt()` formats as structured string with optional `[RELEVANT FACTS]`, `[RELATIONSHIPS]`, `[RELEVANT MEMORIES]` sections

### Knowledge Graph

In-memory `petgraph::DiGraph<GraphNode, GraphEdge>` wrapped in `Arc<RwLock<>>`:
- `GraphNode`: entity_id, name, entity_type, confidence
- `GraphEdge`: relation_id, relation_type, weight
- Hydrated from SQLite on startup (entities first, then edges) to guarantee no missing-node edge warnings
- `get_neighbors(entity_id, depth)`: BFS with `visited: HashMap<NodeIndex, usize>` for cycle-safe traversal
- `find_path(from, to)`: BFS shortest path returning sequence of GraphNodes
- `reload()`: clears graph + node_index, re-hydrates from DB (used after wipe)

### Database Schema

Four SQLite tables with WAL mode, foreign keys enabled, busy_timeout=5000:
- `entities`: UUID-as-TEXT PK, name, entity_type (JSON-serialized enum), aliases (JSON array), attributes (JSON object), confidence, source_count
- `relations`: composite UNIQUE on (from_entity_id, to_entity_id, relation_type), ON CONFLICT increments weight
- `memory_chunks`: content, source, session_id (nullable), embedding (nullable JSON array of f32), metadata
- `memory_chunk_entities`: join table with FK cascade on both sides, PRIMARY KEY (chunk_id, entity_id)

### Entity Deduplication

Key insight: deduplication is by **name only** (not name+type). Two extractions of "Alice" as Person and "Alice" as Organization merge into one entity — the first extraction's type wins. This deliberate simplification avoids duplicate nodes for ambiguous names.

`db.upsert_entity()` on collision: increments source_count, merges aliases (union, deduplicated), overlays new attributes onto existing ones.

### LLM Provider Abstraction

`ProviderType` enum: Ollama, OpenAi, Anthropic, Custom. Two divergent API formats:
- **Anthropic**: POST /messages, x-api-key header, anthropic-version header, system at top level, response via content[]
- **OpenAI-compatible**: POST /chat/completions, Bearer auth, system in messages[], response via choices[0].message.content

`LlmProvider::complete()` retries up to max_retries (default 3) with 500ms backoff.
Temperature 0.1 (deterministic extraction).
Structured output enforced via JSON schema in system prompt (not via tool use / structured output API).

### Scoring Algorithm

**Chunk scoring** (`score_chunk`):
- base 0.5 + keyword_overlap × 0.1 (capped 0.4) + recency bonus (0.1 if <24h, 0.05 if <7d) + session match (0.15)
- Clamped [0.0, 1.0]

**Entity scoring** (`score_entity`):
- confidence × 0.5 + name match (0.3) + first alias match (0.2) + source_count × 0.02 (capped 0.2)
- Clamped [0.0, 1.0]

### Configuration

Precedence: env vars → TOML config file (`--config path`) → compiled-in defaults.
Defaults: Ollama at localhost:11434, llama3, port 8080, mnemo.db.

## Key Techniques

1. **SQLite LIKE as retrieval primary**: No vector embeddings required by default. The `embedding` column exists but is nullable and unused by the core retrieval pipeline. Text search via `LIKE %query%` is fast enough for local-scale memory. Embeddings are opt-in — you bring your own embedding pipeline.

2. **Relation weight as confidence signal**: `ON CONFLICT DO UPDATE SET weight = MIN(1.0, weight + 0.1)`. Each repeat sighting of the same (from, to, relation_type) triple increases confidence. This is a built-in "the LLM keeps seeing this" signal without needing the LLM to know about existing relations.

3. **Chunk-first storage**: The chunk is written to SQLite before extraction runs. If the LLM is unreachable or returns invalid JSON, the raw text is preserved — the extraction pipeline can be re-run later. `extract_with_fallback()` never panics, never errors to the caller.

4. **Graph expansion penalty**: Neighbors found via BFS graph traversal get a 0.5× score multiplier. This is an explicit "less relevant" heuristic that prevents the graph from expanding uncontrollably and drowning out direct matches.

5. **Session affinity**: Chunks and entities from the same session_id get scoring bonuses. This is the simplest possible implementation of "conversation context" — no need to model conversation structure, just tag chunks with a session ID.

6. **Docker health check without curl**: The `--health-check` flag does a raw TCP connection to the API port and exits 0/1. Works in scratch containers (no shell, no curl). Clever minimalism.

## Design Trade-offs

**Optimized for**:
- Simplicity: SQLite, not Postgres. In-memory petgraph, not Neo4j. LIKE search, not pgvector.
- Local-first operation: zero cloud dependency with Ollama
- Graceful degradation: chunk saved before extraction; extraction failures return empty results, not errors
- Speed: sub-50ms retrieval claimed (depends on DB size; graph is in-memory, text search is indexed)

**Sacrificed**:
- Scaling: Single-writer RwLock on the graph blocks reads during ingest. Fine for a personal sidecar, would bottleneck under concurrent writes
- Accuracy: LIKE text search is crude compared to embedding-based semantic search; the "embedding" column is infrastructure without an embedding pipeline
- Sophistication: No NER model, no custom entity linker, no disambiguation beyond name matching. The LLM IS the extraction engine
- Durability: In-memory graph must be reloaded from SQLite on restart — fine for a sidecar, not for a database server
- Type safety at SQL boundaries: UUIDs, enums, and dates are stored as TEXT with manual parse. Simpler to inspect/debug but no compile-time schema enforcement beyond migration files

## Innovation Points

1. **The 6-stage pipeline combines three retrieval paradigms in one pass**: SQLite full-text (LIKE), graph traversal (BFS), and linear scoring. No vector index, no embedding service, no external dependencies. This is a "good enough" design that would surprise engineers expecting a RAG stack.

2. **Dedup-by-name-only is surprisingly pragmatic**: Most entity resolution systems agonize over name+type+context disambiguation. mnemo's approach is "if it's called the same thing, it IS the same thing" — and the ON CONFLICT weight increment handles the confidence signal. In practice, for personal-scale knowledge graphs, this probably works better than more sophisticated approaches that would fragment entities.

3. **The extractor IS the LLM**: No separate NER pipeline, no spaCy, no HuggingFace model. The system prompt + JSON schema IS the extraction specification. This means extraction quality scales with your LLM choice — use llama3 for free/basic, use Claude Opus for better extraction. The architecture makes no assumptions about extraction quality.

4. **Embeddings as opt-in infrastructure**: The schema has an embedding column. The scoring doesn't use it. This is a deliberate separation of concerns — embedding generation is a separate concern from retrieval, and mnemo doesn't pick a winner (local embedding model vs API).

## Comparison Notes

- Unlike **[[Rowboat]]**, which integrates with email/docs and builds a knowledge graph as a personal AI coworker, mnemo is a general-purpose memory sidecar API — source-agnostic, embeddable in any app
- Unlike **[[Understand-Anything]]** and **[[graphify]]**, which build codebase knowledge graphs from tree-sitter + LLM, mnemo extracts from free-text conversations, not structured code
- Unlike **[[docmason]]** which builds knowledge bases from office documents with citations, mnemo is a live HTTP service, not a batch document processor
- Unlike LangChain/LlamaIndex memory modules which are framework-coupled, mnemo is a standalone HTTP service with a Python SDK — language-agnostic
- Unlike vector-only RAG systems, mnemo combines text search + graph traversal + scoring — no embeddings required by default. This is closer to **[[SQLite is All You Need for Durable Workflows]]**'s philosophy of "SQLite is the sensible default"
- The sidecar architecture mirrors **[[Claude Sidecar]]**'s approach of running AI memory as a separate service, but mnemo is focused on knowledge graphs rather than tool execution

## Test Coverage

- Rust: 122 tests (14 model + 22 DB + 15 extractor + 17 graph + 17 retrieval + 20 API integration + 12 provider/config + 5 CLI)
- Python: 21 tests (6 model + 10 sync client + 5 async client)
- Property-based tests (proptest) for scoring functions: ensure scores always in [0.0, 1.0], context prompt always valid UTF-8
- Graph tests cover cycles, large graphs (1000 nodes), subgraph dedup, concurrent upserts
