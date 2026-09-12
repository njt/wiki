---
url: https://github.com/chattocorp/chatto
title: "Chatto"
author: chattocorp (Hendrik Mans)
date_fetched: 2026-07-11
date_published: 2026-03-01
topics:
  - databases-and-data
---

Chatto is a real-time chat application for teams and communities, built as a
single Go binary with a SvelteKit frontend. It uses NATS/JetStream as its sole
persistent data store — no PostgreSQL, no Redis, no MongoDB. The project is
licensed AGPL-3.0-or-later with Apache-2.0 exceptions for the frontend and
integration surfaces.

The architecture is event-sourced: all durable domain facts are appended as
protobuf-encoded events to a JetStream stream (`EVT`), with optimistic
concurrency control on every write. Thirteen in-memory projections consume the
event stream and maintain current state per-process. At startup, a shared
replay fans decoded events out to all projectors, avoiding duplicate work.

The API is protobuf-first, with ~100+ ConnectRPC endpoints across five service
packages and a real-time WebSocket layer using protobuf binary frames.
Permission-only RBAC with hierarchical scope resolution governs access.
Per-user encryption keys with crypto-shredding satisfy GDPR deletion
requirements without mutating the immutable event log.

Key design choices include multi-instance clients connecting to multiple
servers simultaneously, optimistic UI with scoped provisional patches, and
elected leader workers for voice-call reconciliation, asset cleanup, and video
processing. The project is unusually well-documented for its stage: 48 ADRs, 28
FDRs, a 910-line architecture document, and AGENTS.md guidance for LLM coding
agents.

The event-sourcing model was adopted to escape subject-index RAM growth under
the prior KV-based approach, cutting subject cardinality from O(messages) to
O(aggregates). Acknowledged technical debt includes deferred snapshots (cold
starts replay the full event stream) and per-aggregate phased migration from
the old CRUD code.
