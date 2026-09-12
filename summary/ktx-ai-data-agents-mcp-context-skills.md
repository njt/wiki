---
url: https://github.com/Kaelio/ktx-ai-data-agents-mcp-context-skills
title: KTX — Context Layer for Data Agents
author: Kaelio
date_fetched: 2026-05-31
date_published: 2024
topics:
  - databases-and-data
---

# KTX — Full Architecture Analysis

## Summary

**ktx** is an open-source context layer for data agents. It builds a git-versioned, reviewable surface from warehouse metadata, BI tool definitions, query history, docs, and approved metric definitions. Agents access this surface via 11 MCP tools and a CLI. The ingestion pipeline uses LLM-powered work units that reconcile new evidence with accepted context, writing wiki pages (Markdown) and semantic-layer sources (YAML) into a git-tracked directory. All writes are validated through a pre-commit gate (YAML schema + warehouse dry-run) before merging.

The project is split across two languages: TypeScript (~100K lines in `packages/cli/`) for the CLI, MCP server, ingest pipeline, memory system, search, and LLM orchestration; Python (~8K lines in `python/ktx-sl/`) for the semantic layer compiler (query planning, join graph, SQL generation).

## Architecture

**Overall pattern:** CLI-first monorepo with a Python sub-project for the semantic compiler. The TypeScript side follows a layered architecture: CLI commands → context services → database connectors / LLM runtime / file stores.

**Key layers (TypeScript):**

1. **CLI layer** (`packages/cli/src/`): Commander-based CLI with 9 top-level command groups (setup, connection, ingest, wiki, sl, sql, status, mcp, admin). Project discovery via `ktx.yaml` walking up from cwd. Interactive setup wizard using `@clack/prompts`. Telemetry via PostHog.

2. **Config layer** (`packages/cli/src/context/project/config.ts`): Zod-typed `ktx.yaml` schema — connections, LLM providers (anthropic/vertex/gateway/claude-code/none), embedding backends (openai/sentence-transformers/none), ingest adapters, scan configuration, storage backends (SQLite/Postgres), memory, and agent configuration. Schema exported as JSON Schema for tooling.

3. **Connection layer** (`packages/cli/src/connectors/`): Seven primary-source connectors (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite), each with dialect, connector, and live-database-introspection modules. Strict internal-only export boundary enforced by `scripts/check-boundaries.mjs`. Dialect dispatch via single registry tables in `drivers.ts`/`dialects.ts`.

4. **Ingest pipeline** (`packages/cli/src/context/ingest/`): Multi-adapter system with adapters for dbt, Looker, LookML, Metabase, MetricFlow, Notion, live databases, and historic SQL. Six ingest stages: source acquisition → integration → work units (LLM-powered, each with its own skill set and step budget) → reconciliation → finalization → save. Work unit outputs are MemoryActions (wiki creates/updates, SL creates/updates). Conflict detection handles structural duplicates, near-duplicates, definitional contradictions, and re-ingest changes.

5. **Memory system** (`packages/cli/src/context/memory/`): MemoryAgentService ingests free-form knowledge via git worktree isolation. Three-phase flow: (1) create per-session worktree, (2) run LLM loop with skill-based tool selection, (3) squash-merge back to main under lock. Pre-merge gate re-validates all touched SL sources and reverts failures. Session conflicts trigger targeted DB rollback.

6. **Semantic layer service** (`packages/cli/src/context/sl/`): TypeScript wrapper around the Python compiler. Indexes sources into SQLite FTS5 for search. Descriptions normalized. Query execution routes through the Python engine.

7. **Search** (`packages/cli/src/context/search/`): Hybrid search core combining multiple lanes (BM25, vector, keyword) via Reciprocal Rank Fusion (RRF). Each lane is a SearchCandidateGenerator with configurable weight. Lane results are fused into a ranked list with match reason tracking. Supports PGlite-backed Postgres hybrid search prototype.

