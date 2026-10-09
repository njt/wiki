# Durable Actors — Open-Source Durable Objects

Durable Actors (TerseAI) is an open-source, self-hostable take on Cloudflare Durable Objects: stateful serverless actors — classes with `@Persisted` fields and serialized execution — with a Rust runtime (~25K lines), TypeScript and Python SDKs with generated type-safe clients, a Postgres-backed control plane, and GCP self-hosting on GKE Sandbox with gVisor. Its target workloads are exactly where agents now live: chat, collaborative documents, and agent swarms, where many concurrent callers (humans and agents) must safely share one evolving stateful object.

---

## Architecture

Three planes, one binary (v0.7.16, edition 2024):

- **Control plane** (`src/control_plane/`): placement, invocation routing, JWT issuance/verification (`auth.rs`, `issuer.rs`), contract registration, and the socket directory/gateway that tickets WebSocket connections. Oddly, its gRPC protocol is a single RPC — `Execute(ControlPlaneRequest{command_json}) → ControlPlaneReply{reply_json}` (`proto/durable_actors.proto`) — i.e. a JSON-over-gRPC command envelope rather than typed RPCs.
- **Actor hosts** (`src/host/`): the per-actor runtime. `actor_host.rs` is a command-loop over an mpsc channel (capacity 256) with `ActorQueues` serializing requests; `actor_runtime.rs` defines the `ActorStorage` trait (`acquire_actor`, `verify_actor_ownership(host, epoch)`, `prepare_state_write(actor, host_id, owner_epoch, expected_version)`). Hosts hold a lease renewed by `lease_maintenance.rs`, which enforces `renew_every < lease_duration` and gives a 2s shutdown grace on lost leases.
- **Durability layer** (`src/bucket/`, `src/litestream.rs`, `src/state_log.rs`): actor state is a SQLite database. `StateSnapshot { state_version, owner_epoch, request_id, sqlite: SqliteSnapshot { txid, parent, files } }` — snapshots form a parent-linked chain of terse-ltx (Litestream-format) files written to GCS. A fast path ("Rapid", `src/bucket/rapid/`) is a log-structured segment store with 60s rotation, a checkpoint every 32 records, manifest-tracked epochs, and compaction via the embedded `terse-litestream` fork.

Placement (`src/placement.rs`) maps an `ActorStorageKey` to an `ObjectPlacement` carrying `owner`, `owner_epoch`, `home_region`, `state_version`. Sandboxes (`src/sandbox/`) are GKE Sandbox pods (gVisor RuntimeClass) kept warm in a Postgres-tracked spare pool (`spare.rs`, `pool/replenishment.rs`) so the "provider boot gap" — which the code carefully distinguishes from total host startup time — is absorbed before an actor needs a host.

SDK side: the TS SDK ships a TypeScript compiler (`sdk/src/compiler/`) that parses actor classes and emits diagnostics; `@Interleave` must be a public, non-static, async instance method with a body or the build fails. Python mirrors this with `persisted()`/`ephemeral()` field declarations (`sdk-python/src/durable_actors/actor.py`).

## Key techniques

- **Fenced single-writer ownership**: every state write names `host_id + owner_epoch + expected_version`; `verify_actor_ownership` plus epoch fencing means a partitioned old host cannot commit stale state after its lease was reassigned. Classic fencing, applied at snapshot granularity.
- **SQLite as the actor's memory, LTX as the delta format**: instead of diffing JS objects like Durable Objects do, each commit is a real SQLite transaction (txid + parent-linked LTX files). Recovery (`bucket/rapid/checkpoint.rs`) walks the snapshot chain and counts `dependency_gets` vs `checkpoint_gets` — restore is a measured, logged operation.
- **Opt-in reentrancy via compile-time analysis**: the default is serialized execution; `@Interleave` lets other calls run across awaits. Rather than runtime magic, the TS compiler validates the annotation statically and the reentrancy feature file (`compiler/features/reentrancy.ts`) is ~60 lines.
- **Honest observability vocabulary**: `CONTEXT.md` bans the word "trace". Latency milestones are process-local monotonic offsets — six event families (client invocation, target resolution, host provisioning, host startup, host invocation, state write) that share request/host/session IDs but explicitly do not form a distributed trace. A rare case of docs forbidding the overclaiming term.
- **Spare-pool warm sandboxes**: pre-provisioned gVisor pods are claimed by image-ref + region + config key from Postgres, with a 120s claim reservation and 60s readiness wait — cold-start mitigation without a bespoke scheduler.

## Design decisions

The headline trade-off is **durability over cleverness**: state is a real database engine with real replication, not an event-sourced replay log. That directly contrasts with Temporal-style designs — cf. [[Durable Execution Without History Replay]], where Temporal's recovery cost grows with accumulated history while Trigora's TCC restores a live continuation in flat ~0.6–0.9 ms. Durable Actors pays storage (snapshots + LTX deltas in GCS) to make recovery a restore, not a replay; its checkpoint-every-32-records Rapid log is a middle path.

Second, **Postgres + GCS as the substrate**: the control plane's state of record is Postgres, actor state lives in GCS, and the regional design leans on Google Cloud (Workload Identity, Cloud SQL) as the supported platform. That is the opposite of Cloudflare's vertically-integrated edge; you get no lock-in but you inherit a GKE/Cloud SQL operations bill the README's one-command dev experience doesn't hint at.

Third, **JSON-in-gRPC**: the control-plane protocol is one RPC carrying opaque JSON. Pragmatic for evolving contracts and cross-language SDKs, but it forfeits proto's schema discipline exactly where a multi-party distributed contract would most benefit.

## Comparison notes

- Against **Cloudflare Durable Objects**, the pitch is explicit: no vendor lock-in, no 128 KB memory limit, observability built in. DO's strength (co-locating compute with the edge) becomes this project's cost: hosts are gVisor pods on GKE, so an activation crosses a real network and a sandbox pool.
- [[SQLite Is All You Need for Durable Workflows]] argued SQLite is sufficient substrate for durable state; this repo is arguably the strongest existence proof in the wiki — an entire actor platform whose per-actor memory is a SQLite file replicated as LTX deltas, matching [[SQLite Is All You Need]]'s production case.
- [[The Log — Unifying Abstraction for Real-Time Data]] is the intellectual backdrop for the Rapid segment store and Litestream replication; here the log is a durability mechanism under an object model, not the public API.
- [[Cloudflare K2 — Serverless Event Streams]] makes the same "object storage as the replication substrate" bet for event streams; Durable Actors makes it for per-actor snapshots — both offload consensus to GCS instead of running quorum protocols.
- For agent builders, this is infrastructure rather than an agent harness — unlike [[Unreal Agent]] or [[Durable Execution Without History Replay]], which solve durability for the agent loop itself, Durable Actors solves it for the stateful objects agents serve and share.

Tags: #tool #project #distributed-systems #actors #sqlite #sandboxing

---
*Sources: [[raw/durable-actors]], [[summary/durable-actors]]*
*Last updated: 2026-10-09*
