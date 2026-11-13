---
url: https://www.databricks.com/blog/dispelling-ltap-myths
title: "Dispelling LTAP Myths"
author: Databricks
date_fetched: 2026-11-13
date_published: 2026-11-13
topics:
  - databases-and-data
---

Databricks answers four common misreadings of LTAP (Lakehouse Transactional Access Protocol) and the Lakebase architecture — the claim that one governed copy of operational data can serve both Postgres and analytics with no CDC pipeline.

**Myth 1 — "It's just CDC/zero-ETL."** LTAP is a tiered storage hierarchy, not a replica pipeline. The test: a layer that cannot be queried or failed over to on its own is a tier, not a replica — like a Postgres index, which nobody calls a replica of the heap. CDC creates a genuinely second database: an independently queryable upserted table that can drift on types, DDL, and replication-slot loss. LTAP has no second catalog and no opt-in table list.

**Myth 2 — "Two formats means two copies."** Heap pages and Parquet are two lossless encodings of the same rows serving different access paths — point lookups vs scans — the way a Postgres heap and its covering index already coexist. The columnar files are heap-ordered, so warehouse-style clustering can still be derived later; only the CDC pipeline that *created* the lake copy is eliminated.

**Myth 3 — "Analytics is always stale."** Analytical queries read a consistent snapshot at a single LSN, merging transactionally consistent changes from pageservers for commits not yet materialized to object storage. This also fixes CDC's cross-table join inconsistency (each table at a different slot position). Small tables stay in the page cache and skip Iceberg commits.

**Myth 4 — "Just decode the WAL in parallel."** Logical replication replays every change and carries heavy state (slots, DDL gaps, REPLICA IDENTITY FULL); it's the wrong tool for building lake tables. Parallel log decoding can't be made correct because visibility lives outside the tuple. LTAP parallelizes *pages*, not log records — WAL summaries or pageserver LSMs identify dirty pages, each independent, transcoded into one Iceberg snapshot.