8. **MCP server** (`packages/cli/src/context/mcp/`): 11 tools registered: connection_list, wiki_search, wiki_read, sl_read_source, sl_query, entity_details, dictionary_search, discover_data, sql_execution, memory_ingest, memory_ingest_status. Each tool has Zod input/output schemas, title/description/annotations. Telemetry per-request with client identity tracking. Structured output for MCP clients that support it. Progress reporting for long-running queries.

9. **LLM runtime** (`packages/cli/src/context/llm/`): Abstraction over AI SDK v6. Supports Anthropic, Vertex AI, OpenAI-compatible gateways, and Claude Code session. Prompt caching configuration per segment (system/tools/history). Embedding via OpenAI or sentence-transformers. Debug request recorder.

10. **Skills registry** (`packages/cli/src/context/skills/`): File-system-based skill discovery. Skills are directories containing `SKILL.md` with YAML frontmatter (name, description, callers). Caller-gated access (research agent, memory agent). Skills loaded at runtime by agents via `load_skill` tool.

## Key Techniques

**Git worktree isolation for agent writes:** The memory agent creates a per-session git worktree, runs the LLM loop against it, and squash-merges only if validated. This is the same pattern as `ctx` and Claude Code's own worktree isolation, but applied to knowledge ingestion rather than code changes. Concurrent sessions can write in parallel; the `config:repo` lock serializes only the final merge.

**Aggregate locality in SQL generation:** The semantic layer engine's most sophisticated technique. When a query spans multiple independent measure sources (a chasm trap), the engine pre-aggregates each source group in its own CTE before joining. This prevents double-counting from one-to-many joins. The planner detects fanout via the join graph's relationship types and groups measures by safe-merge criteria (one_to_one edges + grain-key many_to_one edges).

**Dijkstra-based join path resolution:** The join graph (`graph.py`) uses Dijkstra's algorithm with path cost biased against one_to_many edges (cost 1 for safe edges, cost 10 for one_to_many). This prefers safe paths through the graph. Ambiguity detection finds multiple equal-cost paths and warns. Steiner tree approximation builds minimal join trees for multi-source queries.

**Reciprocal Rank Fusion (RRF) for hybrid search:** The search system fuses multiple retrieval lanes (BM25, vector, keyword) via RRF with configurable per-lane weights and `k` parameter. Each lane independently generates candidates; scores are fused; deduplication by ID preserves the best rank per lane. Match reasons track which lanes contributed to each result.

**Predefined measure chain expansion:** The query planner fully resolves chains of predefined measures (e.g., profit = revenue - cost, margin = profit / revenue) through recursive dependency expansion. Dependencies are topologically sorted, auto-added when missing, and name collisions between sources are resolved by qualification.

**Zod schema as source of truth:** The entire `ktx.yaml` config schema is a single Zod object (`ktxProjectConfigSchema`) with `.describe()` on every field. This schema serves as validation, JSON Schema export, documentation, and TypeScript type source simultaneously. The pattern eliminates drift between config parsing, documentation, and type definitions.

**Evidence-tool pattern for ingest verification:** The ingest pipeline provides warehouse-verification tools (discover_data, entity_details, sql_execution) to LLM-powered work units so they can verify claims against the actual database before writing. This is the "structural backpressure" pattern applied to knowledge ingestion — the agent can't write something the warehouse contradicts.

**Thin connector boundaries with enforced isolation:** Database connectors are intentionally thin (dialect, connector, introspection). The `/** @internal */` JSDoc convention plus `scripts/check-boundaries.mjs` enforces that connector internals are never imported from outside the connector directory except through the registry tables. This prevents the per-variant switch anti-pattern the codebase explicitly forbids.

**Self-healing memory with validation gate:** After the memory agent's LLM loop writes SL sources, a pre-merge gate re-validates every touched source (YAML schema + warehouse dry-run compilation). Sources that fail validation are reverted to their pre-session state. This is crash-only software applied to knowledge: the agent can write freely, but only validated output survives the merge.

## Design Decisions

**Filesystem as the primary store, databases as index:** KTX's core design choice is that wiki pages (Markdown) and semantic sources (YAML) are the authoritative artifacts, living in git-tracked files. SQLite (or Postgres) maintains search indexes and state, but these are derived from files — they can be rebuilt from `reindex`. This is the inverse of most tools, which keep the database as source of truth and export files as artifacts.

