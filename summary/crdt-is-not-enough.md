---
url: https://loro.dev/blog/crdt-is-not-enough
title: "CRDTs are not enough: From CRDTs to a local-first sync engine"
author: Zixuan Chen
date_fetched: 2026-11-13
date_published: 2026-09-21
topics:
  - distributed-systems
  - agent-orchestration
---

Zixuan Chen (Loro co-author) reflects on building Lody, a local-first control plane for team AI agents, and what building it taught him that CRDT libraries don't solve. Local-first promises instant open, offline, multi-device sync, backend swappability, and data ownership — with Git as the classic shape and GitHub as the hosted convenience layer.

The core argument: CRDTs guarantee eventual convergence of individual documents, but a real product needs much more — document discovery and lazy loading (solved with a new directory-syncing CRDT called Flock, iterated through JSON, an over-engineered binary V1, and a columnar V2), local storage and full-text search under end-to-end encryption (unsolved), prefetch policy, and consistent visibility across documents. His opening example: a conversation index syncs before the message body, so a user gets a new-message notification for a message that isn't there yet. Cross-document visibility is not a convergence property.

On the backend, Cloudflare Durable Objects proved too expensive (per-connection billing, SQLite write costs inside hibernating objects) for CRDT version-diff workloads. They settled on an append-only log — ElectricSQL's Durable Streams protocol — where the server is a dumb relay: offsets for catch-up, client-published snapshots for compaction, an ephemeral channel for presence, and append-CAS for E2E encryption. Raft groups per stream behind a gateway; hot data in memory, cold in object storage.

Remaining challenges: authentication (still centralized), E2E encryption, cross-document visibility, and UX for decentralized systems.
