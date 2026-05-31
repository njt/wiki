# KTX Context Layer for Data Agents

An open-source context layer that builds a git-versioned, reviewable surface from warehouse metadata, BI tool definitions, query history, docs, and approved metric definitions — then serves it to data agents via 11 MCP tools and a CLI. Unlike agent memory systems that create opaque embeddings, KTX produces auditable Markdown and YAML files that humans review, diff, and merge.

---

## Architecture

**Two-language split:** TypeScript (~100K lines in `packages/cli/`) for the CLI, MCP server, ingest pipeline, memory system, search, and LLM orchestration. Python (~8K lines in `python/ktx-sl/`) for the semantic compiler — query planning, join graph analysis, and SQL generation via sqlglot.

**CLI-first monorepo.** A Commander-based CLI (`cli-program.ts:247`) with 9 top-level command groups provides setup, connection management, ingest, wiki, semantic-layer, SQL, status, MCP, and admin commands. Project discovery walks up from cwd looking for `ktx.yaml`. The MCP server is invoked as `npx @kaelio/ktx mcp stdio` — no separate daemon needed.

**Config as Zod schema.** The `ktx.yaml` schema (`config.ts:240-253`) is a single Zod object describing connections (7 databases), LLM providers (Anthropic, Vertex, gateway, Claude Code, none), embedding backends, ingest adapters (dbt, Looker, LookML, Metabase, MetricFlow, Notion, live DB, historic SQL), scan/enrichment config, storage backends (SQLite/Postgres), memory, and agent features. Every field has `.describe()`. The schema exports to JSON Schema for tooling; there is no separate documentation file.

**Ingest pipeline with LLM work units.** Seven adapters pull evidence from context sources; the pipeline splits work into units, each dispatched to an LLM agent with a skill set and step budget (`stage-index.types.ts`). Work unit outputs are MemoryActions (wiki create/update, SL create/update). The reconciliation stage handles conflicts (structural duplicates, near-duplicates, definitional contradictions). LLM agents have access to warehouse-verification tools so they can validate claims against actual data before writing.

**MCP server with 11 tools** (`context-tools.ts`): `connection_list`, `wiki_search`, `wiki_read`, `sl_read_source`, `sl_query`, `entity_details`, `dictionary_search`, `discover_data`, `sql_execution`, `memory_ingest`, `memory_ingest_status`. Each tool has Zod input/output schemas, title/description/annotations, and structured output for MCP clients. Telemetry per request with client identity. Progress reporting for long-running queries.

**Git worktree isolation for agent writes** (`memory-agent.service.ts`). The memory agent creates a per-session git worktree, runs the LLM loop against it, and only squash-merges after a pre-merge validation gate re-checks every touched SL source (YAML schema + warehouse dry-run). Failed writes are reverted. Concurrent sessions write in parallel; only the final merge serializes under a brief lock. Session conflicts trigger targeted DB rollback so state stays consistent.

**Hybrid search with Reciprocal Rank Fusion** (`hybrid-search-core.ts`). Multiple retrieval lanes (BM25, vector, keyword) run in parallel; results fuse via RRF with configurable per-lane weights and `k` parameter. Each lane is a SearchCandidateGenerator. Match reasons track lane contributions per result.

## Key Techniques

**Aggregate locality.** The Python semantic compiler's most sophisticated feature. When a query spans independent measure sources (a chasm trap), the planner (`planner.py:932-1111`) pre-aggregates each source group in its own CTE before joining, preventing double-counting from one-to-many joins. Measure groups are merged only when provably row-safe (one_to_one edges or grain-key many_to_one edges).

**Dijkstra with edge-cost bias.** The join graph (`graph.py:106-158`) uses Dijkstra's algorithm with cost 1 for safe edges (many_to_one, one_to_one) and cost 10 for one_to_many. This naturally prefers safe paths. Ambiguity detection identifies multiple equal-cost paths and warns. Join tree construction uses a Steiner tree approximation — pick a root, find shortest paths to each target, merge edges.

