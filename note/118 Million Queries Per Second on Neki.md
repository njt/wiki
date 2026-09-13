# 118 Million Queries Per Second on Neki

PlanetScale's launch benchmark for Neki, their Postgres-based sharded database newly in platform preview: 512 Postgres primaries fronted by 480 Neki routers, sustaining 118.5 million point-select queries per second over 1.22 PiB for 16 minutes, with throughput scaling linearly as shards grow from 5 to 50 to 512.

---

## Key Quotes

> "The benchmark was very simple. A single-shard point select, one row fetched per-query by primary key. No writes, joins, or cross-shard queries. The workload that each shard receives is isolated, in that there are no single queries that span multiple shards."

This is the load-bearing sentence of the whole piece, and PlanetScale deserves credit for putting it first. A point-select-by-primary-key workload is the *most* parallelizable thing a shared-nothing database can do: no cross-shard coordination, no distributed transactions, no lock contention across nodes. Linear scalability here is close to true by construction — the only things that can break it are the router tier, the network, and the load generator. Which is still an interesting claim to test, but it's a claim about the *plumbing*, not the *database semantics*.

> "Ten times the shards, ten times the throughput. Then ten times again. From 5 shards to 50 the per-shard rate held within 0.8%. At 512 the shards still had headroom, so we let the load generator use it and each shard settled at 231k QPS instead of 200k."

Two numbers hide in here worth pulling out. First, ~200k point-select QPS *per single Postgres primary* on one `r8g.16xlarge` — that's the per-node figure everything else multiplies out from, and it's the strongest evidence in the post that Postgres itself, not the sharding layer, is a serious OLTP engine. Second, the router fleet grew with the cluster (12 → 48 → 480), so the demonstration is really that *the routing tier doesn't become the bottleneck* at a nearly 1:1 router-to-shard ratio. What's not tested is whether the ratio can fall — 480 routers for 512 shards is a lot of proxy hardware to amortize.

> "Worth being clear about this run: the shards were primary-only with no replicas, the workload is read-only across queries ranging in complexity, and we did not fail over during the measured window."

The caveats paragraph is honest and damning in equal measure. No replicas means no replication lag to measure; no failover means the availability story — the actual hard part of a distributed database — is entirely deferred. And the 16-minute window tells you nothing about compaction debt, vacuum pressure, or any of the slow rot that kills sustained write-heavy Postgres fleets. (There's also a small internal tension: the methodology section says single-row point selects, while this caveat says queries "ranging in complexity" — presumably referring to the broader Neki test suite, but as written it muddies what the 118M number measures.)

## The Numbers

- 118,538,803 QPS sustained for 16 minutes; peak 118,747,267
- 512 shards (Postgres primary on `r8g.16xlarge` each), 480 routers (`8xlarge` each)
- p99 latency: 6.06ms at the router, 13.95ms at the client
- 67 errors/sec ≈ one query in 1.8 million
- 15.8M read IOPS fleet-wide; >2 Tb/s network

The error rate and the latency numbers are the quietly impressive part. Holding p99 under 14ms *at the client* while the fleet pushes 2 Tb/s through the routers means the whole path — load generator, router, shard, and back — stays in lockstep. Error rates of 1-in-1.8M at that volume suggest real backpressure and retry handling, not a demo that falls over when you look at it.

## Themes

**#tool** — Neki: Postgres primaries + a proprietary router tier, sold as a managed platform. The interesting design decision is keeping Postgres *stateful* per shard rather than disaggregating.

**#concept** — Linear horizontal scalability as a marketing genre. This is the database equivalent of a drag-strip run: maximally favorable conditions, one metric, nothing about the corners. Vendor launch benchmarks answer the question "can it go fast at all?" — useful, but the questions buyers actually care about (cost per QPS, write behavior, failover, multi-tenant fairness) are all in the fine print.

**#comparison** — The sharded-Postgres bet against disaggregated architectures. Neki's answer to scale is *more Postgres behind a router*; Aurora DSQL's is *no state anywhere*; Lakebase's is *stateless compute over externalized storage*. Same wire protocol, three different physics.

## How It Connects

This benchmark is the throughput datapoint for the sharded-Postgres side of a debate the wiki already holds. It stands against [[Aurora DSQL]]: DSQL's bet on disaggregated, stateless, coordination-free compute targets multi-region correctness and elasticity, while Neki shows the opposite architecture — fat stateful Postgres primaries, isolated shards — can post a raw single-region throughput number DSQL has never claimed. The comparison is unfair in both directions, which is what makes it interesting. It also nuances [[Lakebase and LTAP]], whose diagnosis is precisely that co-locating WAL and data files makes Postgres fragile at scale; Neki's run shows that fragility is a *failure-mode* problem, not a *throughput* problem — the monolith diagnosis and the 231k-QPS-per-primary number are both true.

In miniature, Neki is the managed-platform version of a pattern the wiki has seen as a single binary: [[PgDog]] combines pooling, load balancing, and sharding in front of Postgres, and Neki is that same shape at 512 shards and 480 routers. The PgDog write-up's gap between "managed proxy" and "full distributed middleware" is exactly the territory PlanetScale is staking out.

And read against [[Why Are Databases So Hard]], this post is a clean demonstration of the trilemma in action: the benchmark buys its performance number by parking correctness-under-failure (no replicas, no failover, no writes) at the curb. The speed-of-light ceiling that piece describes is exactly what the next benchmark — failover under load — will have to negotiate.

---
*Sources: [[raw/118-million-queries-per-second-on-neki]], [[summary/118-million-queries-per-second-on-neki]]*
*Last updated: 2026-09-13*
