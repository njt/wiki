---
url: https://ducklake.select/2025/05/27/ducklake-01/
title: "DuckLake: SQL as a Lakehouse Format"
author: DuckDB team (Mark Raasveldt et al.)
date_fetched: 2026-11-03
date_published: 2025-05-27
topics:
  - databases-and-data
  - developer-tools
---

DuckDB's announcement of DuckLake, an open "lakehouse" table format that replaces file-based metadata (Iceberg's JSON/Avro stacks, Delta's transaction logs) with an ordinary SQL database as the catalog and metadata store, while data stays in Parquet on blob storage.

The argument starts from an observation about Iceberg and Delta Lake: they were designed to avoid requiring a database, but the catalog layer they all need already reintroduced one for pointer-swap consistency. Once a database is in the stack anyway, encoding table state as "a maze of JSON and Avro files" is a solved problem being solved badly. DuckLake's format is instead a set of relational tables plus pure-SQL transactions: an insert is a single ACID transaction that records the new Parquet path, updates table/column/file statistics, writes a snapshot row, and logs the change.

The payoff claims are simplicity (no Avro/JSON files, no catalog API, "It's all just SQL"; runs locally with a DuckDB file as catalog, or on Postgres/SQLite/MySQL/MotherDuck/Spanner), scalability (three-way separation of storage, compute, and metadata; metadata transactions are orders of magnitude smaller than data changes), and speed (one query to plan a read instead of many sequential HTTP round trips; sub-millisecond small writes via optional inlining of changes into the metadata store; thousands of concurrent transactions and millions of snapshots).

Shipping alongside the format is the `ducklake` DuckDB extension (MIT, DuckDB Foundation), which turns DuckDB from a single-node engine into a client-server warehouse participant: any number of DuckDB instances attach to a central catalog and shared Parquet storage. The post includes a worked demo — ATTACH, insert, delete, time travel via `AT (VERSION => 2)`, incremental changes via `ducklake_table_changes()` — and notes data files remain Apache Iceberg-compatible for metadata-only migrations.