**Predefined measure chain expansion.** Measure dependencies (profit = revenue - cost, margin = profit / revenue) are resolved recursively through topological sort (`planner.py:713-736`). Circular dependencies error. Missing dependencies are auto-added. Name collisions between sources are resolved by qualification.

**Three-defense validation for agent writes.** Every LLM-powered write passes through: (1) worktree isolation (invisible until merged), (2) skill-based tool gating (agents use only tools their loaded skill exposes), (3) pre-merge validation gate (YAML schema + warehouse dry-run; failures revert). The agent has autonomy within the sandbox but deterministic gates at the boundary.

**Evidence-tool verification pattern.** Ingest work units receive warehouse-verification tools (`discover_data`, `entity_details`, `sql_execution`) so they can verify claims against the actual database before writing. This is the structural-backpressure pattern applied to knowledge ingestion — the agent can't persist something the warehouse contradicts.

**Thin connector boundaries.** Database connectors (`connectors/`) expose only their registry entry. Internal implementation uses `/** @internal */` JSDoc; `scripts/check-boundaries.mjs` enforces that no non-registry code imports connector internals. This prevents the per-variant switch anti-pattern the codebase explicitly calls out in `docs/code-design.md:103-121`.

## Design Decisions

**Filesystem as primary store, databases as index.** Wiki pages and SL sources are authoritatives git-tracked files. SQLite/Postgres stores search indexes and state — rebuildable from `reindex`. This inverts the typical pattern of DB-as-source-of-truth with files as exports.

**Python for the compiler, TypeScript for the harness.** The semantic compiler needs AST-level SQL parsing (sqlglot); the CLI/MCP/server stack needs Node.js. The boundary is a subprocess call. Pragmatic split: strongest ecosystem wins each domain.

**Context layer, not agent memory.** KTX's wiki and SL sources are shared, reviewed, durable definitions — the organization's canonical surface. Per-agent memory is a different problem. The distinction: context is the team wiki; memory is the personal notebook.

**Local-first, npx one command.** The MCP server is `npx @kaelio/ktx mcp stdio`. No cloud, no separate daemon, no API keys beyond the user's existing LLM provider config. Trade-off: requires a `ktx.yaml` project setup first.

**Seven validation checks on every semantic source.** The engine validates: orphan joins, invalid grain, join column consistency, SQL join coverage, disconnected components, column existence, and visibility rules. Unusually thorough for YAML-defined semantics — most tools leave validation to runtime.

## Comparison Notes

**vs. Metrics SQL:** Both define measures in YAML and compile to SQL. KTX auto-invents from dbt/Looker/Metabase; Metrics SQL requires hand-authoring. KTX adds wiki context and review workflow. Metrics SQL has the cleaner SQL interface.

**vs. dbt Semantic Layer / MetricFlow:** Both use YAML semantic models. KTX imports from MetricFlow. KTX runs fully locally, adds wiki context, and provides MCP tools. dbt requires dbt Cloud for the semantic API.

**vs. Wuphf / LLM Wiki:** Similar git+markdown wiki substrate. KTX adds executable metrics (YAML measures + SQL compilation), an ingest pipeline from warehouse/BI tools, and pre-merge validation. Wuphf is wiki-only.

**vs. Claude-Mem / Stash:** Agent memory systems (per-agent, opaque). KTX is a context layer (shared, reviewed, version-controlled). Output is git-tracked files, not embeddings.

**vs. Cube:** Both YAML-defined semantics + SQL generation. Cube is a deployed server (REST/GraphQL). KTX runs in-process via CLI/MCP. Cube targets dashboards; KTX targets agents.

**vs. DAB:** DAB exposes raw database access via REST/GraphQL/MCP. KTX adds a governed semantic layer with business context on top.

---

Tags: #tool #project #agents #database #semantic-layer #context-engine #mcp

*Sources: [[raw/ktx-ai-data-agents-mcp-context-skills]]*
*Last updated: 2026-05-31*
