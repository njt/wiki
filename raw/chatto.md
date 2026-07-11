---
url: https://github.com/chattocorp/chatto
title: Chatto
author: chattocorp (Hendrik Mans)
date_fetched: 2026-07-11
date_published: 2026-03-01
---

# Chatto — Raw Analysis

A real-time chat application for teams and communities, built in Go with a SvelteKit frontend, running entirely on NATS/JetStream with no relational database. The project is a monorepo — `cli/` is the Go binary (embedded NATS + HTTP server + ConnectRPC API), `apps/frontend/` is the SvelteKit SPA compiled into the binary, `proto/` holds the protobuf definitions, and `docs/adr/` holds 48 Architecture Decision Records.

## Project Scale

- **~127K lines of Go** in `cli/internal/` (core domain logic, HTTP server, event framework, ConnectRPC handlers, protobuf bindings, video processing)
- **~1,504 lines** in `cli/internal/core/core.go` — the central hub `ChattoCore`
- **~597 lines** in `cli/internal/events/publisher.go` — the OCC-only event publisher
- **~965 lines** in `cli/internal/events/projector.go` — projection framework with shared replay
- **~60+ e2e Playwright tests** in `apps/frontend/e2e/`
- **48 ADRs** documenting architectural decisions from NATS-as-DB to frontend optimistic UI patches
- **28 FDRs** (Feature Decision Records) documenting per-feature design
- **Go module**: `hmans.de/chatto`, Go 1.26.0, ~80 direct dependencies
- **Frontend**: Svelte 5, SvelteKit, TipTap editor, Paraglide i18n, Playwright, Vitest
- **License**: AGPL-3.0-or-later, with Apache-2.0 exceptions for standalone frontend and integration surfaces

## Core Architecture

Chatto is a **single-binary event-sourced chat server** using NATS/JetStream as the sole persistent data store. There is no PostgreSQL, no MongoDB, no Redis — just NATS. The binary embeds a full NATS server for single-process deployments; for horizontal scaling, multiple Chatto instances connect to an external NATS cluster.

### Storage Architecture

**Three storage tiers, all NATS-native:**

1. **`EVT` JetStream stream** — the event-sourcing log. Every durable domain fact (user created, message posted, reaction added, role created, room membership changed, etc.) is appended as a protobuf-encoded event. This is the system's source of truth.

2. **KV Buckets** — latest-value key-value stores for runtime state:
   - `RUNTIME_STATE` — file-backed, TTL-driven: notifications, push subscriptions, auth tokens, link-preview cache, wrapped application DEK records
   - `MEMORY_CACHE` — memory-backed, volatile: presence state, leader-election leases
   - `ENCRYPTION_KEYS` — KMS key-encryption keys, excluded from backups

3. **Object Stores** — binary assets:
   - `SERVER_ASSETS` — avatars, branding, link previews, message attachments. NATS-backed or S3-backed
   - `ASSET_CACHE` — optional TTL-based cached image transforms

All writes to `EVT` use **optimistic concurrency control** (OCC). The publisher framework (`cli/internal/events/publisher.go`) offers no non-OCC publish primitive. Every write carries `Nats-Expected-Last-Subject-Sequence`, guaranteeing per-subject serialized history with no race gaps.

### Read Model: In-Memory Projections

Domain state is never read directly from `EVT`. Instead, **13 in-memory projections** consume the event stream and maintain current state in process memory:

- **Room Directory** — room catalog, membership, bans (`evt.room.>`)
- **Room Group Layout** — room groups, sidebar ordering (`evt.group.>`, `evt.layout.>`)
- **Room Timeline** — per-room visible message timeline (`evt.room.>`)
- **Threads** — per-thread reply logs, participants, follow state (selected `evt.room.*` events)
- **Reactions** — current per-message reaction sets (`evt.room.>` — intentionally broad for room-tail OCC)
- **Call State** — active LiveKit call state (`evt.room.>`)
- **Server Config** — server config, user preferences (`evt.config.>`)
- **Users** — account/profile/custom-status/auth state (`evt.user.>`)
- **Content Keys** — per-user DEK epochs (`evt.user.*.dek_generated`, `evt.user.*.user_key_shredded`)
- **RBAC** — roles, assignments, permission decisions (`evt.rbac.>`)
- **Mentionables** — @handle namespace across users and roles (full `evt.>`)
- **Assets** — asset lifecycle, processing state, derivatives (`evt.asset.>`, legacy `evt.room.*.asset_*`)

