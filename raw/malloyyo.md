---
url: https://github.com/malloydata/malloyyo
title: Malloyyo
author: Malloy Foundation (malloydata)
date_fetched: 2026-07-18
date_published: 2025
---

# Malloyyo — Repo Analysis

A Next.js 16 monorepo (~9.5K lines of TypeScript across two packages + server) that turns Malloy semantic models into MCP endpoints for AI agents, with a companion web UI (ltool) and CLI for publishing models.

## Architecture

### Three-tier MCP Engine (`packages/mcp-engine/`)

The engine is the intellectual core — a vendor-neutral, zero-dependency library with three layers:

- **Layer 1 — Types** (`types.ts`, 543 lines): The wire contract. `ModelInfo`, `SourceInfo`, `FieldGroups` (dimensions/measures/views/joins), result envelopes (`CompileResult`, `RunResult`, `QueryValidationResult`), and the `HOST_ONLY` channel for data the host needs but the agent must never see (e.g., SQL on `execute:true`). Wire keys are `snake_case`; TS-side option bags use `camelCase`.

- **Layer 2 — Helpers** (`walker.ts` 656 lines, `run.ts` 169 lines, `restricted.ts` 106 lines, `project.ts` 425 lines, `select.ts` 90 lines): Pure functions over an injected `malloy.Runtime`. Never throw on user-input failure — everything actionable returns as `problems[]` with codes, positions, and `help_topic` pointers. The walker (`compile()`) is the critical path: traverses Malloy's Model API, reshapes into the engine's `ModelInfo` wire format, handles anonymous join targets (via `referenceSourceID` / `referencedSource()` experimental API), source text slicing, and canonical-form detection. `buildSourceDescribe()` in `project.ts` assembles the structured describe_source surface (columns-vs-joins split, `fans_out` cardinality, `quoted_path` for backtick-quoted segments).

- **Layer 3 — Turnkey surfaces** (`surfaces/explore.ts` 549 lines, `surfaces/develop.ts` 172 lines): Ready-made tool sets as data + handlers. `exploreSurface(host)` produces `list_sources`, `describe_source`, `query`, `yo_help`. `developSurface(host)` produces `compile_file`, `compile`, `query`, `prettify`, `yo_help`. Construction is zero-I/O — pure closures. The `ExploreHost` contract requires only `withModel(ref, fn)` (O(1) resolution) and optional `list()` (advisory, paged, no completeness guarantee).

### Host layer (`src/lib/mcp-host.ts`, 402 lines)

The host wires the engine to the real world. It:
- Builds an `ExploreHost` backed by DB model resolution and connection pooling
- Adds instance tagging (`[Malloyyo]` prefix on tool descriptions)
- Injects `model` param on every tool schema (self-reported, fallback to `x-author-model` header)
- Makes `question` required on `query` (host policy, not engine)
- Records every tool call to `history` via `recordHistory()`
- Decorates successful runs with share links (`ltool_link`)
- Handles `open_share_link` (host-only tool, not in the engine)
- Applies client-profile-based result formatting (compact markdown tables for clients that hide structuredContent)

### Connection pooling and caching (`src/lib/malloy.ts`, 689 lines)

A sophisticated runtime management system designed for Vercel serverless:

- **RuntimePool**: Per-model-version connection pool (default 5 connections, up to 20). Configurable via `malloy-config.json` `poolSize`. Idle connections are reused; saturated pools queue waiters. LRU-bounded at 16 concurrent pools.

- **ModelDef cache**: Two-level (L1 in-memory LRU, L2 in Postgres `compiled_model_def` bytea). Stores gzip(JSON(Model._modelDef)) so a cold instance rehydrates without paying the per-source schema-fetch compile (worldcup: ~8s → ~0ms). Write-through on cold miss. Immutable per model version. Currently turned off in prod due to a keying bug (entry-path collision).

- **Lazy DB backend registration**: `ensureConnectionTypes()` dynamically imports BigQuery/Postgres/Snowflake/Trino/MySQL connectors so a broken native dependency can't crash the module at load time.

