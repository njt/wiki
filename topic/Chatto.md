# Chatto

A real-time chat application for teams and communities that runs on a single Go binary with no external database — everything is stored in NATS/JetStream using event sourcing with in-memory projections. It's a deliberately unconventional architecture: where most self-hosted chat apps use PostgreSQL or SQLite, Chatto uses an append-only event log and rebuilds all domain state as in-memory read models.

---

## Architecture

Chatto is an **event-sourced monolith** — one binary embeds a NATS server, a ConnectRPC HTTP API, a protobuf realtime WebSocket, and a compiled SvelteKit SPA frontend.

**Storage** uses three NATS-native tiers:

1. The **`EVT` stream** — a single append-only JetStream log for all durable domain facts (users, messages, rooms, RBAC, reactions, assets, etc.). Every write uses optimistic concurrency control. There is no non-OCC publish path.
2. **KV buckets** — `RUNTIME_STATE` (notifications, auth tokens, link-preview cache), `MEMORY_CACHE` (volatile presence, leader leases), `ENCRYPTION_KEYS` (KMS keys, excluded from backups).
3. **Object stores** — `SERVER_ASSETS` for binary data (avatars, attachments, branding) with optional S3 backend, and `ASSET_CACHE` for image transforms.

**Reads** come from 13 in-memory projections — Go data structures rebuilt from `EVT` at startup using a shared replay consumer that decodes each event once and fans it out. Read-your-writes is per-process (wait for local projector to advance). Cross-process consistency is eventual but sub-millisecond in practice.

**The API** is protobuf-first: ~100+ unary ConnectRPC endpoints at `/api/connect` plus a protobuf binary WebSocket at `/api/realtime` for live events. Service packages are tiered by stability: `chatto.auth.v1`, `chatto.discovery.v1`, `chatto.api.v1`, `chatto.admin.v1`, and `chatto.operator.v1` (Unix socket only).

**The frontend** is a SvelteKit SPA compiled into the Go binary. It uses optimistic UI with scoped provisional patches, a service worker for asset URL resolution, Paraglide for i18n, and virtual scrolling for message history.

## Key Techniques

- **OCC everywhere** — `events.Publisher` offers five publish methods, every one requiring an expected last subject sequence. The framework-level guarantee is that per-aggregate history is serialized with no race gaps. State-replacement mutations recompose on conflict; append-only mutations retry the same payload.

- **Shared replay fan-out** — Rather than each projector consuming `EVT` independently (which multiplied decode work by projection count), `ChattoCore.Run` replays through one ordered consumer, decodes each event once, and dispatches to matching projectors. Each projection keeps independent status and waiters.

- **Atomic multi-aggregate batches** — Operations like `MoveRoomToGroup` use NATS's `Nats-Batch-Id` protocol to land events contiguously in stream order, so cross-aggregate invariants ("every room belongs to exactly one group") are never observably broken.

- **Message body/event split** — Even within the event stream, `MessagePostedEvent` (immutable metadata) is separate from `MessageBodyEvent` (retention-controlled content). Message deletion erases body events without touching the conversation timeline. A deliberate departure from textbook event sourcing immutability.

- **Per-user DEK encryption with crypto-shredding** — Each user's messages are encrypted with their own Data Encryption Key managed through a KMS. Deleting a user's key renders all their content permanently unreadable, satisfying GDPR deletion without mutating the event stream.

- **Permission-only RBAC with hierarchical scope** — Permissions resolve through Server → Room Group → Room scope hierarchy, with deny winning at more specific scopes. The owner role is immutable with implicit permission override. Custom roles support reordering; direct user overrides handle exceptions.

- **Elected leader workers** — Background tasks (LiveKit reconciliation, asset cleanup) use `MEMORY_CACHE` KV-based leases for leader election across replicas, with idempotent handlers and retry-after-restart semantics.

## Design Decisions

**Optimized for deployment simplicity**: `chatto run` starts everything — NATS server, HTTP/WebSocket, and the SPA frontend. No Docker Compose required. Single config file, single data directory. The embedded NATS makes self-hosting trivial at the cost of process memory footprint.

**Sacrificed ad-hoc query capability**: No SQL, no joins. Operators can't run arbitrary queries against user or message data — they use `chatto evt list` or the admin event-log API. Analytics, search, and reporting require dedicated projections or external tooling.

**Tied to NATS irreversibly**: The project is deeply coupled to NATS/JetStream semantics — streams, KV buckets, object stores, subject-based routing. This eliminates database dependencies but means the operator must understand NATS, not SQL.

**Deferred snapshots**: Startup replays the entire `EVT` stream from scratch. This is acceptable in alpha but will need snapshot orchestration before GA. The `Projection.Snapshot()`/`Restore()` interface exists; only the orchestration is deferred.

**Per-process projections, not shared**: Each replica maintains its own in-memory state. This avoids a shared-projection single point of failure at the cost of eventual (sub-millisecond) cross-process consistency. Two API responses from different processes can momentarily disagree.

**Acknowledged orphan gaps**: Some best-effort cleanup paths (call-key creation compensation, legacy asset cleanup) have known orphan-object windows. These are documented in `docs/ARCHITECTURE.md`'s Durable Effect Inventory and are being addressed incrementally.

## Comparison Notes

Unlike most chat infrastructure which layers a relational database under a message broker, Chatto collapses both into NATS alone. This is architecturally closer to [[The Log is the Agent]]'s event-sourced agent state than to Discord's PostgreSQL + separate pub/sub, but applied to a fundamentally different domain — durable chat infrastructure rather than agent memory.

The event-sourcing framework itself is notably thin (~600 lines of Go, no third-party libraries) compared to the tooling that typically accompanies event-sourced systems in the .NET/Java ecosystem. The architects read `looplab/eventhorizon` and `blinkinglight/bee` for vocabulary but built their own minimal package shaped to Chatto's specific needs.

Where [[State System]] proposes evidence-first commits and model/code boundaries for organizational state, Chatto implements a concrete version of the same idea at the application level — every state change is an append-only event, every read is a projection, and code owns the integrity boundary.

#tool #project #chat #eventsourcing #nats #selfhosted

---
*Sources: [[raw/chatto]]*
*Last updated: 2026-07-11*
