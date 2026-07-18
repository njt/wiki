# Nubase

An open-source, AI-native backend-and-deploy platform that coding agents drive directly — so a generated app goes live in minutes. It's a self-hosted Supabase alternative designed for multi-project isolation, with eight capability modules (Database, Auth, Storage, Assets, Functions, AI Gateway, Memory, Cron) in a single Spring Boot monolith + Next.js Studio, plus a TypeScript MCP bridge for Claude Code and Codex.

---

## Architecture

Nubase is a **Java 17 Spring Boot 3.2 monolith** with a Next.js Studio frontend and a TypeScript MCP bridge CLI (`nubase_cli`). The core architectural bet is **database-per-tenant isolation** — each project gets its own physical PostgreSQL database — managed through Hibernate's multi-tenancy SPI rather than schema-per-tenant switches.

### Two-layer database model

- **Metadata database** (control plane): platform users, project configs, encrypted credentials, JWT secrets, SQL snippets, execution records
- **Project databases** (data plane): one per project, containing `auth.*`, `storage.*`, `mem.*`, and `public.*` application schemas

This is implemented in three key files:
- `src/main/java/ai/nubase/common/multitenancy/DatabaseMultiTenantConnectionProvider.java` — Hibernate `MultiTenantConnectionProvider` that routes JDBC connections via `RoutingDataSource`
- `src/main/java/ai/nubase/common/multitenancy/DatabaseTenantResolver.java` — resolves the current tenant from ThreadLocal `MultiTenancyContext`
- `src/main/java/ai/nubase/common/multitenancy/UnifiedMultiTenancyFilter.java` — 520-line servlet filter: extracts `apikey` JWT, parses `ref` claim (project reference), validates signature, sets ThreadLocal context, and optionally validates an `Authorization: Bearer` user token

### Request routing

```
apikey: <project JWT> + Authorization: Bearer <user JWT>
       ↓
UnifiedMultiTenancyFilter
  1. Parse apikey → ref (app_code) + role (anon/authenticated/service_role)
  2. DatabaseConfigRepository.findByAppCode() → load tenant config
  3. Validate JWT signature against project secret
  4. RoutingDataSource (HikariCP pool per project database)
  5. MultiTenancyContext.set() → ThreadLocal
  6. authenticateUser() → Spring Security context
  7. Controller → JDBC (routed via Hibernate multi-tenancy)
```

### PostgREST-compatible REST API

The `/rest/v1/*` API is a **Java reimplementation of PostgREST**, not a proxy. The query pipeline (`ApiRequestParser` → `QueryPlanner` → `QueryExecutor`) supports select, filter, order, limit/offset/range, insert, update, delete, upsert, embedded resources, and JSONB conversion — all generating parameterized SQL against the tenant's database.

### MCP surface

The backend exposes MCP tools via Spring AI's `@Tool` annotation (`DatabaseMcpTools`, `FunctionsMcpTools`, `CronMcpTools`, `MemoryMcpTools`, `AssetsMcpTools`, etc.). For agents connecting remotely (Claude Code, Codex), the TypeScript MCP bridge (`frontend/packages/mcp-bridge/src/index.ts`) provides an stdio MCP server that proxies to the Nubase REST API.

### Edge Functions and deployment

`EdgeFunctionAdminService` manages function deployment with versioning and per-function secrets. Supports local execution or Cloudflare Workers for Platforms dispatching via `cloudflare/functions-dispatcher/worker.js`. The Assets module provides a per-project static CDN at `/assets/v1/**` with Cache-Control/ETag semantics. Scheduled jobs run edge functions or named database functions on crontab schedules with Postgres advisory locking (`FOR UPDATE SKIP LOCKED`).

## Key Techniques

### Memory pipeline (the most innovative module)

The `ai.nubase.mem` package implements a Mem0-style three-LLM-call pipeline:

```text
AddMemoryRequest → MemoryService.add()
  ├── FactExtractionService → LLM: extract facts + entities from conversation
  ├── EmbeddingService (Caffeine cache, SHA-256 keyed) → vectorize each fact
  ├── pgvector searchByVector → top-K nearest for each fact
  ├── MemoryDecisionService → LLM: ADD/UPDATE/DELETE/NONE per fact
  └── Transactional writes + entity linking + append-only history
```

