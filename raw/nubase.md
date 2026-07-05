---
url: https://github.com/OtterMind/Nubase
title: Nubase
author: OtterMind
date_fetched: 2026-07-05
date_published: 2025-05
---

# Nubase — Full Source Analysis

## Overview

Nubase is an open-source (Apache 2.0), AI-native backend-and-deploy platform that a coding agent drives directly. It's a self-hosted Supabase alternative designed for multi-project isolation with eight capability modules: Database, Auth, Storage, Assets, Functions, AI Gateway, Memory, and Cron. The project is built as a Spring Boot 3.2 monolith (Java 17) with a Next.js Studio frontend and an MCP bridge (TypeScript CLI).

Repository: https://github.com/OtterMind/Nubase
License: Apache 2.0
Language composition: Java (backend, ~12,500+ LOC main), TypeScript (frontend + MCP bridge), SQL (scripts)

## Architecture Deep Dive

### Two-Layer Database Architecture

The most important architectural decision: Nubase uses **database-per-tenant isolation**, not schema-per-tenant. Each project gets its own physical PostgreSQL database. This is managed through:

1. **Metadata Database** (control plane): stores project configs, encrypted credentials, platform users, JWT secrets
2. **Project Databases** (data plane): one per project, containing `auth.*`, `storage.*`, `mem.*`, and `public.*` application schemas

The critical files:
- `DatabaseMultiTenantConnectionProvider.java` — Hibernate multi-tenant connection provider that routes JDBC connections to the correct project database via `RoutingDataSource`
- `DatabaseTenantResolver.java` — resolves `database_key` from `MultiTenancyContext` (ThreadLocal), returns "default" at startup when no context exists
- `UnifiedMultiTenancyFilter.java` — 520 lines, the heart of request routing. Extracts `apikey` JWT, parses `ref` claim (app_code), looks up database config, validates JWT signature, sets ThreadLocal context

### Token Model

Two-layer auth:
- `apikey` header: JWT with `ref` (project reference) and `role` (anon/authenticated/service_role) claims — selects project + database role
- `Authorization: Bearer <jwt>`: user token for RLS `auth.uid()` and user-scoped memory access

The `UnifiedMultiTenancyFilter` handles both layers: it extracts the apikey to resolve the tenant, then optionally validates the Bearer token to authenticate the end user.

### Request Routing Flow

```
Client → UnifiedMultiTenancyFilter → buildUnifiedContext()
  1. Extract apikey JWT → parse ref (app_code) + role
  2. DatabaseConfigRepository.findByAppCode() → load tenant config
  3. Validate JWT signature against project's secret
  4. RoutingDataSource.initializeDataSource() if not cached
  5. MultiTenancyContext.set() → ThreadLocal
  6. authenticateUser() → Optional Bearer token validation
  7. filterChain.doFilter() → Controller → JDBC (routed via Hibernate multi-tenancy)
  8. finally: MultiTenancyContext.clear()
```

### PostgREST-Compatible API

The `/rest/v1/*` API is a **Java reimplementation of PostgREST**, not proxied to a PostgREST instance. Key components:
- `ApiRequestParser` — parses query strings into structured request objects
- `QueryPlanner` — generates SQL from parsed requests
- `QueryExecutor` — executes against the tenant's database via `RoutingDataSource`
- `SchemaCache` — per-project table/column/FK metadata cache, refreshed on-demand
- `SqlGenerationTest.java` / `CompleteSqlGenerationTest.java` — comprehensive SQL generation tests

Supports: select, filter (with PostgREST operator syntax), order, limit/offset/range, insert, update, delete, upsert, embedded resources, JSONB conversion.

### Memory System (the most innovative module)

The memory system in `ai.nubase.mem` implements a **Mem0-style pipeline** with three LLM calls per ingestion:

```
AddMemoryRequest → MemoryService.add()
  ├── FactExtractionService.extract() → LLM call #1: extract facts + entities from conversation
  ├── EmbeddingService.embedBatch() → vectorize each fact (cached with Caffeine, SHA-256 keyed)
  ├── MemoryRepository.searchByVector() → pgvector: top-K nearest for each fact
  ├── MemoryDecisionService.decide() → LLM call #2: ADD/UPDATE/DELETE/NONE per fact
  └── applyFactsWithDecisions() → transactional writes
      ├── ADD: insert memory + embedding + link entities
      ├── UPDATE: update text + re-embed + strip old entity links + link new entities
      ├── DELETE: soft-delete + unlink entities
      └── NONE: relink entities only (idempotent)
```

