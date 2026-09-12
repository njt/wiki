---
url: https://github.com/denoland/celld
title: celld
author: Deno Land Inc.
date_fetched: 2026-08-06
topics:
  - databases-and-data
---

# celld — Summary

celld is an open-source daemon (Rust, Apache 2.0) that runs Cloudflare Workers and
Durable Objects on your own infrastructure. Each Durable Object is its own SQLite
database, continuously replicated to an S3-compatible bucket you own. Nodes
coordinate through that bucket alone using object-storage Compare-And-Swap — no
control plane, no consensus protocol, no leader election.

The architecture has three crates: `celld` (the executable — V8 runtime, HTTP
server, S3 client, alarm/wake system), `celld-logic` (a pure, deterministic
state machine with zero dependencies that makes every lifecycle decision), and
`celld-ltx` (a vendored Rust port of Litestream for SQLite WAL replication to
object storage). The boundary between logic and execution is absolute: the logic
crate has no async, no I/O, no clocks, and no randomness, making the entire
system replayable and testable under deterministic simulation.

Every cell write is fenced by an output gate: the HTTP response is held until the
replicator proves the write durable (RPO=0), so celld never acknowledges a write
it could lose. Idle cells hibernate to near-zero cost, with SQLite snapshots
stored in the bucket for cold restart. Peer-to-peer requests are
HMAC-authenticated and replay-protected; TLS is deliberately excluded from the
peer protocol, with the design calling for a private network or encrypted overlay
instead.

The project is in alpha (v0.1.0). Pull requests are disabled; contributions
arrive via `git format-patch` email to the maintainer.
