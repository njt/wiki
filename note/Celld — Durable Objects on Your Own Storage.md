# Celld — Durable Objects on Your Own Storage

Thomas Lockney's AI-assisted book on celld v0.6.0, Deno's self-hosted implementation of the Cloudflare Durable Objects model: a named, single-threaded object with its own SQLite, where the only coordinator is an object-storage bucket you own. The book's real contribution is showing how far you can get with no membership protocol, no failure detector, and no consensus service — just conditional writes, epochs, and an output gate tied to durability proofs.

---

## The architecture in one paragraph

Each **cell** is a Durable Object: one thread, one private SQLite database, states of resident/hibernated/inactive. A **fleet** is any set of nodes sharing one bucket. Cell ownership is a record in the bucket claimed by conditional create or compare-and-swap; every activation advances the **epoch**, and SQLite data replicates as Litestream LTX segments under an epoch-qualified prefix. Durability is explicit and mode-dependent: one node waits for the bucket every write (**bucket proof**); two or more nodes acknowledge once one or two followers have the data on disk (**fleet proof**), with the bucket upload trailing. The **output gate** holds every response until a proof covers it — no caller ever sees a success for an unpersisted write. A node that cannot renew its lease **self-fences**: it stops its cells, logs `SELF-FENCE:`, and exits code 3.

## Key quotes

> "There is no membership protocol, no failure detector, and no consensus service."

The thesis. The bucket's four required properties — conditional create, conditional overwrite (CAS), read-after-write consistency, exact ranged reads — are enough to build placement, fencing, and RPO=0 replication. This is the same bet as the object-store-as-log school: pay one round trip of latency, delete an entire coordination subsystem.

> "A partitioned node can commit locally and replicate into its superseded prefix, but the ownership read reveals the new owner, so the write is not acknowledged."

The fencing trick that makes clock-freedom real: after a bucket proof, celld re-reads the ownership record and only acknowledges if it still names this node at this epoch. A paused process or skewed clock can't pass; a stale node writes into the garbage prefix of a dead epoch. Elegantly, the same record that grants ownership also revokes it.

> "celld does not retry a call after transmission begins, because it keeps no replay copy of the body... make operations idempotent at the application layer."

An honest limitation pushed upward, in classic distributed-systems style: the runtime refuses to guess, refuses ambiguous retries, and hands the at-least-once problem to the application with stable operation IDs. Compare with the queue-philosophy in [[Queues Don't Fix Overload]] — treat the cause, don't paper over it.

> "keep the constructor light... restore state from SQLite storage inside the handler, not in the constructor."

The hibernation economy: a cell that wakes per-message re-runs its constructor every time, and an idle fleet costs "almost nothing" — roughly $0.05/month per resident cell on a 1,000-cells-per-8GB-node budget. This is the pricing model the whole entity-per-user design lives or dies by.

## What the book teaches beyond celld

Part I is the best compact survey I've seen of the two ideas Durable Objects fuse: the **actor** (Hewitt 1973 → Erlang's let-it-crash → Akka → Orleans' virtual actors) and **durable execution** (replay, deterministic control flow, idempotent side effects, five engines from Temporal to DBOS). The synthesis is crisp: a Durable Object is a virtual actor *with colocated storage and gates* — the exact property an Orleans grain lacks, and the reason its consistency bugs live in the database round trip. The entity/process distinction lands as a design rule: model what persists as an object, what finishes as a workflow, never force one shape into the other.

The compatibility chapter is a masterclass in honesty as a spec: every unavailable feature must "fail loudly, at deploy or first use; a silent gap is a bug." BroadcastChannel is defined but throws. Containers are *Experimental* and fenced with nftables. Facets get their own SQLite and replication stream — with the sharp consequence that a facet write does not roll back with a root transaction.

## Analysis

This is the second serious open-source "Durable Objects on your own bucket" runtime to land in the wiki, and the convergence with [[Durable Actors — Open-Source Durable Objects]] is the interesting fact: independent teams reaching nearly the same design — SQLite per actor, LTX/segment replication to object storage, owner-epoch fencing, RPO=0 as an explicit promise. When two attempts independently refuse to build Raft, that's a signal the object store really has become the coordination substrate (see [[The Log — Unifying Abstraction for Real-Time Data]]).

Two caveats worth holding onto. First, the book is AI-generated from curated materials — the colophon says so plainly and asks readers to verify load-bearing claims against their release; the executed labs are the strongest defense, but the book is a guide, not ground truth. Second, the qualified-bucket table is the real product constraint hiding under the "any S3-compatible bucket" marketing: conditional writes and exact ranged reads exclude B2, Hetzner, and Spaces, and MinIO is merely "passes test, not qualified." Whoever holds the bucket credentials controls the fleet — a self-hosting trade, not a free lunch.

**Tags:** #concept #tool #pattern

**Related:**
- [[Durable Actors — Open-Source Durable Objects]] — the near-twin project (Rust, gVisor, GCS): celld's book strengthens its case with a fully specified proof mechanism and reveals how much of the design is convergent evolution.
- [[Durable Execution Without History Replay]] — celld's Workflows use classic replay discipline; TCC's continuation-commit proposal is the alternative celld explicitly does not take, making this a useful contrast.
- [[SQLite Is All You Need for Durable Workflows]] — celld is the strongest production-grade confirmation of that thesis: SQLite in every cell, WAL shipped to the bucket, alarms and queues built on it.
- [[Cloudflare K2 — Serverless Event Streams]] — same object-store-as-authority bet at Cloudflare itself; celld shows the pattern portable to hardware you own.

---
*Sources: [[raw/celld-book]], [[summary/celld-book]]*
*Last updated: 2026-10-09*
