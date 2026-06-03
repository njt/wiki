# Mnemo

A local-first AI memory layer that runs as a sidecar HTTP service: it watches conversations, extracts entities and relationships using any LLM (Ollama, OpenAI, Anthropic), builds a persistent knowledge graph in SQLite + petgraph, and retrieves relevant context for injection into future prompts — all in sub-50ms, with zero cloud dependency by default. Single static binary, Python SDK, works as a Docker compose stack with Ollama for fully local operation. The interesting design choice: no embeddings required — retrieval runs on SQLite LIKE + graph BFS + linear scoring, not a vector database.

Tags: #tool #project #knowledge-graph #memory #agents #local-first

---

## Architecture

A Rust workspace with strict one-way dependency: `mnemo-core` (lib, all business logic) → `mnemo-api` (thin Axum HTTP layer), `mnemo-cli` (10 subcommands), `mnemo-bench` (12 benchmark suites). The core has no knowledge of HTTP.

**Ingestion flow** (`crates/mnemo-api/src/main.rs:156-182`, `crates/mnemo-core/src/graph.rs:70-138`):
1. Chunk persisted to SQLite *before* LLM extraction — no data loss on failure
2. LLM extracts entities + relations via structured JSON schema prompt (`crates/mnemo-core/src/extractor.rs:7-37`)
3. Graph ingestion: deduplicates entities by name only, merges aliases/attributes, upserts relations with weight increment on repeat sightings (`ON CONFLICT weight = MIN(1.0, weight + 0.1)`)
4. Updates in-memory `petgraph::DiGraph` atomically under write lock

**Retrieval flow** (`crates/mnemo-core/src/retrieval.rs:29-134`) — 6-stage pipeline:
1. `LIKE %text%` chunk search + session-filtered chunks
2. `LIKE %text%` entity search + entities from top-10 chunks
3. BFS graph expansion from top-scored entities (neighbors get 0.5× penalty)
4. Relation filter (only relations where both endpoints are in accumulated set)
5. Score, sort, truncate by confidence threshold
6. Assemble `context_prompt` string with `[RELEVANT FACTS]`, `[RELATIONSHIPS]`, `[RELEVANT MEMORIES]` sections

**Database** (`crates/mnemo-core/src/db.rs`): Four SQLite tables — entities, relations, memory_chunks, memory_chunk_entities. WAL mode, foreign keys, busy_timeout=5000. UUIDs as TEXT, complex fields (aliases, attributes, embedding) as JSON strings. Migrations in `crates/mnemo-core/migrations/`.

**Knowledge graph** (`crates/mnemo-core/src/graph.rs:27-331`): `Arc<RwLock<DiGraph<GraphNode, GraphEdge>>>` hydrated from SQLite on startup (entities first, then edges). BFS with `visited: HashMap<NodeIndex, usize>` handles cycles. `find_path()` returns shortest path between entities.

**Provider abstraction** (`crates/mnemo-core/src/provider.rs`): Ollama/OpenAI/Anthropic/Custom with divergent API formats. Anthropic uses `/messages` + `x-api-key` header; all others use `/chat/completions` + Bearer auth.

## Key Techniques

**LIKE over embeddings**. The `embedding` column exists in the schema as a nullable JSON array but the core retrieval pipeline never uses it. Text search is `LIKE %query%` — fast enough for personal-scale memory, zero infra. Embeddings are opt-in infrastructure.

**Relation weight as confidence signal**. `ON CONFLICT DO UPDATE SET weight = MIN(1.0, weight + 0.1)` means every repeat sighting of the same (from, to, relation_type) triple increases confidence. The LLM doesn't need to know about existing relations — the database accumulates signal automatically.

**Dedup-by-name-only**. Two extractions of "Alice" as Person and "Alice" as Organization merge into one entity — the first extraction's type wins. `resolve_or_create_entity()` in `graph.rs:140-191` does the merge: aliases union-deduplicate, attributes overlay. Simplistic, but avoids fragmenting personal-scale graphs.

**Graph expansion penalty**. Neighbors found via BFS get 0.5× their base score. An explicit "less relevant" heuristic that prevents graph drift drowning out direct matches.

**Chunk-first storage**. The raw text hits SQLite before the LLM is called (`ingest_handler` in `main.rs:160-170`). If the LLM is down or returns garbage, `extract_with_fallback()` returns empty — no error, no data loss, re-extraction possible later.

**Docker health check via raw TCP**. The `--health-check` flag opens a TCP connection and exits 0/1. Works in scratch containers with no shell or curl. `main.rs:345-354`.

## Design Decisions

**Optimized for simplicity and locality**: SQLite over Postgres, in-memory petgraph over persistent graph DB, LIKE over vector search. The bet is that personal-scale memory fits in one SQLite file and one process's RAM.

**What was sacrificed**:
- **Concurrent writes**: Single `RwLock::write()` on the graph during ingest blocks all reads. Fine for a personal sidecar; would bottleneck under load
- **Semantic search quality**: LIKE is crude compared to embedding search. The embedding column is scaffolding without a pipeline
- **Entity disambiguation**: Name-only dedup means "John Smith the CEO" and "John Smith the plumber" are the same entity
- **Compile-time schema safety**: UUIDs, enums, dates are TEXT with runtime parsing. Simpler to inspect but no type-level guarantees beyond migration files

**The LLM IS the extraction engine**: No spaCy, no NER model, no entity linker. The system prompt + JSON schema (`EXTRACTION_SCHEMA` const in `extractor.rs:15-37`) defines the extraction contract. Quality scales with your LLM choice — use llama3 for free, Claude Opus for better extraction.

## Comparison Notes

Unlike **[[Rowboat]]**, which integrates email/docs into a knowledge graph for a personal AI coworker, mnemo is source-agnostic — a general-purpose memory API for any app to call. Rowboat's graph lives in Obsidian-compatible Markdown for transparency; mnemo's lives in SQLite + in-memory petgraph for speed.

Unlike **[[Understand-Anything]]** and **[[graphify]]**, which build codebase knowledge graphs from tree-sitter + LLM pipelines, mnemo targets conversational memory — free-text extraction rather than structured code analysis.

Unlike the **[[Agent Memory and Context]]** approaches that use vector search as the primary retrieval primitive, mnemo's 6-stage pipeline combines SQLite LIKE, graph BFS, and linear scoring — no embeddings required. This is philosophically closer to **[[SQLite is All You Need for Durable Workflows]]**'s "boring technology" thesis.

The sidecar service model mirrors **[[Claude Sidecar]]**, but mnemo focuses on knowledge extraction and retrieval rather than tool execution — it's a memory service, not an agent extension.

The dedup-by-name-only approach is a deliberate trade-off that would make entity resolution researchers wince — but for personal-scale knowledge graphs, it's probably right. Sophisticated disambiguation fragments entities and requires the user to resolve conflicts they don't care about.

---

*Sources: [[raw/mnemo]]*
*Last updated: 2026-06-04*
