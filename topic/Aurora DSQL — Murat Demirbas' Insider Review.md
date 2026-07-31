# Aurora DSQL — Murat Demirbas' Insider Review

A former AWS engineer who helped design Aurora DSQL reads the published paper and adds the architecture review, tradeoffs, and team dynamics the paper leaves out. The result: a rare insider's account of what it took to blow up every component of a traditional database into an independent, horizontally scalable service — and what still hurts.

---

## What the Paper Says vs. What Murat Adds

The arXiv paper ([[raw/aurora-dsql]]) describes the architecture. Murat's post describes the *thinking behind* the architecture, because he was in the room for 2022–2023 when the decisions were made.

**The one-sentence summary he gives is better than anything in the paper:**

> "We took a traditional monolithic database and blew out every single component into an independent, horizontally scalable service."

The five-component decomposition — Query Processors, Storage Nodes, Adjudicators, Journals, Crossbars — is the key innovation. Each scales independently. Each can fail independently. Each is a service with its own team, its own operational concerns, and its own scaling properties.

## The Architectural Bets, With Insider Commentary

### Bet 1: Synchronized Clocks Replace Coordination

DSQL bets its correctness on AWS TimeSync, the in-house clock synchronization service. A Query Processor checks its local clock and asks storage for data from that exact microsecond — no leader election, no consensus round, no cross-region ping. This is what makes 0-RTT reads possible.

Murat notes this is the Spanner bet (TrueTime), but he doesn't mention uncertainty intervals — a conspicuous omission. TrueTime exposes `[earliest, latest]` as an explicit API, forcing applications to reason about clock error. DSQL appears to paper over the gap. If TimeSync is good enough that the gap never matters in practice, that's an engineering win hiding a theoretical risk.

### Bet 2: Optimistic Concurrency + Snapshot Isolation

OCC normally means high abort rates under contention. DSQL's trick: pair OCC with MVCC under Snapshot Isolation so that readers look at a past snapshot, making read-write conflicts structurally impossible. For write-skew (the classic SI vulnerability), customers are told to use `FOR UPDATE` and design schemas that create write-write conflicts at the business-logic level.

This is honest. It's not "we solved write-skew" — it's "we gave you the tools to avoid it, and if you don't use them, the database will happily let you build a broken application." That's a defensible position for a production database, but it puts more burden on application developers than most SQL databases do.

### Bet 3: Eventual Consistency Is Dead (Long Live Linearizability)

Murat quotes the paper's position directly: developers "simply cannot write correct business logic on eventually consistent systems." This is a strong claim and he doesn't soften it — he clarifies it. Linearizability governs single-object real-time ordering. Snapshot Isolation governs multi-object transaction visibility. DSQL gives you both, and they control different things. This is the kind of precision a distributed systems researcher brings that a product blog post wouldn't.

### Bet 4: Guardrails as Architecture, Not Afterthought

Transactions capped at 3,000 rows and 10 MiB. The justification is Little's Law: smaller transactions → predictable, stable tail latency. This is a production database making a hard trade: we'd rather tell you "no" at design time than surprise you with a 10-second commit at 3 AM. Most databases add limits after the fact, as an ops band-aid. DSQL builds them into the architecture.

### Bet 5: Linearized 2PC Without the 2PC Tax

Traditional 2PC over WAN costs 2 RTTs. DSQL's adjudicators vote, but only the leader writes the final commit to its Journal — inspired by the Warp protocol. This avoids coordinating across multiple logs. It's the kind of optimization that only matters at global scale but is transformative when you need it.

## The Payoff: What You Actually Get

| Capability | Mechanism | Latency |
|---|---|---|
| Consistent reads | Local clock timestamp, MVCC | 0 RTT |
| Writes (single or multi-row) | Optimistic buffering, commit-time adjudication | 1–1.5 RTT |
| Scaling compute | Spin up/down Firecracker QP MicroVMs | Cold start |
| Scaling storage | Add shards | Online |
| Scaling coordination | Tune adjudicator range placement | Operational |

The 0-RTT read is the headline. OLTP workloads are read-heavy. Most writes are reads first (UPDATEs, INSERTs with unique index checks). Making reads free changes the economics of every query pattern.