**Python for the semantic compiler, TypeScript for the harness:** The semantic layer — the part that needs AST-level SQL parsing, join graph analysis, and query planning — is in Python using sqlglot. The CLI, MCP server, ingest pipeline, and memory system are in TypeScript. The boundary is crossed via subprocess invocation. This is a pragmatic split: Python has the stronger SQL parsing ecosystem, TypeScript has the stronger CLI/network/server ecosystem.

**Controlled LLM autonomy with worktree isolation:** Unlike tools that trust LLM output directly, KTX wraps every LLM-powered write in three defenses: worktree isolation (writes are invisible until merged), skill-based tool gating (agents can only use tools their loaded skill exposes), and pre-merge validation (failed writes are reverted). The agent has autonomy within the sandbox but the sandbox has deterministic gates.

**Context layer, not agent memory:** KTX is explicitly positioned as a "context layer" rather than "agent memory." The distinction: context is a shared, reviewed, durable surface that multiple agents use; memory is what a single agent remembers. This maps to the team wiki vs. personal notebook distinction. The files are reviewed by humans, versioned by git, and serve as the organization's canonical definitions — not one agent's ephemeral notes.

**No MCP overhead — npx one command:** KTX's MCP server is packaged as a single `npx @kaelio/ktx mcp stdio` command. No separate server process, no API keys to manage (it reads the user's existing LLM provider config), no Docker. The trade-off: it requires a `ktx.yaml` project to be set up first, which means there's a setup step most MCP tools don't require.

**Complete validation pipeline:** The semantic layer engine has seven validation checks: orphan join targets, invalid grain, join column consistency, SQL join coverage, disconnected components, column existence, and visibility rules. A `validate()` method returns a structured report. This is unusually thorough for a YAML-defined semantic layer — most tools (dbt, LookML, Cube) leave validation to runtime errors.

## Comparison Notes

**vs. Metrics SQL:** Both define measures in YAML and compile to SQL. KTX's approach is more automated — it ingests from dbt, Looker, Metabase, and query history to bootstrap definitions. Metrics SQL requires hand-authoring. KTX also adds wiki context (business definitions around the metrics) and a review workflow. Metrics SQL has the cleaner SQL interface for humans; KTX has richer agent-native tooling.

**vs. dbt Semantic Layer / MetricFlow:** Both use YAML-defined semantic models. KTX can import from MetricFlow. The key difference: KTX runs entirely locally (no cloud dependency), adds wiki context alongside metrics, and provides MCP tools for agents. dbt's semantic layer requires dbt Cloud for the semantic layer API. KTX's Python compiler is open-source and self-contained.

**vs. Wuphf / LLM Wiki:** Similar git+markdown wiki substrate. KTX goes further by adding a semantic layer (YAML-defined measures and joins with SQL compilation), a configurable ingest pipeline from warehouse/BI tools, and a pre-merge validation gate. Wuphf is pure wiki; KTX is wiki + executable metrics.

**vs. Claude-Mem / Stash:** These are agent memory systems. KTX is a context layer — shared, reviewed, durable definitions rather than per-agent memory. The memory agent inside KTX does capture knowledge from conversations, but the output is version-controlled files humans review, not opaque embeddings or vector stores.

**vs. Cube:** Both provide a semantic layer with YAML definitions and SQL generation. Cube is a server (REST/GraphQL/SQL API) that you deploy. KTX is a CLI + MCP server that runs locally in the agent's process. Cube targets dashboards and applications; KTX targets AI agents doing ad-hoc analytics.

**vs. DAB (Microsoft Data API Builder):** DAB exposes databases via REST/GraphQL/MCP. KTX goes a layer up — it ingests metadata, builds a semantic layer, then exposes it via MCP. DAB gives you raw SQL access; KTX gives you governed metrics with business context.

## Tags
#tool #project #agents #database #semantic-layer #context-engine #mcp
