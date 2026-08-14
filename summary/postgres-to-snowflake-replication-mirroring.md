---
url: https://www.snowflake.com/en/blog/engineering/postgres-to-snowflake-replication-mirroring/
title: "Postgres to Snowflake Replication & Mirroring"
author: Snowflake Engineering
date_fetched: 2026-08-14
date_published: unknown
tags: [cdc, postgres, snowflake, iceberg, replication, data-mirroring]
---

## Source Content

Snowflake's engineering deep-dive into **data mirroring**, a public-preview feature of Snowflake Postgres that replicates Postgres tables into Snowflake with low cost, low lag, and transactional consistency — with "no extra infrastructure."

### Core Thesis

Traditional Postgres change data capture (CDC) is fragile because external consumers (pull-based logical decoding) know nothing about Postgres's internal state — schema changes, snapshot alignment, or whether the database is even alive. Snowflake's answer is to **push** changes out of Postgres instead: a new extension (`snowflake_cdc`) runs inside Postgres and pushes transactional batches of changes directly into Apache Iceberg tables (compressed Parquet) in object storage. Snowflake then applies those batches transactionally and serverlessly.

The punchline: *"transactional push into the data lake, transactional apply in Snowflake, no extra infrastructure"* turns replication "from a chaotic process … to a simple clockwork that will run forever."

### Key Mechanics

- **Four-stage timeline** — every write passes through Write → Decode → Capture → Apply, each a continuous process on the same timeline at a different point in time.
- **Historic snapshots** — the decoder reads Postgres's catalog *as it was at write time*, so WAL records can be decoded even after a table was altered or dropped.
- **Meta log + change logs** — changes land in Iceberg as per-table change logs plus a "meta log" of instructions; the Snowflake-side apply process is a finite state machine executing the meta log.
- **Transactional boundaries on both sides** — batches are pushed in one Postgres transaction and applied in one Snowflake transaction, moving every table forward exactly to a Postgres transaction boundary, preserving foreign keys and join correctness.
- **Built on pg_lake** — the generally-available "Postgres for your data lake" (open-source `pg_lake`) that lets Postgres run transactions across Postgres tables *and* Iceberg tables.
- **Append, never upsert** — because replication is transactionally controlled from Postgres, changes are "perfect deletions and insertions applied exactly once." Inserts are appended, making insert-heavy workloads fast and cheap, and avoiding the inconsistent intermediate states and expensive columnar-match of the upsert approach.
- **Live views** — combine unapplied change-log rows with target-table data, pushing filters/projections down into Parquet and base-table scans. Applying changes *infrequently* still keeps lag well under a minute.

### Product Framing

Snowflake Postgres offers two ways to unify operational and analytical workloads: **data mirroring** (always-on, automatic, set-once replication) and **Postgres for your data lake** (developer-controlled SQL-based movement into Iceberg).