## Critical Analysis: What Murat Is Actually Saying

**"This felt too easy" is the most interesting sentence in the post.** Building a novel global production database should be one of the hardest things in software engineering. Murat says it wasn't — credit to great upfront design, aggressive reuse of existing AWS infrastructure (PostgreSQL engine, internal Journal service), and lessons from prior failures (JournalDB, QLDB). The implicit claim: AWS's internal infrastructure is now good enough that building a global database is an *integration* problem, not a *research* problem. If true, that's a bigger deal than DSQL itself.

**The absences are as revealing as the content.** Murat doesn't discuss: (1) cold-start latency for Firecracker MicroVMs at database scale, (2) what happens during a network partition between regions, (3) how adjudicator membership changes when adding/removing regions, (4) P99.9 latency numbers, (5) the clock synchronization failure mode. These are the questions that separate "architecturally elegant" from "operationally survivable."

**The Marc Brooker tribute is genuine but also tells you something.** Murat calls Brooker "technically brilliant" with "amazing leadership." He describes weekly whiteboard sessions with principal engineers as genuinely enjoyable. Then he notes he wasn't there for the final year. The subtext: the hardest part of building a global database isn't the design — it's the grind of productionizing it, and Murat wasn't around for that part. The paper's Lessons Learned section (friction with foreign keys, high-locality sequences) hints at what that year was like.

**"Eventually consistent" is a slur now.** The paper's position that developers cannot write correct business logic on eventually consistent systems is a shot across the bow of DynamoDB, Cassandra, and every other AP system. AWS is now on both sides of this bet: DynamoDB for "we'll handle the complexity" and DSQL for "you shouldn't have to." The internal tension at AWS must be fascinating.

**OCC's dirty secret is the late abort.** Under contention, transactions abort at commit time — after you've already paid the WAN latency for the commit round-trip. A traditional database with pessimistic locking would have told you upfront that the row was locked. DSQL tells you after you've spent the money. For cross-region writes to hot keys, this turns into paying WAN latency twice (original attempt + retry), making tail latency on contended workloads genuinely painful.

## Bottom Line

DSQL is what you build when "just use Postgres" ([[Postgres Transactions Are a Distributed Systems Superpower]]) stops working at continent scale. It's the disaggregation thesis taken to its logical extreme, backed by AWS infrastructure that makes the hard parts look easy. Whether the clock bet holds, whether the contention story is survivable at production scale, and whether the operational gaps get filled in — those are the questions that separate a great paper from a great database.

---

## Key Themes

#distributed-systems #database #AWS #OLTP #MVCC #consensus #linearizability #optimistic-concurrency #concept #paper-review

## Related Pages

- [[Aurora DSQL]] — The topic page for the paper itself; this review covers what the paper omits
- [[Distributed Systems]] — Hub page; DSQL is a working implementation of ideas that have been in the literature for years
- [[Databases and Data]] — Hub page; the disaggregated OLTP entry in the database landscape
- [[Postgres Transactions Are a Distributed Systems Superpower]] — The co-location thesis that DSQL inverts
- [[Lakebase and LTAP]] — Databricks' bet on stateless Postgres; another disaggregation play
- [[Meerkat — QuePaxa Consensus at Cloudflare]] — A different consensus architecture making different tradeoffs (leader-optional vs. adjudicator-mediated)
- [[DocumentDB]] — The same "Postgres-compatible but architecturally alien" strategy, reversed
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — The fallacies that make multi-region strong consistency hard
- [[SDPD — Systems Design Police Department]] — The failure modes ("Split Brain", "Conflicting Orders") DSQL must survive
- [[Queues Don't Fix Overload]] — Fred Hebert's thesis that treating symptoms ≠ fixing causes; DSQL's guardrails (3,000 rows, 10 MiB) are an attempt to fix the cause
- [[ARIES — Write-Ahead Logging Recovery]] — The recovery architecture DSQL's Journal reimagines for the cloud

---
*Sources: [[raw/aurora-dsql-murat-demirbas]], [[raw/aurora-dsql]]*
*Last updated: 2026-08-01*
