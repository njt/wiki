# We Replaced Redis with MySQL for Inventory Reservations

Shopify moved inventory reservations from Redis to MySQL and survived Black Friday 2025 at $5.1M/minute peak sales. The engineering story isn't "MySQL beat Redis" — it's a masterclass in diagnosing real bottlenecks, avoiding InnoDB footguns, and running a zero-downtime cutover on live commerce traffic.

---

## The Problem

Shopify's oversell protection reserves inventory during checkout (a short hold while payment processes) and claims it on payment success. Reservations lived in Redis (`DECR`/`INCR` on quantity keys); the inventory ledger lived in MySQL. The two couldn't be wrapped in a single atomic transaction. Depending on operation ordering, this caused either overselling (item sold but never deducted from ledger) or underselling (deducted from ledger but stuck as reserved).

> "Reservations and the inventory ledger were in two different systems. The claim step required updating MySQL and cleaning up Redis, and these couldn't be wrapped in a single atomic step."

The fix was deceptively simple: move reservations into the same MySQL database as the ledger so the entire reserve→claim flow becomes one ACID transaction.

---

## The Architecture: One Row Per Unit

Instead of a quantity column (`inventory_count = 10`), Shopify uses **one row per sellable unit**. An item with 10 units gets 10 rows. Reserving three means selecting and moving three rows in a single transaction. This lets MySQL's `SKIP LOCKED` handle concurrency: if another transaction has locked some rows, MySQL skips them and returns other available rows.

> "MySQL's `SKIP LOCKED` handles concurrency: if another transaction has locked some rows, MySQL skips them and returns other available rows, reducing contention."

```sql
BEGIN;

SELECT id
FROM reservation_units
WHERE ...
  AND status = 'available'
LIMIT 3
FOR UPDATE SKIP LOCKED;

UPDATE reservation_units
SET status = 'reserved'
WHERE id IN (...);

INSERT INTO reserved_quantities (...);

COMMIT;
```

### Bounded Pool Design

To prevent table bloat from items with huge inventory (e.g., 50,000 units across 10 locations), Shopify caps the available row pool at **1,000 per item/location**. This absorbs flash sale bursts while keeping the table compact and the `SKIP LOCKED` scan fast.

If the pool empties during an extreme flash sale, **inline replenishment** fires — one transaction acquires a lock and inserts more rows while others wait. The buyer never sees the item as unavailable unless it genuinely is.

---

## InnoDB Gotchas (and How They Fixed Them)

### Composite Primary Key

An early prototype used auto-increment ID as PK. `SHOW ENGINE INNODB STATUS` revealed **two row locks per reservation** — InnoDB was locking both the secondary index and the clustered index. Switching to a composite PK `(shop_id, inventory_item_id, inventory_group_id, id)` made the filter columns part of the PK, reducing to one lock per row.

### READ COMMITTED Isolation

`SELECT ... FOR UPDATE SKIP LOCKED` on an empty table produced **gap locks** including the "supremum" pseudo-record, blocking replenishment inserts and causing deadlocks. Switching to `READ COMMITTED` — their first use of a non-default isolation level in this codebase — eliminated the gap locks.

> "Running `SELECT ... FOR UPDATE SKIP LOCKED` on an empty table produced gap locks including on the 'supremum' pseudo-record, which blocked replenishment inserts and caused deadlocks."

### Consistent Lock Ordering

Reserve and claim touched two tables in different orders, creating deadlock cycles. The fix: reserve always `DELETE`s from `reservation_units` first, then `INSERT`s into `reserved_quantities`. Claim only touches `reserved_quantities`. Both paths now acquire locks in the same order — a classic deadlock prevention pattern.

### UNION ALL Batching

For carts with multiple line items, reservation queries are batched:

```sql
(SELECT id FROM reservation_units
  WHERE ... AND item_id = 1 LIMIT 2 FOR UPDATE SKIP LOCKED)
UNION ALL
(SELECT id FROM reservation_units
  WHERE ... AND item_id = 2 LIMIT 3 FOR UPDATE SKIP LOCKED)
```

One round trip instead of N.

---

## The Real Bottleneck: Connections, Not Queries

Throughput hit a ceiling below target despite acceptable P90 latency and non-maxed CPU. The team added **per-caller SQL tagging** — every SQL statement annotated with a comment tag identifying the business process:

```sql
/* conn_tag:checkout_completion */ SELECT ...
```

