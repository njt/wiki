---
url: https://www.databricks.com/blog/lakebase-ltap-rethinking-database-storage
title: "From monolith to Lakebase to LTAP: rethinking the database from storage up"
author: Reynold Xin
date_fetched: 2026-07-05
date_published: 2026-06-30
site: Databricks Blog
category: Engineering
topics:
  - databases-and-data
---

Reynold Xin recounts his journey from PhD skepticism about OLTP databases being "solved" through to Databricks' Lakebase architecture and the LTAP (Lake Transactional/Analytical Processing) paradigm. The core argument: traditional databases are fragile monoliths because the WAL and data files live on one machine. Lakebase fixes this by externalizing both into distributed cloud services (SafeKeeper for WAL, PageServer for data files), making Postgres compute stateless. LTAP goes further: it stores operational data once in open columnar formats (Delta/Iceberg as Parquet) so both Postgres and Lakehouse engines read the same fresh data — no CDC pipeline, no second copy, no performance penalty to transactions.

The article opens with a four-point summary:

1. Traditional databases keep WAL and data files on one machine's disk, causing data loss risk, expensive read replicas, and analytics queries harming transactional performance.
2. Lakebase makes Postgres compute stateless by moving the log and data files into external cloud services (SafeKeeper and PageServer), enabling unlimited storage, elastic compute, durable writes, simpler HA, and instant branching.
3. LTAP stores operational data once in open columnar formats read by both Postgres and Lakehouse engines, so analytics runs on the same fresh data transactions just wrote — "with no CDC pipeline, no second copy, and no slowdown to the transactional workload."
4. Unlike HTAP (unifying workloads in one engine), LTAP unifies at the storage layer and keeps the best engine for each job.

## Section 1: The Database as a Monolith

Xin recounts his PhD at UC Berkeley 16 years ago, where his advisor told him OLTP databases were a solved problem. He says they later realized "OLTP databases were far from a solved problem: they were clunky, difficult to scale, and incredibly fragile."

**Monolith architecture explained:** Most databases (MySQL, Postgres, Oracle) run the engine and storage on one machine, with two critical on-disk components:
- **Write-Ahead Log (WAL)** — sequential log; a transaction commits when the log entry is durably written
- **Data files** — updated asynchronously afterward

The core insight: "the WAL exists to make *writes* fast (and safe), and the data files exist to make *reads* fast."

**Challenges of monolithic design:**

1. **Data loss from misconfiguration** — If the WAL write is acknowledged before actual durable flush, commits can vanish. "The operating system might even decide to lie to you about flushing."
2. **Data loss from node loss** — WAL and data files live on one machine; if that disk dies, data dies.
3. **Scaling reads requires a physical clone** — Read replicas are full copies streaming and replaying the WAL. "For a large database, that is not a quick operation and might even bring down the database."
4. **HA requires a physical clone** — At minimum twice the infrastructure; synchronous replication needed to avoid data loss.
5. **Analytics contend with transactional traffic** — Heavy analytical queries share the same hardware as latency-sensitive OLTP.

Root cause: "the WAL and the data files are stored inside a single machine."

## Section 2: Lakebase Architecture

The design approach uses cheap, durable cloud object storage paired with elastic compute, building on the Neon team's work. The core principle: **make Postgres compute instances stateless** by externalizing WAL and data files into purpose-built, independently scalable services.

## Section 3: Scaling Writes — WAL Becomes SafeKeeper

In Lakebase, the WAL is externalized to **SafeKeeper**, a distributed storage service. Durability comes from "replicating the log record across a quorum of SafeKeeper nodes using Paxos-based network replication" rather than disk flush. No disk failure can lose data; no misconfigured flush undermines durability.

**Latency question:** Moving commits from local disk to SafeKeeper does not increase write latency because serious Postgres deployments already require synchronous replication with an extra network hop. The combination of SafeKeeper and PageServer can yield "5X higher write throughput and 2X lower read latency."

## Section 4: Scaling Reads — Data Files Become PageServer

Data files move to **PageServer**, another distributed storage service. The WAL streams from SafeKeeper into PageServer, which asynchronously applies changes and materializes pages into low-cost cloud object storage. PageServer acts as a write-through cache for underlying object storage.

When a page is requested and PageServer doesn't have the latest version, it applies logs from SafeKeeper to reconstruct the latest state.

**Latency:** Read latency isn't meaningfully increased due to multi-layered caching. Postgres first checks its buffer pool (local memory), then a local disk cache, and only goes to PageServer on a cache miss. "For the vast majority of operations, read latency is indistinguishable from a monolith."

## Section 5: What This Unlocks

Capabilities that become natural consequences of the architecture:

- **Still Postgres** — Wire protocol, SQL, drivers, and extensions work as-is.
- **Unlimited storage** — Data lives in cloud object storage, not provisioned local disk.
- **Serverless, elastic compute** — Stateless compute scales up under load and down to zero when idle.
- **Durable writes and zero data loss** — "A commit is durable once it is replicated across SafeKeeper nodes via Paxos, not when a single local disk claims to have flushed it."
- **Simpler high availability** — Durable state lives in a replicated storage layer independent of any single compute instance.
- **Instant branching, cloning, and recovery** — Xin calls this his favorite: "a branch or a clone is a metadata operation rather than a physical copy." A large production database can be branched in seconds.

Xin notes that separating compute from storage is not new, but the key with Lakebase is storing operational data on commodity object storage in an open format, enabling other engines to read it directly — leading to LTAP.

## Section 6: LTAP — One Copy for Transactions and Analytics

Once data lives in externalized storage, the transactional database and analytical system no longer need to be separate worlds.

