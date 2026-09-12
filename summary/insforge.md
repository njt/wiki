---
url: https://github.com/InsForge/InsForge
title: InsForge — Open-Source Backend-as-a-Service for Agentic Coding
author: InsForge
date_fetched: 2026-05-18
date_published: 2026-03-06 (v2.0.0)
topics:
  - mcp-and-tool-protocols
---

# InsForge Full Analysis

## Project Summary

InsForge is an open-source backend-as-a-service (BaaS) platform designed specifically for AI coding agents. It provides database (PostgreSQL with RLS), auth (JWT + OAuth), storage (S3-compatible), edge functions (Deno), compute (Fly.io), deployments (Vercel), real-time (WebSocket + pg_notify), email (SMTP), payments (Stripe), and an AI model gateway (OpenRouter). Agents interact via an MCP server or CLI+Skills. It runs as a self-hosted monolith or a cloud-managed service.

## Architecture Analysis

### Monorepo Structure (Turborepo)

```
/
  backend/           — Express server (TypeScript, ESM), the entire backend
  frontend/          — React+Vite shell for self-hosting mode
  packages/
    shared-schemas/  — Zod schemas shared across packages (@insforge/shared-schemas)
    dashboard/       — Publishable React dashboard (self-hosting + cloud modes)
    ui/              — Reusable design-system primitives (shadcn/ui-based)
  docs/              — Product and agent-facing documentation
```

### Backend Layered Architecture

Routes → Services → Providers/Infra (strict layering, enforced by project skills):

- **Routes** (`backend/src/api/routes/`): HTTP handlers — auth parsing, input validation (Zod safeParse + AppError), delegation to services. Never access the database directly.
- **Services** (`backend/src/services/`): Business logic and orchestration. The only layer allowed to interact with PostgreSQL. Singleton pattern (`getInstance()`).
- **Providers** (`backend/src/providers/`): External system wrappers — S3, Stripe, OpenRouter, Deno Subhosting, Fly.io, Vercel, OAuth providers (Google, GitHub, Discord, LinkedIn, Facebook, Microsoft, X, Apple), CloudWatch, SMTP.
- **Infra** (`backend/src/infra/`): Lower-level infrastructure — DatabaseManager (pg Pool), TokenManager (JWT), EncryptionManager, RealtimeManager (pg_notify listener), SocketManager (Socket.IO).

### Key Design Decisions

1. **Singletons everywhere**: Every service, provider, and manager is a singleton accessed via `getInstance()`. This means the system is inherently single-process — not horizontally scalable without redesign. The backend skill file explicitly warns: "InsForge currently runs as a single-instance server."

2. **PostgreSQL as the universal datastore**: Everything lives in Postgres — auth users, storage metadata, secrets, deployments, schedules, payments, real-time messages, compute services. The system uses 43+ migration files to manage schema evolution.

3. **Row Level Security (RLS)**: Database-level access control via PostgreSQL RLS policies. The `withUserContext` pattern opens a transaction, sets `SET LOCAL ROLE` plus `request.jwt.claims` via `set_config`, executes the query, commits, resets role. Admin callers get `isAdmin: true` and bypass RLS via elevated role. The pattern is explicitly documented as the required approach: "Do not write `WHERE user_id = $1` filters in services; let RLS evaluate `auth.jwt() ->> 'sub'` against the row."

4. **Dual-mode (cloud + self-hosted)**: Environment-gated behavior. `isCloudEnvironment()` checks toggle between cloud-managed credentials (fetched from api.insforge.dev) and self-hosted env vars. The OpenRouterProvider is the clearest example: cloud projects get an API key from the cloud backend; self-hosted reads `OPENROUTER_API_KEY` from env.

5. **Branch mode S3 storage**: When `PARENT_APP_KEY` is set, the S3 provider runs in "branch mode" — read paths fall back to the parent's S3 prefix on 404, writes go to the branch's own prefix. This enables preview deployments that share parent data.

6. **MCP server interface**: The package.json description says "MCP integration." The README explains two agent interfaces: an MCP server exposing operations as tools, and a CLI+Skills interface for cloud users. The MCP server is the primary channel for coding agents.

## Key Techniques

### SQL Parser with WASM-based libpg_query

