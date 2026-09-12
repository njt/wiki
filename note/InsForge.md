# InsForge

An open-source backend-as-a-service platform designed specifically for AI coding agents. InsForge gives agents database (PostgreSQL with RLS), auth (JWT + OAuth), S3-compatible storage, edge functions (Deno), compute (Fly.io), deployments (Vercel), real-time, email, payments (Stripe), and an AI model gateway (OpenRouter) — all exposed as MCP tools. Unlike Supabase which targets human dashboard users, InsForge is built to be operated entirely by coding agents through an MCP server interface.

Tags: #tool #project #agents #database #BaaS

---

## Architecture

InsForge is a TypeScript monorepo managed by Turborepo with a strict layered backend: **Routes → Services → Providers/Infra**.

```
backend/src/
  api/routes/     — HTTP handlers, auth parsing, Zod validation, delegation
  services/       — Business logic, PostgreSQL access (singleton pattern)
  providers/      — External system wrappers (S3, Stripe, OpenRouter, Fly, Vercel, OAuth)
  infra/          — Lower-level: DatabaseManager, TokenManager, EncryptionManager, RealtimeManager
packages/
  shared-schemas/ — Zod schemas shared across all packages
  dashboard/      — Publishable React dashboard (self-hosting + cloud modes)
  ui/             — Reusable design-system primitives (shadcn/ui)
frontend/         — React+Vite shell mounting the dashboard in self-hosting mode
```

The backend runs as a **single-instance Express server** (explicitly documented as such — the backend skill file warns not to introduce distributed coordination assumptions). All data lives in PostgreSQL, including auth users, storage metadata, secrets, deployments, real-time messages, and compute service records.

### Core architectural patterns

- **Singleton everywhere**: Every service, provider, and manager uses `getInstance()`. Simple and debuggable at the cost of horizontal scalability.
- **PostgreSQL RLS for authorization**: Not `WHERE user_id = $1` in app code. The `withUserContext` pattern (`backend/src/services/db/user-context.service.ts`) opens a transaction, sets `SET LOCAL ROLE` + `request.jwt.claims` via `set_config`, runs the query, and commits. Admin callers bypass RLS via elevated role.
- **Provider abstraction for dual-mode**: `isCloudEnvironment()` gates between cloud-managed credentials and self-hosted env vars. OpenRouterProvider is the clearest example: cloud projects fetch API keys from `api.insforge.dev`; self-hosted reads `OPENROUTER_API_KEY`.
- **Branch-aware S3 storage**: When `PARENT_APP_KEY` is set, reads fall back to the parent's S3 prefix on 404; writes go to the branch's own prefix. Enables preview deployments without data duplication.

## Key Techniques

### SQL parsing via libpg_query WASM

`backend/src/utils/sql-parser.ts` uses PostgreSQL's actual C parser compiled to WASM (`libpg-query`). This gives perfect SQL understanding — it can distinguish `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` (allowed on managed schemas) from `ALTER TABLE ... DROP COLUMN` (blocked). Regex-based approaches can't do this. The parser is used to:
- Block database-level DDL (DROP/CREATE/ALTER DATABASE)
- Prevent writes to InsForge-managed schemas (auth, storage, realtime, payments, secrets) with explicit exception lists for RLS policies and triggers
- Classify statement types for cache invalidation after DDL operations

### S3 protocol gateway

`backend/src/api/routes/s3-gateway/` implements the S3 wire protocol. A dispatcher (`dispatch.ts:46-80`) maps HTTP method+path+query to 18 S3 operations (ListBuckets, PutObject, CreateMultipartUpload, etc.). The gateway is mounted **before** JSON body parsing so request bodies stream through untouched — critical for `STREAMING-AWS4-HMAC-SHA256-PAYLOAD` chunked signatures. This means any S3 SDK works with InsForge storage without modification.

### Real-time via pg_notify + Socket.IO

The `RealtimeManager` uses a dedicated (non-pooled) PostgreSQL connection for `LISTEN realtime_message`. Database triggers call `pg_notify('realtime_message', payload)`, and the manager fans out via Socket.IO rooms (WebSocket) and HTTP POST webhooks. No external message broker needed — this is Postgres-native real-time.

### PKCE OAuth with CSRF tokens

Auth uses PKCE flow with HTTP-only refresh token cookies. The CSRF token is `HMAC-SHA256(refresh_token, JWT_SECRET)` verified with `crypto.timingSafeEqual` — cryptographically bound to the session, not a separate random value.

### PostgreSQL advisory locks for payment operations

Payment webhook processing uses `pg_try_advisory_lock` (`backend/src/services/payments/payments-advisory-lock.ts`) to prevent race conditions without external distributed locking.

## Design Decisions

| Trade-off | Choice | Consequence |
|-----------|--------|-------------|
| Scalability vs. simplicity | Single-instance server | Can't horizontally scale without redesign |
| Flexibility vs. correctness | RLS-enforced access control | Harder to add custom auth logic, but impossible to accidentally expose data |
| Performance vs. safety | WASM SQL parser | Slightly slower startup, but perfect SQL analysis |
| Latency vs. data integrity | Everything in Postgres | No Redis/Kafka, but single source of truth and transactional consistency |
| Ease of use vs. security | PKCE OAuth + HTTP-only cookies | More complex client setup, but resistant to CSRF and token theft |

The single-instance choice is the most defining trade-off. It simplifies the codebase dramatically (no distributed locking, no leader election, no eventual consistency) but means InsForge can't scale horizontally without a significant rewrite. For the target use case — individual developers and small teams using coding agents — this is the right call.

### Opinionated take on the RLS approach

Using PostgreSQL RLS with `SET LOCAL ROLE` transactions is more correct than application-level WHERE clauses, but it adds significant complexity to testing. Tests must mock the pool/client and pin exact SQL sequences (see `tests/unit/user-context.service.test.ts`). The RLS enforcement is also invisible at the route level — a developer reading route code won't see the authorization logic, which can be surprising. The trade-off is worth it for production safety but has a real debugging cost.

## Comparison Notes

**vs. Supabase**: InsForge mirrors Supabase's feature set (Postgres, auth, storage, real-time, edge functions) but targets AI agents instead of human developers. Both use RLS-based authorization and Postgres-centric architecture. InsForge strips away the dashboard in favor of MCP tools and adds compute (Fly.io), deployments (Vercel), and payments (Stripe) that Supabase handles differently.

**vs. [[Xano]]**: Xano is a no-code BaaS with visual builders targeting business users. InsForge is code-first with agent-first interfaces. Both provide Postgres-backed APIs, but the user personas are completely different.

**vs. [[DAB]]** (Microsoft's Data API Builder): DAB provides REST, GraphQL, and MCP over any database. InsForge is a full BaaS with auth, storage, functions, and more. DAB is a narrower tool; InsForge is a platform. Both expose MCP, but at different levels of abstraction.

**vs. [[How Intercom Uses Claude Code]]**: Intercom built plugins and skills on top of Claude Code for their existing infrastructure. InsForge provides the infrastructure itself — the database, auth, storage, and compute that agents need to build apps from scratch. Complementary approaches: Intercom layers agent tooling on existing systems; InsForge provides the systems.

**vs. [[Agent-Native Architectures (Every)]]**: Every's design guide emphasizes files as universal interfaces and composability. InsForge operationalizes this at the infrastructure level: the backend IS the universal interface that agents interact with through MCP tools. The layered architecture (Routes → Services → Providers) mirrors the composability principle.

---

*Sources: [[summary/insforge]]*
*Last updated: 2026-05-18*