Each projection is a Go `struct` implementing `Apply(event, seq) error`. At startup, `ChattoCore.Run` replays `evt.>` through **one shared ordered consumer**, decodes each event once, then fans it out to all projectors whose subject filters match. This shared replay avoids duplicate decode work and JetStream metadata parsing.

**Read-your-writes** is per-process: after a successful publish, the writer waits for the local projector to advance past the published sequence before returning. Cross-process consistency is eventual (sub-millisecond, bounded by NATS Core latency).

### Write Path: Event-Sourced with OCC

`events.Publisher` provides the write primitives:

- `Append(ctx, subject, event)` — auto-computes expected seq, publishes with OCC
- `AppendEventually(ctx, subject, event)` — retries OCC conflicts up to 5 times with exponential backoff (for append-only events like messages, memberships)
- `AppendAt(ctx, subject, event, expectedSeq)` — explicit expected sequence (for deterministic replay)
- `AppendAtFilter(ctx, subject, event, filter, expectedFilterSeq)` — OCC against a wildcard subject filter (for cross-aggregate invariants like unique room names)
- `AppendBatch(ctx, entries)` — atomic batch publish for multi-aggregate cascades, using NATS batch protocol

Conflict handling is explicit: state-replacement mutations (profile updates, config changes) must re-read the projection and recompose; append-only mutations (messages, reactions) can retry the same payload.

### API Layer

**Protobuf-first** with two surfaces:

1. **ConnectRPC** mounted at `/api/connect` — ~100+ unary RPCs across five service packages:
   - `chatto.auth.v1` — external identity auth
   - `chatto.discovery.v1` — unauthenticated server discovery (CORS-wildcard)
   - `chatto.api.v1` — integration-oriented API (rooms, messages, users, notifications, etc.)
   - `chatto.admin.v1` — administration (server config, RBAC, event log)
   - `chatto.operator.v1` — root-equivalent API on a Unix socket only

2. **Realtime WebSocket** at `/api/realtime` — protobuf binary frames (`chatto.realtime.v1.Realtime*`), authenticate with bearer token in hello frame or cookie session. Live-only (v1), no replay cursors; missed events recovered via projected ConnectRPC reads.

The authentication model uses `session.{hmac}` typed runtime credentials (ADR-046). Bearer tokens for cross-origin clients, cookie sessions for same-origin browsers.

### RBAC Model

Permission-only RBAC (ADR-040) with hierarchical scope resolution (ADR-005):
- Server scope → Room Group scope → Room scope
- Deny wins at more specific scopes
- Default roles (owner, admin, moderator, member, everyone) with system-immutable positions
- Custom roles with reordering
- Direct user permission overrides for exceptions
- Owner role is immutable and has an implicit override on all permissions

### Encryption: Per-User with Crypto-Shredding

Each user has a Data Encryption Key (DEK) managed through a KMS (`cli/internal/kms/`). Message bodies are encrypted with per-user keys. Deleting a user's key shreds all their messages permanently — satisfying GDPR deletion requirements without touching the immutable event stream (ADR-007).

### Client Architecture

Multi-instance client (ADR-025): the SvelteKit SPA connects to multiple Chatto servers simultaneously. Server state is managed per-server in the frontend state layer (`src/lib/state/`). The frontend uses:
- **Optimistic UI** with scoped provisional patches (ADR-048)
- **Service Worker** for asset URL resolution (ADR-039) and push notifications
- **Svelte 5** runes for reactive state management
- **TipTap** for the rich-text composer with mentions and emoji autocomplete
- **Paraglide** for i18n (German locale shipped)
- **Virtual scrolling** with O(1) scroll position recovery

### Runtime Units

Optional long-running processes (`chatto <unit>` or embedded under `chatto run`):
- `chatto exporter` — Prometheus metrics across the NATS cluster
- Voice call reconciler — elected leader reconciles LiveKit state with durable facts
- Asset cleanup worker — elected leader deletes physical binaries after `AssetDeletedEvent`
- Video processing — best-effort local ffmpeg for attachment transcoding

All elected workers use `MEMORY_CACHE` KV-based leases for leader election.

## Key Implementation Details

### Subject Design

Event subjects follow `evt.{aggregateType}.{aggregateId}.{eventType}`:
- `evt.room.{roomId}.message_posted`
- `evt.room.{roomId}.member_joined`
- `evt.user.{userId}.profile_updated`
- `evt.rbac.role.{roleName}.created`
- `evt.config.{configId}.server_updated`