**Search** uses multi-signal fusion (`ScoreFusion.java`, 180 lines):
```
SearchMemoryRequest → MemoryService.search()
  ├── EmbeddingService.embed(query) → vector
  ├── MemoryRepository.searchByVector() → pgvector cosine distance (over-fetch: min(topK*4, 60))
  ├── MemoryRepository.searchByText() → Postgres full-text (ts_rank_cd), same over-fetch
  ├── QueryEntityExtractionService.extract() → LLM call #3: extract entities from query
  ├── EntityStoreService.computeBoosts() → boost memories linked to those entities
  └── ScoreFusion.fuse() → combined = (semantic_sim + bm25_norm + entity_boost) / max_possible
```

**Key design decisions in memory:**
- **SHA-256 content hashing** for dedup within owner scope (`findByHash` before INSERT)
- **Owner-scoped authorization** throughout: every id-based lookup uses `findByIdForOwner()`, defense-in-depth with Java-side re-check
- **Session window**: stores recent N messages in `mem.session_messages` with eviction, injects history into extraction context for conversation continuity
- **Entity linking**: extracted entities (person, location, organization, product, event, food, date) are linked to memories; UPDATE strips old links, DELETE removes them; entities survive NONE decisions (relinking is idempotent)
- **Soft delete**: `deleted_at` column, never hard-deletes (except `reset()` for admin)
- **Append-only history**: every ADD/UPDATE/DELETE writes to `mem.memory_history`

### LLM Provider Abstraction

`LLMProviderRegistry.java` — plugin architecture via Spring bean discovery:
- Chat providers: `openai`, `anthropic`, `generic` (OpenAI-compatible) — each implements `ChatLLMProvider`
- Embedding providers: same three names, each implements `EmbeddingProvider`
- Per-tenant overrides via `MemConfigResolver` which reads `mem.config` JSON from the project database and falls back to YAML defaults
- `ObjectProvider` pattern breaks circular dependency between resolver and registry

### MCP Server

Uses Spring AI's `@Tool` annotation for MCP tool exposure. Tool files:
- `DatabaseMcpTools.java` (21,767 bytes — largest): listTables, inspectTable, exportRlsPolicies, executeSql, initDatabase, etc.
- `FunctionsMcpTools.java`: deploy, invoke, list functions
- `CronMcpTools.java`: create/update/delete/list scheduled jobs
- `AssetsMcpTools.java`: upload static assets to CDN
- `MemoryMcpTools.java`: add/search/list/delete memories
- `StorageMcpTools.java`, `AuthMcpTools.java`, `GatewayMcpTools.java`, `DeploymentsMcpTools.java`

The frontend MCP bridge (`nubase_cli`, TypeScript) provides an alternative MCP stdio server that proxies to the Nubase REST API, used when agents connect via Claude Code / Codex without direct access to the backend's MCP endpoint.

### Edge Functions

`EdgeFunctionAdminService.java` (294 lines): manages function deployment with versioning, active version selection, per-function secrets, and rate limits. Supports local execution or Cloudflare Workers for Platforms dispatching. The Cloudflare worker at `cloudflare/functions-dispatcher/worker.js` handles the actual function invocation in the WfP model.

### Cron / Scheduled Jobs

`ScheduledJobRunner.java` — claims jobs via `FOR UPDATE SKIP LOCKED` (Postgres advisory-style locking), runs them at their cron schedule, records execution history. Supports two target types: edge function invocation and named database function calls.

## Key Techniques

1. **Database-per-tenant with Hibernate multi-tenancy**: Rather than Postgres schema-per-tenant (which Supabase self-hosted does), Nubase uses Hibernate's `MultiTenantConnectionProvider` interface backed by a `RoutingDataSource` that maps `database_key` → `HikariDataSource`. Each project gets its own connection pool. This is more resource-intensive but gives stronger isolation — a runaway query in one project can't affect another's database.

