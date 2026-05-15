# PgDog

A PostgreSQL proxy that combines connection pooling, load balancing, and sharding into a single binary — deployed as a drop-in replacement requiring zero application code changes. Targets the gap between managed proxies like RDS Proxy and full distributed middleware like Citus. #tool

---

## What It Does

Three capabilities in one executable that sits between your application and Postgres:

1. **Connection pooler** — Multiplexes 100K+ clients onto a small number of Postgres connections. Preserves session state (SET, advisory locks, LISTEN/NOTIFY) without connection pinning. Claims 50K+ transactions/s per thread.

2. **Load balancer** — Routes reads to replicas, writes to primary. Detects replication lag, hardware failure, and primary failover automatically. Uses an internal SQL parser for read/write splitting rather than relying on client-side hints.

3. **Distributed database** — Extracts shard keys from queries and routes to the correct shard. Cross-shard queries (without a shard key) fan out and aggregate via scatter/gather. Cross-shard writes use two-phase commit. DDL propagates automatically to all shards.

Deployable via Docker, Helm (Kubernetes), or standalone binary. Configuration via `pgdog.toml`.

---

## Key Quotes

> "We run thousands of pods and couldn't have scaled this far without PgDog."
> — Sam R., Modal

Modal runs serverless containers at scale — they'd hit connection limits quickly without something like this. The fact they chose PgDog over AWS-native tooling says something.

> "Moved complicated geospatial sharding logic from the application tier into a simple config."
> — Hutch Ingold, Earth Genome (4 billion rows)

The "drop the complexity into config, not code" pitch. When it works, it's compelling. When it doesn't, you're debugging a proxy you don't control.

> "Since switching to PgDog, we've only had 100% uptime."
> — Mike Matkiwsky, TripStack

The kind of quote that either means the product is excellent or the previous setup was held together with string. Probably both.

---

## Critical Analysis

**What's genuinely interesting:** PgDog is betting that database scaling should be infrastructure, not application code. The "no app changes" promise is the right one — it's the same bet that made managed databases win over hand-rolled sharding. If they can actually deliver cross-shard transactions, integer primary keys (no UUID migration), and schema propagation, they've solved the hard parts that usually force application-level workarounds.

**What gives me pause:** The sharding model is range-based (tenants 0–33%, 34–66%, 67–100%). Range sharding creates hot spots the moment your key distribution isn't uniform. The page doesn't mention hash sharding or resharding — which are table stakes for production sharding. If you can't rebalance shards online, you're buying a future migration.

**The connection pooling claim** — 50K transactions/s per thread with session state preserved — is ambitious. Most poolers trade session features for throughput. If PgDog delivers both, it's a genuine advance over pgbouncer. If not, it's marketing.

**The competitive landscape:** PgBouncer handles pooling, HAProxy handles load balancing, and Citus handles sharding. PgDog's thesis is that running all three from one binary with one config is better than stitching together three tools. For teams that don't have the ops bandwidth to run the full stack, that's a real value proposition. For teams that already have the stack tuned, the switching cost is high.

**The business:** Open source with an Enterprise tier, 4.3k GitHub stars, logos from Coinbase and Ramp. Not a fly-by-night project. But the pricing page is empty — "talk to us" pricing at this stage means either "we haven't figured out monetization" or "it's expensive and we want to qualify you first."

**Worth watching if:** You're running Postgres at a scale where connection pooling alone isn't enough, but you don't want to adopt Citus or move to a distributed-by-design database like CockroachDB. Also worth watching if you believe (as I do) that proxy-layer infrastructure beats application-layer workarounds for data problems.

---

*Sources: [[raw/pgdog]]*
*Last updated: 2026-05-15*
