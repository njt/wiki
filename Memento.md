# Memento

A local-first knowledge layer that turns years of email into source-attributed living documents. Built on top of [`msgvault`](https://www.msgvault.io/) by Jesse Vincent / Prime Radiant, Memento runs as a single Go binary embedding a statically exported Next.js frontend — no Node.js at runtime, no cloud dependency. It organizes long-term email history into five memory surfaces (People, Projects, Newsletters, Concepts, and Ask Memento chat), each with its own purpose-built agent workflow. The core insight: distinct memory surfaces need distinct search strategies and evidence assembly — a project timeline, a concept overview, and a relationship wiki are fundamentally different retrieval and authoring tasks.

Tags: #tool #project #agents #local-first #email #knowledge-management #memory

---

## Architecture

Memento is a single Go binary (`memento`) that embeds the statically exported Next.js frontend and serves both UI and API on `127.0.0.1:8787`. The msgvault SQLite archive is read-only; Memento writes only `memento_*` tables.

The backend is structured around dimension-specific packages (`backend/internal/person`, `backend/internal/project`, `backend/internal/concept`, `backend/internal/newsletter`, `backend/internal/people`, `backend/internal/social`) that do deterministic extraction, plus an agent runtime (`backend/internal/agentrunner`) that handles LLM generation. All five agents (collector, project_compile, concept_compile, person_enrich, dashboard) run in Go and stream over same-origin SSE — the browser calls the Go API directly with no proxy layer.

The data model uses materialized rollup tables (`memento_people_report`, `memento_projects_report`, `memento_newsletters_report`, `memento_concepts_report`) rebuilt by `Refresh*` functions in transactions. Index pages read these rollups; no N+1 archive joins on the request path. This is the same rollup pattern used by production analytics databases — compute once, read many.

Agent durability is first-class: runs persist in `memento_agent_session`, steps in `memento_agent_loop`, tool calls in `memento_agent_tool_call`, and SSE events in `memento_agent_event`. Browser disconnects don't cancel runs; events replay via `after_seq` / `Last-Event-ID`.

Key files:
- `backend/internal/agentrunner/runner.go:804` — agent loop, tool dispatch, SSE emit, usage logging
- `backend/internal/agentrunner/providers.go:1080` — Gemini Interactions API + OpenAI-compatible streaming
- `backend/internal/server/agent_runs.go:751` — per-agent `RunSpec` assembly, completion contract wiring
- `backend/internal/server/agent_prompts.go:445` — system prompts for all five agents
- `backend/internal/server/agent_tool_registry.go:404` — 36-tool registry with lock keys and handlers
- `backend/internal/person/matcher.go:459` — deterministic email-to-person resolution
- `backend/internal/person/merge.go:860` — person dedup with social-graph signatures
- `src/components/agent/useAgentStream.ts:174` — frontend SSE consumer

## Key Techniques

**Deterministic-first pipeline.** Before any LLM call, the Go backend resolves canonical people from raw email participants (normalization + cluster merging + rule-based classification), detects newsletter sources (domain/local-part/unsubscribe heuristics), and assembles evidence bundles from attached message IDs. The LLM generates prose from pre-assembled evidence, never rediscovering the archive shape from scratch. This is the opposite of "throw everything at the model and hope" — it's more like a compiler's analysis phase before code generation.

**Completion contracts with bounded repair.** Each agent run must satisfy `RequiredOutcomes` — a list of `OutcomeRequirement` structs specifying `{ToolName, ArgEquals, AnyOfGroup, RequiredCount}`. The `outcomeTracker` counts successful tool calls; if required writes are missing when the model finishes, one bounded repair turn fires with a prompt that says "call only these missing tools using already-loaded evidence." If repair fails, the run fails. This prevents the common agent failure mode of "finished talking but wrote nothing."

Project compile requires `write_section` for `summary`, `phases`, `friction_points`, `current_understanding`. Concept compile requires `write_concept_section` for `scope_summary`, `distilled_insights`, `evolving_understanding`. Person enrich requires at least one `write_facet`, one attribute decision (`write_person_attribute` or `record_no_person_attributes`), and narrative section writes for non-user-edited sections.

**Tool locking with parallel read-only dispatch.** Read-only tools (`fts_search`, `vector_search`, `get_message_batch`, etc.) run concurrently within a model-emitted batch — capped at 8 when `MEMENTO_MSGVAULT_API_URL` is set (HTTP API path), otherwise 4 (direct SQLite). Mutating tools serialize via lock keys: `draft:{id}` for collector, `{agent_type}:{entity_id}` for others, `decision:{agent_type}:{entity_id}` for `propose_backfill`. This is a pragmatic design — mutating tools are rare compared to reads, so the overhead of fine-grained locking isn't worth it.

**Materialized rollups.** Every dimension index reads from a `memento_*_report` table rebuilt atomically in a transaction. `RefreshPeopleReport` pulls from `memento_people_candidates` (pre-classified as `candidate`/`weak_signal`/`excluded`), joining in aliases, recent timeline items, and top shared-thread correspondents. This means the People page loads with a single query, not hundreds of joins across `messages` + `participants` + `message_recipients`.

**Human-in-the-loop via propose_backfill.** The collector agent's `propose_backfill` tool is a `ToolHumanWaiting` — it emits a `proposed_backfill` SSE event, sets the run to `waiting_for_user`, polls `memento_agent_decision` every second until the user accepts/skips or 90 seconds expire, then resumes the loop. The decision is durable (persisted to `memento_agent_decision`) and the UI posts decisions to `POST /api/drafts/[id]/backfill`. This is a key design choice: the agent suggests, the human decides, the decision is recorded — no silent auto-folding of backfill candidates into bundles.

**Context budget awareness.** The `context_status` tool reads persisted token usage from the current run and computes a `budget_level` (normal / watch / low / critical) using `MEMENTO_AGENT_CONTEXT_LIMIT_TOKENS` (default 128000) as the denominator. Prompts instruct agents to call this before broad expansion and switch to compact tools at `watch` or worse. The implementation is honest about its limitations: "no trimming or summarization of completed steps mid-run" is documented as a known structural problem.

## Design Decisions

**Single binary, no Node.js at runtime.** The `pnpm package` script runs `next build` (static export), stages output to `backend/internal/webui/dist`, and compiles the Go binary with the UI embedded. End-user entry point is `memento app`. This was recorded in `DECISIONS.md` on June 13, 2026, as a deliberate trade: simpler deployment and zero runtime dependencies, at the cost of losing Next.js SSR/dynamic routes. The Go server rewrites dynamic `[slug]` URLs to the placeholder-param shell files, and clients read the slug from `window.location`.

**Demo mode isolation.** Demo data lives in a separate `data/memento-demo.db` — it must not seed rows into the user's real msgvault archive. The demo must work without a real LLM key, Ollama, or a msgvault vector index. This makes the first-run experience zero-config while maintaining a hard boundary between synthetic and real data.

**msgvault as read-only substrate.** Memento intentionally does not implement its own mail search engine. It uses msgvault's hybrid search (FTS + vector), falls back to FTS for keyword/operator queries, and prefers the `msgvault serve` HTTP API for concurrent reads. This is a scope decision: Memento builds memory and narrative, not mail infrastructure.

**User edits are authoritative.** `write_section`, `write_facet`, `write_person_section`, and `write_concept_section` all protect user-edited rows. Skipped writes don't satisfy completion requirements, so a fully user-edited narrative won't cause the agent to fail — it just won't produce new output for those sections. Generated sections carry `edited_by` tracking so the backend knows which rows to protect.

**Per-agent model override.** `MEMENTO_COLLECTOR_MODEL`, `MEMENTO_PROJECT_MODEL`, `MEMENTO_CONCEPT_MODEL`, `MEMENTO_PERSON_MODEL`, `MEMENTO_MEMENTO_MODEL` allow different models per agent. A user could run collector on a cheap fast model and project compile on a reasoning model. This is production-grade thinking — not "one model to rule them all" but "right model for each task."

## Comparison Notes

**vs. Mnemo**: Both are local-first memory layers, but Memento is email-specific (msgvault archive) while Mnemo is conversation-agnostic. Mnemo builds a graph with no embeddings (SQLite LIKE + BFS); Memento uses msgvault's existing vector search. Mnemo is a sidecar service; Memento's agents run in-process with the archive.

**vs. Rowboat**: Both build knowledge from email, but Rowboat builds an Obsidian-compatible knowledge graph and acts as a general coworker. Memento is purpose-built for five specific memory surfaces with hardcoded agent workflows. Rowboat treats the knowledge graph as the product; Memento treats the source-attributed living document as the product.

**vs. Sawtooth Memory**: Sawtooth is async non-blocking hierarchical middleware (L0 system / L1 working / L1.5 entity ledger / L2 archival). Memento's context management is simpler — monotonic accumulation with tool-level mitigations. Sawtooth's compression and dual-extraction approach would directly address Memento's open problem of mid-run step trimming.

**vs. Personal Agents frameworks** (Hermes, clawdBot, Pi): Memento is not a general-purpose agent platform. The five agents are hardcoded per dimension with specific tools and completion contracts. There's no user-defined agent creation, no MCP server integration, and no plugin system. This is a focused product, not an extensible platform.

**vs. generic RAG systems**: Memento's "deterministic extraction before LLM generation" is the opposite of search-at-prompt-time RAG. The evidence bundle is assembled deterministically from attached message IDs; the LLM writes prose from a known evidence surface. This gives source attribution (every claim traces to a message ID) at the cost of coverage (messages must be explicitly attached to a project/concept).

---

*Sources: [[raw/memento]], README.md, DECISIONS.md, docs/agent-runtime.md, docs/agent-loop-and-prompts.md, docs/agent-context-management.md, docs/deterministic-extraction.md, backend/internal/agentrunner/runner.go, backend/internal/server/agent_runs.go*
*Last updated: 2026-06-15*