`backend/src/utils/sql-parser.ts` uses `libpg-query` (a WASM binding to PostgreSQL's actual parser) to analyze SQL statements. This is used to:
- Block dangerous operations (DROP DATABASE, CREATE DATABASE)
- Prevent writes to InsForge-managed schemas (auth, storage, realtime, etc.)
- Classify statement types (INSERT, ALTER TABLE, CREATE INDEX, etc.) for cache invalidation
- Allow specific exceptions (RLS policy creation on storage.objects, trigger creation on payment tables)

This is more robust than regex-based SQL parsing. The project uses the actual PostgreSQL parser, so it understands SQL the same way the database does.

### S3 Protocol Gateway with Signature V4

`backend/src/api/routes/s3-gateway/` implements a full S3-compatible API. A `dispatch.ts` module maps HTTP method+path+query combinations to 18 S3 operations (ListBuckets, PutObject, DeleteObjects, CreateMultipartUpload, etc.). Each operation has its own handler in `commands/`. The gateway is mounted BEFORE JSON body parsing middleware so request bodies stream through untouched — critical for `STREAMING-AWS4-HMAC-SHA256-PAYLOAD` chunked signatures. This is a significant technical feat: a partial reimplementation of the S3 protocol for compatibility with any S3 SDK.

### Real-time via PostgreSQL LISTEN/NOTIFY + Socket.IO

The `RealtimeManager` (`backend/src/infra/realtime/realtime.manager.ts`) uses a dedicated PostgreSQL connection (not pooled) for `LISTEN realtime_message`. When a database trigger fires `pg_notify('realtime_message', payload)`, the manager receives it and fans out via:
- Socket.IO rooms (WebSocket to connected clients)
- HTTP POST webhooks (for server-side consumers)
- Message delivery tracking (updates message records in Postgres with delivery stats)

This is a clean, Postgres-native approach to real-time — no external message broker needed.

### Encrypted Secrets Manager

Secrets (API keys, credentials) are stored encrypted in PostgreSQL. The `EncryptionManager` (`backend/src/infra/security/encryption.manager.ts`) provides encryption/decryption with an app key. Secrets are deduplicated (migration 035 adds a unique constraint) and versioned.

### OAuth PKCE Flow with HTTP-Only Cookies

The auth system uses PKCE (Proof Key for Code Exchange) for OAuth flows. The `OAuthPKCEService` stores code verifiers in a map with cleanup intervals. Refresh tokens are set as HTTP-only cookies with CSRF protection via HMAC-derived CSRF tokens (not double-submit pattern — the CSRF token is `HMAC-SHA256(refresh_token, JWT_SECRET)` and verified with timing-safe comparison).

### PostgreSQL Advisory Locks for Payment Operations

`backend/src/services/payments/payments-advisory-lock.ts` uses PostgreSQL advisory locks (`pg_try_advisory_lock`) for payment session operations. This prevents race conditions in Stripe checkout flows without external distributed locking.

## Innovation Points

1. **Agent-first backend design**: Unlike Supabase which targets human developers with a dashboard, InsForge targets coding agents with an MCP server. The entire backend is designed to be operated by AI, not clicked through a UI. This is a category-defining choice.

2. **libpg_query for SQL validation**: Most BaaS platforms use regex or custom parsers. InsForge uses the actual PostgreSQL parser compiled to WASM, giving it perfect SQL understanding — it can distinguish `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` (allowed) from `ALTER TABLE ... DROP COLUMN` (blocked on managed schemas).

3. **S3 protocol gateway for storage compatibility**: Rather than a custom storage API, InsForge implements the S3 wire protocol so any S3 SDK works. This is non-trivial — it requires handling AWS Signature V4, chunked transfer encoding, multipart uploads, and XML responses.

4. **Branch-aware S3 provider**: The parent/child app key pattern for preview deployments is clever. A branch deployment shares the parent's storage data (read-through fallback) but writes to its own prefix, enabling realistic previews without data duplication.

5. **Agent skills for repo development**: The `.claude/skills/insforge-dev/` directory contains detailed skill files for contributing to InsForge itself. The backend skill is 74 lines of strict conventions (use RLS not WHERE clauses, idempotent migrations, transaction discipline). This is meta: a project for AI coding agents uses AI coding agents to build itself.

## Design Trade-offs

| Trade-off | Choice | Consequence |
|-----------|--------|-------------|
| Scalability vs. simplicity | Single-instance server | Can't horizontally scale without redesign, but architecture is simple and debuggable |
| Flexibility vs. correctness | RLS-enforced access control | Harder to add custom authorization logic, but impossible to accidentally expose data |
| Performance vs. safety | WASM SQL parser | Slightly slower startup (async WASM load), but perfect SQL parsing |
| Latency vs. data integrity | Everything in Postgres | No specialized stores (no Redis, no Kafka), but single source of truth |
| Ease of use vs. security | PKCE OAuth with HTTP-only cookies | More complex client setup, but resistant to CSRF and token theft |
| Cloud vs. self-hosted | Provider abstraction pattern | Code complexity from dual-mode branching, but works in both environments |

## Test Coverage

The project has extensive tests (~80+ test files in `backend/tests/`), including:
- Unit tests for services, providers, and migrations
- Idempotency tests for migrations (re-run safety)
- Security tests (SQL injection PoC, rate limiting, RLS enforcement)
- Integration tests (S3 gateway CRUD, multipart uploads)
- Local shell scripts for curl-based e2e testing
- Manual test scripts for AI model plugins, embeddings, bulk operations

## Comparison to Related Projects

**vs. Supabase**: Supabase is a full BaaS with a polished dashboard for humans. InsForge mirrors much of Supabase's feature set (Postgres, auth, storage, real-time, edge functions) but targets AI agents instead of human developers. The architecture is similar (Postgres-centric, RLS-based), but InsForge strips away the dashboard in favor of MCP tools.

**vs. Xano**: Xano is a no-code BaaS with visual builders. InsForge is code-first with agent-first interfaces. Xano targets business users; InsForge targets developers using coding agents.

**vs. n8n**: n8n is workflow automation with 400+ integrations. InsForge is infrastructure provisioning and management for agents building apps.

**vs. Firebase**: Firebase is Google's managed BaaS. InsForge is open-source, self-hostable, and Postgres-based vs. NoSQL. Firebase targets mobile/web developers; InsForge targets coding agents.
