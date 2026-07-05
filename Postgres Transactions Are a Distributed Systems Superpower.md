# Postgres Transactions Are a Distributed Systems Superpower

Peter Kraft and Qian Li argue that co-locating workflow engine state in the same Postgres database as application data eliminates entire categories of distributed systems problems — idempotency bugs, outbox infrastructure, and reconciliation jobs — by making workflow checkpoints and business updates share a single transaction boundary.

---

> "In distributed systems, co-location is a superpower."

The central insight is dead simple and underappreciated: when your workflow engine is a separate service, every checkpoint involves a network round-trip. The workflow can crash between completing a step and recording that fact. When it restarts, it re-executes — and now you need application-level idempotency logic or bookkeeping tables to avoid double-charging customers. But if the workflow engine lives *inside* your Postgres database, the checkpoint and the business update happen in the same transaction. Either both commit or neither do. Exactly-once semantics fall out of the transaction model you already have.

## The Idempotency Argument

> "Durable workflows alone do not solve the idempotency problem."

This is the article's sharpest technical claim. Durability means workflow state survives restarts — but it doesn't prevent double-execution of the last step before a crash. The authors walk through a concrete example: crediting $100 to a bank account, crashing, then re-executing. Without co-location, you need an `applied_payments` table and a check-before-credit pattern. With co-location, the checkpoint write and the credit happen atomically. The commit IS the checkpoint.

> "Transactional steps no longer need application-level idempotency logic or bookkeeping tables."

Code becomes simpler, correctness becomes structural rather than procedural. This is the kind of argument that makes experienced engineers lean in — it's not about performance or convenience, it's about eliminating a class of bugs at the architecture level.

## The Outbox Simplification

> The transactional outbox pattern "introduces additional operational complexity" — polling infrastructure, retries, monitoring, and reconciliation jobs to fix drift.

The second argument applies co-location to the transactional outbox pattern. Traditionally, if an order placement needs to atomically update the database AND trigger a fulfillment workflow, you write to an `outbox` table in the same transaction, then run a background poller to deliver messages. This works — but now you own polling infrastructure, retry logic, monitoring dashboards, and reconciliation jobs for when the poller misses a row.

With co-location, a Postgres UDF called in the same transaction creates a workflow row (name, queue, input). Workers dequeue and execute asynchronously. The pattern is identical in principle — single transaction ensures atomicity — but the infrastructure collapses to a database function and a worker pool. No separate outbox table to manage, no poller to monitor.

## Critical Analysis

**The argument is real, and it generalizes.** Co-location isn't just about DBOS or even Postgres — it's a distributed systems principle. When two pieces of state must be updated atomically, keeping them in the same transactional boundary eliminates the need for compensating transactions, saga patterns, and reconciliation infrastructure. The article picks the right two examples (idempotency and outbox) because they're the ones every working engineer has wrestled with.

**But Postgres isn't the whole answer.** The article implicitly assumes Postgres is your primary database, that your write volume fits in a single Postgres instance, and that your workflow engine can absorb being database-coupled. If you're on DynamoDB, or you need cross-database workflows, or your workflow engine needs to survive database failover independently — the pattern breaks. "Co-locate with your data" is the durable insight; "co-locate inside Postgres" is the implementation detail.

**The vendor bias is present but doesn't invalidate the argument.** This is a DBOS blog post, and DBOS is a product that implements this pattern. The call-to-action at the end is promotional. But the technical reasoning stands on its own — you could implement this pattern with a hand-rolled Postgres function and a worker pool, no DBOS required.

**The "Just Use Postgres" convergence is worth watching.** This article sits alongside the same authors' "Just Use Postgres for Task Queues" and Microsoft's [[DocumentDB]] (MongoDB-on-Postgres). The trend is toward treating Postgres as the universal backend substrate — not just a relational database, but a queue, a document store, a workflow engine, and a search index. There are real scaling limits here (single-writer bottleneck, connection pooling at scale, operational complexity of a database that does everything), but for the 95% of applications that will never outgrow a single Postgres instance, the simplicity argument is compelling.

**What's missing:** No discussion of what happens when the workflow worker pool backs up, how to handle poison-pill workflows that crash every worker that dequeues them, or how to observe and debug workflows when they're database rows rather than service calls. The operational happy path is simple; the unhappy paths are where you discover whether co-location was a good idea.

---
*Sources: [[raw/co-locating-workflow-state-with-your-data]]*
*Last updated: 2026-07-05*

## Related

- [[Your Backend Is Full of Hidden Workflows]] — the coordination logic that accretes invisibly across services; making workflows explicit
- [[Apache Burr]] — explicit state machines as the alternative to implicit workflow state
- [[Process Flow]] — choreography without a central workflow DAG; each stage designates its successor
- [[Swamp Club]] — typed, versioned agent workflows with immutable data on every run
- [[State System]] — organizational state layer with append-only journals and evidence-first commits
- [[Databases and Data]] — storage as a design problem; the convergence of database categories
- [[Distributed Systems]] — agent orchestration IS distributed systems; the primitives keep getting reinvented
- [[Streambed]] — Postgres-to-Iceberg CDC in a single Go binary; the Postgres-as-platform thesis
- [[DocumentDB]] — MongoDB wire protocol on Postgres; treating Postgres as universal backend
- [[Queues Don't Fix Overload]] — why queues treat symptoms not causes; the co-location argument's unstated assumption that your database can handle the load
- [[Event-Driven vs Polling Architectures]] — the polling problem the transactional outbox was designed to solve
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — the network fallacies that co-location sidesteps