The subject pattern is what makes OCC work — each aggregate gets its own lane with independent sequence numbering.

### Message Body/Event Split

Even within EVT, messages use two layers (ADR-011):
- **Public message facts** (`MessagePostedEvent`, `MessageEditedEvent`) — immutable, visible in timeline
- **Private body payloads** (`MessageBodyEvent`) — retention-controlled, can be securely deleted

This lets the system delete message content (for policy or GDPR) while preserving the immutable conversation metadata. A deliberate departure from textbook event sourcing.

### ID System

All entities use NanoID with type prefixes (ADR-022): `user_`, `room_`, `msg_`, `evt_`, `role_`, `grp_`, `link_`, `asset_`, `not_`, `bn_`, `call_`, `srv_`, `file_`, `upload_`, `token_`, `ext_`. Event IDs use NanoID independently of stream sequence numbers (ADR-026).

### Batch Atomicity for Multi-Aggregate Operations

When a `MoveRoomToGroup` needs to atomically remove a room from one group and add it to another, `Publisher.AppendBatch` uses NATS's `Nats-Batch-Id` protocol — the entries land contiguously in stream order, so projections observe both changes together (or neither).

### Shared Replay Fan-Out

ADR-033 initially had each projector running its own `OrderedConsumer`, but the duplicate delivery cost was significant. `ChattoCore.Run` now replays `evt.>` through one consumer, decodes once, and fans out. Each projection still maintains independent status, lag, failure, and wait state. This cuts startup replay work from `O(projections × events)` to `O(events)`.

## Design Decisions & Tradeoffs

### Optimized for: Deployment simplicity
Everything runs in one process. No database to configure. Single binary, single config file, single data directory. The embedded NATS makes this possible.

### Sacrificed: Ad-hoc query capability
No SQL, no joins. All reads go through projections. Operators can't run `SELECT * FROM messages WHERE...` — they use `chatto evt list` or the admin event-log API. Analytics and full-text search require dedicated projections or external systems.

### Bold choice: Event sourcing for a chat app
Event sourcing is typically associated with financial ledgers and audit systems, not chat. Chatto adopted it to solve a real problem: subject-index RAM growing 1:1 with message count under the old KV-based model. The event-sourcing switch cut subject cardinality from O(messages) to O(aggregates).

### Bold choice: NATS as the only data store
Most self-hosted chat apps use PostgreSQL or SQLite. Chatto puts everything in NATS — messages, users, config, RBAC, sessions, everything. This eliminates a deployment dependency but ties the project irreversibly to NATS's semantics.

### Bold choice: Per-process projections (not shared)
Each Chatto process maintains its own in-memory projections. Cross-process consistency is eventual. A read on process A might not yet see a write committed on process B — but in practice the delay is sub-millisecond. This avoids a single-point-of-failure shared projection service at the cost of eventual consistency.

### Technical debt acknowledged
- Snapshots are deferred (ADR-033) — cold starts replay the entire event stream. Acceptable in alpha, needs addressing before GA
- Migrations are per-aggregate and phased (ADR-035) — old CRUD+KV code coexists with event sourcing during migration
- Orphan-object gaps exist in some best-effort cleanup paths (call-key creation compensation, asset binary cleanup for legacy room-scoped facts)

## Comparison with Related Approaches

- **vs. Discord/Matrix**: Uses a traditional DB (PostgreSQL) + separate message broker. Chatto collapses them into NATS alone.
- **vs. Mattermost/Slack**: Multi-service architectures. Chatto is a single binary.
- **vs. The Log is the Agent (Nakajima)**: Both use event sourcing, but Chatto applies it to chat infrastructure while Nakajima applies it to agent state. Chatto's event sourcing is about durable domain state; Nakajima's is about agent memory.
- **vs. EventStoreDB-backed systems**: Chatto builds its own event sourcing framework (~600 lines) on NATS rather than using a dedicated event store. The framework is intentionally thin — no third-party event sourcing libraries.

## Documentation Quality

Exceptionally well-documented for an early-stage project:
- 48 ADRs with dates, context, decisions, and consequences
- 28 FDRs for per-feature design
- 910-line ARCHITECTURE.md with complete NATS resource inventory
- Glossary with canonical naming conventions
- AGENTS.md files at repo, directory, and subdirectory levels for LLM coding agent guidance
- `.agents/skills/` directory with 16 skills for the dev team's LLM agents