2. **ThreadLocal context propagation**: `MultiTenancyContext` uses ThreadLocal to carry tenant identity through the request lifecycle. Set by `UnifiedMultiTenancyFilter.doFilterInternal()`, cleared in `finally`. Every downstream component (repositories, services, query planners) reads from it.

3. **Pre-flight LLM availability checks**: Before every LLM call, both `FactExtractionService` and `MemoryDecisionService` check `provider.isAvailable()`. Without this, every `add()` with `infer=true` would rack up doomed HTTP attempts + 401 errors when no API key is configured. The embedding service instead throws — embeddings are mandatory for memory to function.

4. **Deterministic fallback for LLM failure**: If the decision LLM call fails or returns unparseable JSON, `MemoryDecisionService.fallbackAllAdd()` treats every fact as ADD. The system degrades to duplicate-prone but functional behavior rather than losing data.

5. **Caffeine embedding cache with batch dedup**: `EmbeddingService.embedBatch()` does a cache lookup per text, collects only misses, and handles intra-batch duplicates (same text appearing multiple times) by filling from cache after the upstream call. Content-addressed by `SHA-256(provider_name::dimensions::text)` — safe to share across tenants.

6. **ScoreFusion mirrors mem0's scoring.py**: `ScoreFusion.fuse()` implements the exact formula from `mem0/utils/scoring.py`: `combined = (semantic_sim + bm25_norm + entity_boost) / max_possible`. BM25 scores are min-max normalized per result set. Entity boost weight is 0.5 (matching mem0's constant).

7. **SqlRiskClassifier**: SQL executed through MCP tools goes through a risk classifier that categorizes statements (DML vs DDL vs destructive). The MCP bridge (`sql-risk.ts`) also has client-side classification.

8. **Subdomain-based tenant resolution**: For apikey-free endpoints (OAuth callback, public storage, SSO ACS), the tenant is resolved from the request's subdomain (e.g., `app20260108111430zrjprnycyi.nubase.co` → `app20260108111430zrjprnycyi`). The code deliberately does NOT fall back to `Referer` header for security reasons.

## Design Decisions & Trade-offs

### Optimized for: AI agent UX and multi-project self-hosting
- The MCP bridge, memory API, and "one command to deploy" ergonomics are designed for coding agents, not human developers
- Database-per-tenant over schema-per-tenant: stronger isolation at the cost of more connections/pools
- Java monolith over microservices: simpler to self-host (single Docker image), at the cost of scaling granularity

### Sacrificed: Operational maturity
- No Realtime (WebSocket push)
- No managed backups, PITR, or HA
- No per-project billing
- No enterprise SSO/SCIM
- The architecture.md acknowledges these as "known gaps"

### Interesting trade-offs:
- **Auth in Java instead of GoTrue**: avoids a second service to deploy, but means Nubase must keep up with Supabase auth changes independently
- **PostgREST reimplementation vs. proxying**: gives full control over query generation and tenant routing but requires maintaining compatibility with PostgREST's query syntax
- **Embedding cache shared across tenants**: content-addressed, so same text = same vector regardless of tenant. Safe because the cache key includes the provider+model+dimensions
- **Session messages for extraction context**: writing current messages AFTER extraction (not before) prevents the LLM from seeing the same turn twice. The session window is for the *next* call, not the current one

## Comparison with Related Projects

- **vs InsForge**: InsForge is also "Supabase for agents" with Postgres+RLS, auth, S3, Deno functions, and MCP tools. But Nubase adds first-class Memory (not just vector search — full fact extraction + decision pipeline + entity linking), Assets/CDN, and an AI Gateway. InsForge uses Deno for functions; Nubase uses Cloudflare Workers for Platforms or a local executor.
- **vs Xano**: Xano is a no-code backend with visual builder. Nubase has no visual builder — it's designed for agents to drive via MCP tools and REST APIs.
- **vs Supabase self-hosted**: Supabase's self-hosted stack is single-project. Nubase's differentiator is multi-project isolation (one Studio provisions many isolated project databases) plus built-in Memory.
- **vs Sawtooth Memory / Memento / Mnemo**: These are agent-side memory middleware. Nubase's memory is server-side, multi-tenant, with pgvector + BM25 + entity boost fusion. It integrates directly with the auth system (user-scoped memory ownership).