- **DuckDB spill dir fix**: On Vercel (read-only CWD), sets `home_directory` and `temp_directory` to `/tmp` so large queries don't fail with "Read-only file system."

- **Per-instance diagnostics**: `INSTANCE_ID` (4 hex bytes), `INSTANCE_HOST`, `INSTANCE_IP`, `instanceReqN`, and structured timing logs (acquire → compile → run → serialize) for cold-start vs warm-reuse diagnostics.

### DB schema (`src/db/schema.ts`, 427 lines)

Drizzle ORM schema for Neon Postgres. Key tables:

- `users` — with `slug` for `/mcp/<slug>` paths, `isAdmin`
- `datasets` — status lifecycle (pending → ingesting → introspecting → modeling → ready → failed), GitHub repo config, publish provenance
- `malloy_models` — versioned, with git provenance (sha, branch, dirty flag), `compiled_model_def` bytea for cold-start rehydration
- `malloy_model_files` — one row per file in a multi-file model
- `malloy_artifacts` — dashboard artifacts (manifest + Dashboard.tsx source)
- `history` — every MCP tool call and ltool run, with time-window sessionization (30-min gap = new session), shareable slugs
- `saved_queries` — durable, favoritable queries promoted from history
- `favorites` — many-to-many user-saved_query
- `oauth_clients`, `oauth_authorization_codes`, `oauth_access_tokens`, `oauth_refresh_tokens` — full OAuth 2.1 provider (PKCE S256 mandatory, refresh token rotation, theft canary via `replacedById`)

### CLI (`packages/cli/`)

A standalone `malloyyo` CLI (Commander.js) with commands: `login`, `logout`, `publish`, `status`, `lint`, `mcp` (stdio server for Claude), `init` (scaffold `.mcp.json` + `index.malloy`), `author`/`test` (launch Claude), `dashboard dev` (preview dashboards locally).

### Auth

- **User auth**: NextAuth v5 with Google OAuth (Okta optional), DrizzleAdapter, email allow-list, slug assignment on first sign-in with retry on collision.
- **MCP auth**: Full OAuth 2.1 provider — dynamic client registration, authorization code flow with PKCE S256, bearer access tokens (24h TTL), refresh token rotation (90d TTL), hash-stored tokens.

## Key Techniques

### Restricted query governance

The explore surface routes ALL queries through `loadRestrictedQuery()` — no `import`, no `given:` declarations, no `connection.table/sql`, no raw-SQL forms, no `##!` flags. The model author controls exactly what an AI can query. The `restricted.ts` functions are deliberately named so no implementer can accidentally route explore input through the open run.

### SQL withholding on execute:true

A subtle security/privacy design: when `execute:true`, the generated SQL is parked on the `HOST_ONLY` channel — `toContent()` drops it, so the agent never sees SQL on an executed run. SQL only rides `execute:false` (the confirmatory-inspect channel). The host records it for auditing.

### Source-centric resolution with model_ref backstop

The explore surface is source-centric (the user thinks in terms of "what can I query"), but addressing is model-keyed underneath. `list_sources` discovers the catalog; `describe_source`/`query` take a bare `source`, resolving it against the catalog when unique (ambiguous → "pick one, here are the model_refs"). An explicit `model_ref` enables back-door access to non-exported sources.

### Field-not-found recovery with sibling detection

`queryFieldFix()` in `surfaces/explore.ts` detects when the "not defined" name is actually defined IN the query body — a sibling-field reference that Malloy doesn't allow. Instead of misleading the agent into hunting for a missing field, it points at the `extend:` composition fix. This is the kind of nudge that makes AI agents dramatically more effective.

### Canonical-form signal vs. echoing prettified text

