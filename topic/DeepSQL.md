# DeepSQL

DeepSQL is a self-hosted database agent for PostgreSQL and MySQL: ask questions in plain English, and it drafts, validates, and executes read-only SQL to answer BI questions, diagnose slow queries, recommend indexes, and watch your schema. Its one-line bet is *correctness over autonomy* — the LLM drafts, but deterministic machinery (schema whitelists, `EXPLAIN` validation, database-enforced read-only sessions) owns every safety-critical decision. That posture makes it the closest thing in this wiki to a working, code-complete answer to [[Text-to-SQL in the Real World]]'s finding that raw text-to-SQL fails on real schemas.

---

## Architecture

A monorepo with four cooperating components, all self-hosted:

- **`backend/`** — Spring Boot 4 on Java 25, ~221K lines of Java under `com.dbaagent`. This is the brain: REST API, credential vault, RAG, the query pipeline, and both agent layers.
- **`src/`** — React 19 + Vite + Tailwind + Zustand + TanStack Query frontend.
- **`agent/`** — the **DeepSQL Agent**, a heavily customized [Nous Hermes Agent](https://hermes-agent.nousresearch.com/) runtime. It carries a DBA persona (`agent/SOUL.md`) and six procedural skills (`agent/skills/*/SKILL.md`: `bi-query`, `schema-exploration`, `index-advisor`, `slow-query-optimize`, `workload-analysis`, `dashboard-design`). DeepSQL doesn't fork the runtime — it ships persona + skills + skins as an overlay.
- **`mcp/`** — `@deepsql/mcp`, a Node.js package shipping three things in one binary: the `deepsql` CLI, a `deepsql agent` TUI, and a stdio MCP server exposing 44 tools to Claude Code/Cursor/Codex.

The signature architectural move is the **provider registry pattern**, applied twice. `provider/DatabaseProviderRegistry.java` auto-discovers `DatabaseDialect` beans; each dialect is a composite of ~10 capability interfaces (`ConnectionProvider`, `IntrospectionProvider`, `SlowQueryProvider`, `ExplainPlanProvider`, `QueryExecutionProvider`, …). `AGENTS.md` states the anti-pattern explicitly: *never* write `if/else` or `switch` on database type in a service. The LLM side mirrors it — `llm/LlmProviderRegistry.java` indexes chat and embedding providers separately, because Anthropic has no embeddings API and a single index would need exactly the provider-type switch the codebase forbids.

There are **two agent layers**, which is unusual:

1. **`service/agent/AgentOrchestrator.java`** — an in-house orchestration loop. `LlmOrchestrationService` decides intent, thread scope, and a `SourcePlan` (an ordered list of "source families" that must be scouted, e.g. `key_column_analysis`, `slow_query_history`, `workload_profile`). It then loops: execute a tool → `verifyProgress` → clarify or continue, up to `max-steps` (8), and won't stop before the source plan is satisfied.
2. **The DeepSQL Agent** (Hermes runtime) — persona + skills over the MCP tools, with host-affecting toolsets (terminal/file/browser) disabled.

## Key Techniques

**Read-only by default, enforced by the database.** `QueryExecutorService.java` doesn't trust its own SQL classifier. When the caller may not mutate, it calls `connection.setReadOnly(true)` and *throws* if the driver refuses — the comment is blunt: "Classification is a parser heuristic, so a read-only session is what keeps a future gap from becoming data loss — PostgreSQL then refuses the write itself." Row caps use `stmt.setMaxRows(limit + 1)` at the driver level, so a `LIMIT` hidden inside a CTE, comment, or string literal can't defeat them; the `+1` detects truncation. Mutation flows through a `requiresConfirmation` → `confirmMutation: true` round-trip, admin-only.

**One LLM provider, dispatched on endpoint shape.** There is a single `openai` provider id. `llm/openai/OpenAiEndpoints.java` decides authentication by *host* — `.azure.com`/`.azure.us`/`.azure.cn`/`.azure-api.net` get an `api-key` header, everything else (api.openai.com, vLLM, Ollama, LM Studio, LiteLLM, TGI) gets `Authorization: Bearer`. `ResponsesApiChatModel` is a hand-rolled Spring AI `ChatModel` speaking both the Responses API and Chat Completions; `use-responses-api=auto` picks by model-name prefix (`gpt-5`, `o1`, `o3`, `o4`, `codex` → Responses). Retries use **jittered backoff** (`BACKOFF_JITTER_FRACTION = 0.5`) because the agent fires several model calls in parallel and deterministic backoff makes retries collide into the same burst that rejected them.

**Credential resolution with no default tier.** `llm/LlmConfigResolver.java` resolves from exactly two places — the encrypted DB bundle, then environment — and deliberately omits a properties-file default, because "a credential default in a properties file is how the production Azure key reached git history." That's scar tissue turned into a design invariant.

**Hallucination filtering in the query pipeline.** `service/pipeline/QueryGenerationPipeline.java` runs BI questions through: (1) embedding-based query-history match (threshold 0.92) with always-on `EXPLAIN` validation; (2) LLM table/column resolution whose JSON is post-processed in `parseResolvedContext` to drop *every table and column that isn't literally in the schema*; (3) column-value fetch (capped at 50 values × 10 columns, 3s timeout); (4) SQL generation; (5) `EXPLAIN` validation. A "single-table fast path" skips the LLM when exactly one table name appears in the question, and a "source-of-truth promotion" heuristic steers the resolver off derived/aggregate tables toward raw business-measure tables.

**The brain is a governed, shared semantic memory.** A single `vector(3072)` column in pgvector (so the embedding model is pinned to `text-embedding-3-large`) backs RAG over business context. `save_brain_note` writes durable facts about the data — column meanings, join paths, business rules, anti-patterns — that ground *every* agent's SQL on that connection. `SOUL.md` routes memory to one of two planes: shared company brain (a note) vs. per-user preference (a DeepSQL skill).

## Design Decisions

**Correctness over autonomy, everywhere.** Every LLM output is fenced by deterministic validation, which is the direct engineering response to Stonebraker's 10% accuracy finding — don't ask the model to be right, ask it to draft and let the machine verify.

**Self-hosted, BYO-model.** No managed SaaS, no bundled provider, no prebuilt images. The trade-off is a real ops burden (bootstrap script, ~4 GB Docker memory, 3 GB JVM heap, JDK 25 + Node 22, builds-from-source) in exchange for a clean story that credentials and data never leave your infra. The one thing DeepSQL can't decide for you is the model — that's step 3 of setup, and the README is explicit about it.

**Scar-tissue-driven API hygiene.** The two-tier credential resolver, the fail-fast collision detection in both registries, and the single `OpenAiEndpoints.isAzure()` definition (whose javadoc notes every past copy "has been subtly wrong in a different way") all encode production incidents as architecture. This is a codebase that has shipped and learned.

**Two agent layers is redundant by design, not accident.** The in-house orchestrator is deterministic-leaning and auditable (an `EvidenceLedger`, a `SourcePlan`, a `VerificationDecision` per step); the Hermes-runtime agent is persona + skills for conversational DBA work. They share the same MCP tools and brain, so DeepSQL gets both a traceable pipeline and a flexible chat surface.

**Latency and cost are the accepted sacrifice.** A single turn can fire many model calls (classify → plan → tool → verify → synthesize), and the JVM needs a 3 GB heap. It's optimized for trustworthy answers on a low-volume, high-consequence workload — the DBA-asks-a-question case, not high-TPS serving.

## Comparison Notes

- **[[Text-to-SQL in the Real World]]** — DeepSQL is the strongest counterexample in this wiki: it concedes the premise (an LLM can't reliably assemble a correct enterprise query) and routes around it with deterministic fencing rather than better prompting. Stonebraker's own Rubicon hint — "don't let the LLM assemble the query" — is close to what DeepSQL actually ships, though DeepSQL still lets the LLM draft and validates after the fact.
- **[[Interdict]]** — same problem (safety between an AI and a SQL database), different interception point. Interdict sits at the wire and measures blast radius with the real Postgres AST; DeepSQL sits in the application and leans on RBAC + a database-enforced read-only session + a two-step admin confirm. Interdict can *undo* writes; DeepSQL mostly *prevents* them.
- **[[AI-Assisted Database Work — The Machine Reads, The Human Decides]]** — that piece draws a boundary: AI fails at *generating* queries but excels at *reading* them. DeepSQL does both — generation (fenced) and reading (schema classification, slow-query analysis, key-column inference) — but keeps the human as the gate on anything that mutates.
- **[[OzBrain]]** — DeepSQL's `save_brain_note` is a domain-scoped instance of the shared-brain pattern: one governed store of business rules and anti-patterns that every agent on a connection reads before writing SQL. The scope is a database, not a person's whole toolchain, but the write-path discipline (durable, audited, shared) rhymes.
- **[[Production RAG in .NET]]** — convergent evolution on the RAG stack: Postgres/pgvector embeddings, hybrid retrieval, a vector-store abstraction (`VECTOR_STORE_TYPE` = `pgvector` or `azure`), and an `EMBEDDING_FAIL_OPEN` knob for graceful degradation.

Tags: #tool #project #database #agents #text-to-sql #rag #security

---
*Sources: [[raw/deepsql]], [[summary/deepsql]]*
*Last updated: 2026-08-22*