**The old problem:** Even with Lakebase, data in object storage was in Postgres's native row-by-row page format — "great for transactions and poor for analytics." Analytical engines needed either conversion cost on every read or a separate copy kept in sync by a pipeline (brittle, governance nightmare).

**LTAP (Lake Transactional/Analytical Processing):** Removes the two-copies-of-data problem by unifying at the **storage** layer rather than the **engine** layer. Postgres handles transactions with full ACID semantics; Lakehouse engines handle analytics. "What changes is the data underneath them" — one durable copy in open columnar formats (Delta, Iceberg, stored as Parquet) that both sides read.

## Section 7: Materializing in Columnar Form

As PageServer materializes pages into object storage, it transcodes Postgres data from row format into Parquet's columnar layout. This preserves exact Postgres representation of every value, "down to the bits." Unlike CDC, which ships logical change events into a foreign schema and leaves Postgres semantics behind, LTAP keeps them.

**Two key preservation mechanisms:**

1. **Type system** — Most Postgres types map to native Parquet types. Exotic values (NaN, ±Infinity, extended-precision NUMERICs, extension types) are carried in a structured overflow field within the same table, holding canonical Postgres text. This field is "both directly queryable by any engine and sufficient to reconstruct the original Postgres bytes exactly on the way back."

2. **Multi-versioning** — Postgres retains every row version any transaction could observe (snapshot isolation, PITR). Open table formats expose table-wide consistent snapshots without intermediate row versions. LTAP separates durability from visibility: every row carries its physical heap address; the classic Postgres heap page becomes a cache, while the durable source of truth lives in columnar files. Intermediate row versions preserve MVCC and PITR but are invisible to Iceberg/Delta readers and eventually garbage-collected.

**Side effect:** Columnar data compresses "often by more than ten times" better than row data, cutting network volume between caching and object store. The format that makes analytics fast also makes storage cheaper. During LTAP's transitional rollout, both row and columnar formats are dual-written for data verification.

## Section 8: Reading the Latest Data Without Affecting Postgres

**Freshness challenge:** If analytics reads from a copy in the lake, how does it see data committed moments ago and not yet materialized?

**LTAP's approach:**
1. When an analytical query starts (e.g., from Lakehouse//RT), it asks Postgres for the current **LSN** (log sequence number) — a cheap metadata lookup.
2. The analytical engine reads the overwhelming majority of data — everything materialized up to that point — directly from object storage.
3. The small set of very recent changes not yet materialized to the lake is fetched from PageServer and merged on top.

The result is a "consistent, fully up-to-date read of your data as of that LSN." Almost all work lands on cheap, scalable object storage. "Postgres itself serves none of the analytical read traffic other than returning a single number (LSN)."

**Optimization:** Very small tables (handful of rows) are not converted to columnar form — "the bookkeeping would cost more than it saves."

## Section 9: Every Table, Automatically

Xin contrasts LTAP with CDC approaches (also called "mirroring," "zero CDC," or "zero ETL"): "because the data replication pipeline costs something, it cannot be applied to all the tables." Users must explicitly select tables, and replication comes with delay.

"In LTAP, there is nothing to opt into." Every table is already in the lake and already queryable. "There is no list of replicated or mirrored tables, because there is no replication." A single governed copy in open formats with no ETL pipeline — "analytics is always reading the same data the application just wrote."

## Section 10: What About HTAP?

LTAP is a deliberate play on HTAP (Hybrid Transactional/Analytical Processing). Xin argues no widely adopted HTAP system exists, citing three problems:

1. **Incomplete feature set** — Building a new proprietary engine for one job is multi-year investment; building one for multiple jobs compounds the challenge. Systems "often lag on things people assume are always there, from the breadth of SQL support (e.g. foreign key support) to the maturity of the query optimizer."

2. **No ecosystem** — "Postgres and Spark each sit at the center of a vast ecosystem." A new engine starts outside all of it.

3. **No performance isolation** — Many HTAP systems run both workloads on the same hardware, creating the same contention problem as the monolith.

All three trace back to "unifying the two workloads into one engine." Lakebase/LTAP avoids these by unifying at the storage layer while using different compute engines for different workloads.

## Section 11: Closing Thought

Xin says the Lakebase architecture's benefits (unlimited storage, elastic compute, durable writes, simpler HA, instant branching) followed "almost mechanically once the WAL lived in the SafeKeeper and the data files lived in the PageServer." The LTAP idea came later, after the Neon and Databricks teams collaborated. As LTAP rolls out, "all of your Lakebase tables will just be available for analytics as high performance as the Lakehouse data."

He notes the same design opens optimization opportunities for separating other heavyweight maintenance operations from core transactional workloads: "We are just beginning to explore what this architecture makes possible."

## Key Technical Concepts Summary

| Concept | Description |
|---|---|
| **WAL** | Write-Ahead Log — sequential log used for fast/durable writes in databases |
| **SafeKeeper** | Distributed service externalizing the WAL using Paxos-based quorum replication |
| **PageServer** | Distributed service externalizing data files; materializes pages into cloud object storage |
| **Lakebase** | Databricks' serverless Postgres with stateless compute and externalized storage |
| **LTAP** | Lake Transactional/Analytical Processing — single storage copy in open columnar formats for both transactional (Postgres) and analytical (Lakehouse) engines |
| **HTAP** | Hybrid Transactional/Analytical Processing — single engine approach that LTAP deliberately contrasts with |
| **LSN** | Log Sequence Number — cheap metadata lookup that enables fresh analytical reads without burdening Postgres |

**Acknowledgment:** Xin thanks the Lakebase team for making the technology real and reviewing the blog.