`CompileResult.formatted` is a single boolean: "would prettify change this?" Costs one token instead of echoing the full prettified source (which would double every compile's token cost). Only emitted when `readSource` is available (develop path only).

### Problems, not protocol errors

Every helper/tool failure the agent can act on comes back as `problems[]` with stable codes, positions, and `help_topic` pointers. `isError` is reserved for the host (auth, unknown tool). The engine never uses `isError` for compile/run failures — the agent reads `ok: false` and `problems[]`, not error codes.

### Annotation dual-channel promotion

Malloy annotations split into two promoted channels: `#"` → `description` (human documentation), `#(agent)` → `instructions` (agent-facing guidance). Both are promoted from annotation lists to first-class fields, then annotations are stripped of those routes so text isn't shipped twice.

### Result byte budgeting with spill

Results are budgeted by bytes (not token estimates — deterministic, no tokenizer dep), dropping whole rows from the end until under budget. A single row over budget → zero rows + SQL + `row_count` + hint. No cursors — a spill link is strictly better for stateless hosts.

### Client-profile-aware result formatting

The host detects the calling client (user-agent → profile) and adapts: clients that file away structuredContent and cite only a snippet (ChatGPT) get compact markdown tables in `content` and NO `structuredContent` — so rows can't be hidden behind a citation.

## Design Decisions

### Model-first engine, source-centric surface

The engine is model-keyed (a model is the compilation unit), but the explore surface is source-centric (matching how users think). This is documented as a deliberate reversal from the original source-first design, which would have leaked the `index.malloy` package convention into the contract and caused shadow-collisions under `findBySource`'s newest-dataset-wins resolution.

### @malloydata/malloy as peerDependency

The engine declares `@malloydata/malloy` as a peerDep, never bundled. Two copies would break `instanceof MalloyError` checks, degrading every compile error to `internal-error`. This forces a single shared instance across the monorepo.

### Zero-I/O construction

Surface instantiation is pure closures — no scanning, listing, or compiling until a tool call. This makes per-request principal-bound hosts cheap (build a host per user per request), keeps lifecycle out of the engine, and means tree-size irrelevance (a 100-file model takes no more construction time than a 1-file model).

### One query tool with execute:false

Unlike malloy-cli's split compile/run tools, the engine has one `query` tool with `execute:false` for validation. This matches what's proven against claude.ai tool-search ranking — fewer tools means better ranking.

### Lean descriptions, policy in instructions

Tool descriptions are one or two lines owning distinct concept words. Behavioral policy lives in server instructions. Long descriptions dilute the client's tool-search ranking.

### yo_help as the single reachability channel

Every piece of guidance must be reachable through `yo_help` — the only channel every host has (the hosted endpoint has no prompts/resources capability). Topics are bundled markdown, the language reference auto-split on `##` headings, and error codes map to topic names.

### Annotation routes as structured dual-channel

Rather than dumping all annotations as an opaque list, the engine promotes `#"` to `description` and `#(agent)` to `instructions`, then strips those routes from `annotations[]`. Other routes (render tags, app-staked routes) are kept — no way to know what a client wants from them.

### Quoting and name safety

Every name written in Malloy carries `must_quote: true` when it needs backticks. Maps are built on null-prototype objects so reserved names (`__proto__`, `constructor`, `hasOwnProperty`) are ordinary data keys. Join paths have both a clean key (for lookup) and `quoted_path` (for writing).

## Comparison Notes

Unlike **DAB (Microsoft's Data API Builder)** which auto-generates REST/GraphQL/MCP from any database schema, Malloyyo requires a human-authored (or Claude-authored) semantic model. This is a feature: the model is the governance boundary. The AI can only query what's in the model.

Unlike **Nubase** which provides a full backend (database-per-tenant, auth, storage, MCP bridge), Malloyyo is deliberately thin — it serves models, nothing else. It doesn't create databases; it connects to yours.

Unlike raw **MCP servers** that expose fixed tools, Malloyyo's tools are model-driven — `list_sources` and `describe_source` output changes based on which models are published. The surface is data, not code.

The **restricted query** governance is a genuinely novel security model for AI data access: the model author defines the semantic surface, `loadRestrictedQuery()` enforces it, and the AI composes queries against the model rather than writing SQL. This means: no invented columns, no wrong joins, no fan-out double-counts, no access outside the model.

The **connection pooling with ModelDef caching** design — per-model-version pools with shared CacheManager, write-through compiled-model persistence, LRU eviction — is unusually sophisticated for a Next.js app. It treats serverless cold starts as a first-class performance problem with a two-level cache architecture, per-instance diagnostics, and pool-size configurability from the model config itself.