ProxySQL parsed the tag and measured how long each caller held a connection. This revealed that **other checkout processes** were holding connections longer than necessary. They hadn't been optimized because "they hadn't been the first to hit the limit."

> "Reservations were the straw that broke the camel's back — not slow themselves, but the pool was already near depletion."

**Results after cleanup:**
- 50% of reads removed from the primary database
- 33% of transactions removed from the primary database
- InnoDB thread concurrency increased (was set conservatively years earlier)
- Writer CPU under 50%, reader CPU under 16% during flash sales

---

## The Cutover: Dual-Write Shadow Mode

Every reservation was written to both Redis and MySQL, with Redis remaining the source of truth. This ran on production traffic until correctness and performance were validated. No in-flight reservations needed migration.

> "Redis reservations continued to be honored while MySQL built up its own state."

After validation, source of truth switched to MySQL. A kill switch allowed instant reversion since the dual-write path kept Redis complete. Rollout was gradual, pod by pod, starting with low-traffic pods.

---

## Critical Analysis

**This isn't really about Redis vs. MySQL.** The core insight is that splitting atomic operations across two data stores creates correctness problems that no amount of application-level coordination can fully fix. The move to MySQL was a correctness play first, a performance play second. The fact that it also scaled better is almost a side effect.

**The one-row-per-unit design is the clever part**, and it's not obvious. The instinct is to model inventory as a quantity — it's simpler, fewer rows, no "waste." But that design makes `SKIP LOCKED` impossible because you can't skip *part* of a quantity. The row-per-unit design trades row count for lock granularity, and that trade pays off massively under contention. This is the kind of insight that only comes from understanding how the database actually locks rows, not just what the SQL looks like.

**The bounded pool is pragmatic but fragile.** Capping at 1,000 rows means the system can handle up to 1,000 concurrent reservations per item/location before hitting the replenishment path. That's fine for most items, but the replenishment lock during a flash sale is a serialization point. Shopify clearly tested this — but the article doesn't say what happens when multiple items hit replenishment simultaneously across different locations. The serialization is per-item/location, which limits the blast radius.

**The connection pool diagnosis is the most transferable lesson.** Per-caller SQL tagging to measure connection hold time is something every team with a shared database should steal. The pattern — "our service isn't slow, but it's the one that exhausted a shared resource" — is universal. N+1 queries, long transactions, and connection leaks in *other* services are invisible until something tips over.

**The cutover pattern is textbook.** Dual-write with old system as source of truth, gradual rollout, kill switch. No in-flight state migration. This is how you do a database migration on live commerce traffic. The only thing missing is a discussion of how long they ran shadow mode and what discrepancies they found between Redis and MySQL states.

**The real headline is "be a safe neighbor."** Emilie frames the whole project around being a good database citizen — not maxing out your own throughput, but sustaining it without degrading everyone else. This is the database equivalent of tail latency awareness. Most teams optimize their own queries; Shopify optimized for the health of the shared database. That framing is rare and correct.

---

## Key Themes

#pattern — One-row-per-unit inventory with `SKIP LOCKED` for lock-level concurrency
#pattern — Bounded pool with inline replenishment as a safety valve
#pattern — Dual-write shadow mode for zero-downtime data store migration
#pattern — Per-caller SQL tagging for connection pool diagnostics
#tool — MySQL, ProxySQL, Redis
#concept — ACID transactions as correctness primitive for e-commerce
#concept — Connection pool pressure as the hidden bottleneck

---

## Related Pages

- [[Databases and Data]] — Synthesis page covering convergent databases, data quality, and the Redis→DB graduation pattern
- [[Materialized Views Are Obviously Useful]] — Same pattern: Redis cache as temporary fix, database as durable truth
- [[Dapper Performance Trap]] — How small database misconfigurations silently kill performance at scale
- [[DocDB — Stripe's Zero-Downtime Database]] — Another company operating a database at massive e-commerce throughput
- [[Idempotency Is Easy Until the Second Request Is Different]] — Redis vs. durable storage for correctness-critical operations
- [[PgDog]] — Connection pooling and proxying for database connection management
- [[Write Snapshot Isolation]] — Transaction isolation levels and correctness
- [[AliSQL]] — MySQL at massive scale (Alibaba), including OLAP and vector search extensions
- [[Software Engineering Craft]] — Synthesis page for fundamentals that don't change

---
*Sources: [[summary/scaling-inventory-reservations]]*
*Last updated: 2026-05-31*
