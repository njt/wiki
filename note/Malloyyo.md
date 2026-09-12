# Malloyyo

A thin web service that turns [Malloy](https://malloydata.dev) semantic models into MCP endpoints for AI agents, so any LLM can run consistent, governed analytical queries against your data. The AI composes queries against a model (measures, dimensions, joins defined once, correctly) instead of writing SQL from scratch — so the same question always yields the same numbers, with no wrong joins or invented columns.

---

## Architecture

Malloyyo is a **Next.js 16 monorepo** (~9,500 lines of TypeScript) with three layers:

### MCP Engine (`packages/mcp-engine/`)

A vendor-neutral, zero-dependency library that is the intellectual core. Three sub-layers:

- **Types** (`types.ts`): The wire contract — `ModelInfo`, `SourceInfo`, four field groups (dimensions/measures/views/joins), result envelopes, and the `HOST_ONLY` channel for data the host needs but the agent never sees (SQL on `execute:true`).
- **Helpers**: Pure functions over an injected `malloy.Runtime`. The **walker** (`walker.ts`) compiles models into the engine's structured `ModelInfo` format, handling annotation dual-channel promotion (`#"` → description, `#(agent)` → instructions), anonymous join targets, source text slicing, and canonical-form detection. `project.ts` assembles the structured describe_source surface with columns-vs-joins split and `fans_out` cardinality. `restricted.ts` enforces `loadRestrictedQuery()` — no import, no raw SQL, no `##!` flags.
- **Turnkey surfaces**: Ready-made tool sets as data + handlers. `exploreSurface(host)` → `list_sources`, `describe_source`, `query`, `yo_help`. `developSurface(host)` → `compile_file`, `compile`, `query`, `prettify`, `yo_help`. Construction is **zero-I/O** — pure closures, no scanning or compiling until a tool is called.

### Host layer (`src/lib/mcp-host.ts`)

Wires the engine to the real world: DB-backed model resolution, connection pooling, instance tagging (`[Malloyyo]` prefix on tool descriptions), per-call history recording, share-link decoration, and the `open_share_link` tool. Injects `model` param on every tool schema for model attribution.

### Server runtime (`src/lib/malloy.ts`)

A sophisticated connection management system for Vercel serverless:

- **RuntimePool**: Per-model-version connection pools (default 5, configurable up to 20) with idle reuse, waiter queues, LRU eviction at 16 pools
- **ModelDef cache**: Two-level (L1 in-memory, L2 Postgres bytea). Stores `gzip(JSON(Model._modelDef))` so a cold instance rehydrates in ~0ms instead of paying the per-source schema-fetch compile (~8s for worldcup). Write-through on cold miss
- **Lazy backend registration**: Dynamically imports BigQuery/Postgres/Snowflake/Trino/MySQL connectors so a broken native dependency can't crash at load time
- **Per-instance diagnostics**: Structured timing logs (acquire → compile → run → serialize) with instance identity for cold-start vs warm-reuse analysis

### Data model (`src/db/schema.ts`)

Neon Postgres via Drizzle ORM. Key tables: `datasets` (status lifecycle, GitHub config), `malloy_models` (versioned, git provenance, compiled_model_def bytea), `malloy_model_files` (multi-file models), `malloy_artifacts` (dashboard artifacts), `history` (every tool call, time-window sessionized), `saved_queries` (durable, favoritable), plus full OAuth 2.1 provider tables.

### CLI (`packages/cli/`)

Standalone `malloyyo` CLI: `login`, `publish` (bundle + push model files), `status`, `lint` (validate dashboards), `mcp` (stdio server for Claude), `init` (scaffold `.mcp.json`), `author`/`test` (launch Claude in develop/explore mode), `dashboard dev` (local preview).

## Key Techniques

### Restricted query governance

The explore surface routes ALL queries through `loadRestrictedQuery()` — the model author controls exactly what an AI can query. No import, no raw SQL, no `##!` flags. This is enforced structurally, not by prompt.

### SQL withholding on execute:true

The generated SQL is parked on the `HOST_ONLY` channel — the agent never sees SQL on an executed run. SQL only rides `execute:false` (the confirmatory-inspect channel). The host records it for auditing.

### Source-centric resolution with model_ref backstop

The explore surface is source-centric (matching how users think: "what can I query?"), but addressing stays model-keyed underneath. A bare source resolves against the catalog when unique; ambiguous → "pick one, here are the model_refs."

### Field-not-found recovery with sibling detection

`queryFieldFix()` detects when a "not defined" name is actually defined IN the query body — a sibling-field reference that Malloy doesn't allow. Instead of misleading the agent into hunting for a missing field, it points at the `extend:` composition fix.

### Annotation dual-channel promotion

`#"` → `description` (human), `#(agent)` → `instructions` (agent). Promoted to first-class fields, then stripped from annotations so text isn't shipped twice.

### Problems-as-data, not protocol errors

Every helper/tool failure returns as `problems[]` with stable codes, positions, and `help_topic` pointers. `isError` is reserved for the host (auth, unknown tool).

### Client-profile-aware result formatting

The host detects the calling client and adapts: clients that hide structuredContent get compact markdown tables — rows can't be hidden behind a citation.

## Design Decisions

- **Model-first engine, source-centric surface**: Engine addressing is model-keyed (a model is the compilation unit), but the explore surface flattens to source-first. Documented as a deliberate reversal from the original design that would have caused shadow-collisions.
- **`@malloydata/malloy` as peerDependency**: Never bundled — two copies would break `instanceof MalloyError`, degrading every compile error to `internal-error`.
- **Zero-I/O construction**: Surface instantiation is pure closures — per-request principal-bound hosts are cheap, lifecycle stays out of the engine.
- **One `query` tool with `execute:false`**: Fewer tools → better claude.ai tool-search ranking. Validate-and-inspect is one call, not two.
- **Lean descriptions, policy in instructions**: Tool descriptions are one or two lines. Behavioral policy lives in server instructions where it doesn't dilute search ranking.
- **`yo_help` as single reachability channel**: Every guidance topic reachable through one tool — critical because the hosted endpoint has no prompts/resources capability.

## Comparison

Unlike **[[DAB]]** which auto-generates endpoints from any database schema, Malloyyo requires a semantic model — which IS the governance boundary. Unlike **[[Nubase]]** which provides a full backend platform, Malloyyo is deliberately thin: it serves models, nothing else. Unlike raw [[Building Agents for Production Systems with MCP|MCP servers]] with fixed tools, Malloyyo's tools are model-driven — the surface changes based on what's published.

The restricted query governance is genuinely novel: the model defines the semantic surface, the engine enforces it structurally, and the AI composes against the model rather than writing SQL — no invented columns, no wrong joins, no fan-out double-counts. The connection pooling with two-level ModelDef caching is unusually sophisticated for a Next.js app, treating serverless cold starts as a first-class problem.

---
*Sources: [[raw/malloyyo]]*
*Last updated: 2026-07-18*
