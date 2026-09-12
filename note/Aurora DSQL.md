# Aurora DSQL

AWS's serverless, multi-region active-active SQL database that disaggregates compute, storage, and transaction coordination into independently scalable services — PostgreSQL-compatible at the wire level but architecturally alien underneath. Defers all coordination to commit time via distributed adjudicators, making reads free and writes optimistic.

---

## What It Is

Aurora DSQL is a cloud-native OLTP database designed from scratch around three architectural bets:

**Disaggregate everything.** Compute (Firecracker MicroVMs), storage, and transaction coordination are independent, horizontally scalable services. Query processors carry no local state — they're disposable, stateless, and can scale from zero to however many you need. This is the opposite of the "co-locate everything in Postgres" philosophy — DSQL argues that disaggregation is the only path to multi-region scale.

**Defer coordination to commit time.** Reads are coordination-free because MVCC timestamps tell you what you're allowed to see without asking anyone. Writes accumulate optimistically, then a distributed adjudicator decides at commit time whether they conflict. This means individual SQL statements carry no cross-region latency penalty — the coordination tax is paid once per transaction, not once per statement.

**Physical clocks as the foundation.** The entire edifice rests on precise, synchronized timestamps. MVCC visibility, conflict detection, and replication ordering all depend on clocks. This is a bet that AWS's time infrastructure (presumably the Amazon Time Sync Service) is good enough to serve as a database's spinal column. If clocks drift, correctness drifts with them.

## Key Architecture

The system has three tiers: stateless query processors (PostgreSQL-compatible, running in Firecracker), a distributed storage layer, and a transaction coordination service (adjudicators + Journal). Query processors accept connections, parse SQL, execute reads against storage using MVCC snapshots, and buffer writes locally. At commit time, the adjudicator checks for conflicts, and if the transaction passes, the Journal replicates the commit decision across regions.

The cross-region write path looks like: client → local query processor (buffers writes) → commit request → adjudicator (conflict check) → Journal (replicate decision) → ack to client. Reads never leave the local query processor unless they explicitly request the latest committed state.

## Critical Analysis

**Disaggregation is the opposite of the "Just Use Postgres" thesis, and both are correct.** Kraft & Li ([[Postgres Transactions Are a Distributed Systems Superpower]]) argue that co-locating workflow state with application data eliminates idempotency bugs and outbox infrastructure. DSQL argues the opposite: separate everything, scale each layer independently, and push coordination to a specialized service. The resolution isn't one-is-right — it's that co-location wins at single-region scale (the 95% case) and disaggregation wins at global scale (the 5% case that AWS has to solve because their customers demand it). DSQL is what you build when "just use Postgres" stops working at continent scale.

**Commit-time coordination is architecturally elegant but operationally risky.** Making reads free and writes optimistic means the system feels fast until commit time — then the adjudicator becomes the bottleneck. Under high contention (many writers to the same rows), the optimistic concurrency model will abort more transactions, and throughput will degrade. The paper presumably quantifies this, but the abstract's "millions of transactions per second" claim is almost certainly for low-contention workloads. This is the same tradeoff every OCC system makes, just at cloud scale.

**Clock-based MVCC is a bet on infrastructure quality.** Google Spanner made this bet with TrueTime. DSQL makes it with (presumably) Amazon's time sync. The difference: Spanner's TrueTime exposes uncertainty as an API (`[earliest, latest]`), letting the database reason about clock error explicitly. DSQL's abstract doesn't mention an uncertainty interval — it just says "precise timestamps." If that precision is good enough to paper over the uncertainty, fine. If not, the edge cases are ugly and quiet.

**PostgreSQL compatibility is a strategic moat play.** DSQL speaks the Postgres wire protocol but shares no code with Postgres. This is the same pattern as [[DocumentDB]] (MongoDB-on-Postgres in reverse), [[Lakebase and LTAP]] (stateless Postgres), and Aurora PostgreSQL itself. The Postgres API has become the SQL standard that actually matters — not ISO SQL, but "whatever psql speaks." AWS is betting that customers want the Postgres interface without the Postgres architecture, and they're willing to pay AWS prices for it.

**Serverless from zero is a genuinely hard problem at database scale.** Scaling compute from zero means cold-starting Firecracker MicroVMs, attaching them to storage, and routing traffic — all before the first query times out. Scaling storage from zero means provisioning capacity on demand without data loss. DSQL apparently does both. If the paper describes the cold-start path in detail, that's the most operationally interesting section.

**The real innovation is making this survivable.** Multi-region active-active with strong consistency is the hardest problem in distributed databases. Most systems settle for active-passive (easier), eventual consistency (cheaper), or single-region (simpler). DSQL claims all three: active-active, strong consistency, and ACID. If the paper's evaluation section holds up, this is a genuine advance — not a new idea, but a working implementation of ideas that have been in the literature for years without shipping.

**What to watch for when the full paper is available:** (1) How does the adjudicator handle contention hotspots? (2) What's the clock synchronization mechanism and its failure mode? (3) What happens during a network partition between regions? (4) How does membership change work — adding or removing regions from a running database? (5) What are the latency numbers at P99 and P99.9, not just P50? The abstract says "millions of TPS" but the operational question is what happens at the tail.

---

## Key Themes

#database #distributed-systems #AWS #PostgreSQL #serverless #consensus #OLTP #MVCC #concept

## Related Pages

- [[Databases and Data]] — Hub page; DSQL is the missing "cloud-native disaggregated OLTP" entry
- [[Distributed Systems]] — Hub page; DSQL addresses several "What's Missing" items (consensus, partition tolerance, distributed transactions)
- [[Postgres Transactions Are a Distributed Systems Superpower]] — The co-location argument that DSQL inverts
- [[Meerkat — QuePaxa Consensus at Cloudflare]] — Another consensus system making different architectural bets (leader-optional vs. adjudicator-mediated)
- [[Lakebase and LTAP]] — Databricks' stateless Postgres architecture; another bet on disaggregation
- [[DocumentDB]] — MongoDB wire protocol on Postgres; the same "compatible but different" strategy
- [[DocDB — Stripe's Zero-Downtime Database]] — Cloud-scale database design at a different company; the operational concerns converge
- [[Streambed]] — Postgres CDC infrastructure; DSQL presumably needs its own CDC story
- [[ARIES — Write-Ahead Logging Recovery]] — The recovery architecture that DSQL's Journal presumably reimagines for the cloud era
- [[Write Snapshot Isolation]] — The MVCC correctness argument; DSQL's isolation guarantees warrant comparison
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — The fallacies that make multi-region consistency hard
- [[SDPD — Systems Design Police Department]] — The failure modes ("Split Brain", "Conflicting Orders") that DSQL's architecture must handle
- [[SQLite Is All You Need]] — The opposite end of the spectrum: single-machine SQLite vs. global-scale DSQL
- [[Constraint Decay]] — The finding that databases are the primary failure driver for coding agents; relevant to DSQL as an agent-facing database
- [[Aurora DSQL — Murat Demirbas' Insider Review]] — Former DSQL engineer's candid architecture review with tradeoffs the paper omits

---

*Sources: [[raw/aurora-dsql]], [[raw/aurora-dsql-murat-demirbas]]*
*Last updated: 2026-08-01*