**Search** (`MemoryService.search()`) uses multi-signal fusion via `ScoreFusion.fuse()`:
```text
combined = (semantic_sim + bm25_norm + entity_boost) / max_possible
```
This mirrors mem0's `scoring.py` exactly. Semantic hits are gated by cosine distance threshold, BM25 scores are min-max normalized per result set, and entity boost weight is 0.5. The system over-fetches (`min(topK × 4, 60)`) from both channels before fusion.

### LLM provider abstraction

`LLMProviderRegistry` uses Spring bean discovery for a plugin architecture: chat providers (`openai`, `anthropic`, `generic`) and embedding providers (same three names) each implement simple interfaces. Per-tenant overrides are read from `mem.config` JSON in the project database via `MemConfigResolver`, which falls back to YAML platform defaults. Provider selection happens per-request, not at startup.

### Graceful degradation

- If the decision LLM fails or returns unparseable JSON, `MemoryDecisionService.fallbackAllAdd()` treats every fact as ADD — the system degrades to duplicate-prone but functional behavior
- If the entity extraction LLM call fails during search, the search continues without entity boosts — never fails the query
- If no chat provider is configured, fact extraction and decision steps are entirely skipped (pre-flight `isAvailable()` check)

### Subdomain-based tenant resolution

For apikey-free endpoints (OAuth callback, public storage, SSO ACS, public assets), the tenant is resolved from the request's subdomain (e.g., `app20260108111430zrjprnycyi.example.com`). The code deliberately does NOT fall back to `Referer` header for security — an attacker-controlled Referer could select which tenant's context these endpoints run against.

### Per-project database isolation

Each project gets its own HikariCP connection pool. The `RoutingDataSource` lazily initializes pools on first access and records access timestamps to avoid idle eviction. Hibernate's `MultiTenantConnectionProvider` interface handles the JDBC routing transparently — repositories never know which database they're talking to.

## Design Decisions

**Optimized for**: AI agent UX and multi-project self-hosting. The MCP bridge, memory API, Assets/CDN, and "one command to deploy" ergonomics are designed for coding agents, not human developers. Database-per-tenant gives strong isolation at the cost of more connection pools. This is one extreme of Comartin's [[Multi-Tenancy Isn't About Databases]] spectrum: maximum control, maximum operational cost — the trade-off acknowledged and accepted.

**Sacrificed**: Operational maturity. No Realtime (WebSocket push), no managed backups, no PITR, no HA, no enterprise SSO/SCIM. The architecture docs acknowledge these as "known gaps."

**Interesting trade-offs**:
- **Auth in Java instead of GoTrue**: avoids a second service to deploy but means independent maintenance of auth compatibility
- **PostgREST reimplementation vs. proxying**: full control over query generation and tenant routing, but must maintain compatibility with PostgREST syntax
- **Embedding cache shared across tenants**: content-addressed by `SHA-256(provider::dimensions::text)`, so same text = same vector regardless of tenant. Safe because vectors are not tenant-tagged
- **Session messages written AFTER extraction**: prevents the LLM from seeing the current turn twice during fact extraction. The session window is injected into the *next* call's extraction context
- **All-in-one Docker image**: bundles Postgres + Redis + backend + Studio into one container for one-command self-hosting. Production deployments use separate services

## Comparison Notes

- **vs [[InsForge]]**: Both are "Supabase for agents" with Postgres+RLS, auth, S3, and MCP tools. Nubase adds first-class Memory (not just vector search — full fact extraction + decision pipeline + entity linking), Assets/CDN for frontend publishing, an AI Gateway, and Cron. InsForge uses Deno for functions; Nubase uses Cloudflare Workers for Platforms.
- **vs [[Xano]]**: Xano is a no-code backend with visual builder. Nubase has no visual builder — it's designed for agents to drive via MCP tools and REST.
- **vs [[Agent Memory and Context]] (hub)**: The wiki's memory taxonomy distinguishes between agent-side middleware ([[Sawtooth Memory]], [[Memento]], [[Mnemo]]) and server-side platforms. Nubase is server-side, multi-tenant, with hybrid retrieval (vector + BM25 + entity boost) and auth-integrated ownership scoping. Like [[MELT]], it tracks lifecycle dynamics (ADD/UPDATE/DELETE/NONE), but does so as a production service rather than a benchmark harness.
- **vs [[TradingGoose Bear Researcher]]**: That project uses Supabase Edge Functions for agent orchestration. Nubase replaces both the Supabase backend and the function runtime.

---
*Sources: [[summary/nubase]]*
*Last updated: 2026-07-05*
