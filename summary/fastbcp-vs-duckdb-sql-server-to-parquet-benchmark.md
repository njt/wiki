---
url: https://blog.arpe.io/fastbcp-vs-duckdb-sql-server-to-parquet-benchmark
title: "FastBCP vs DuckDB: 150M SQL Server Rows to Parquet"
author: arpe.io
date_fetched: 2026-11-11
date_published: 2026-11
topics:
  - databases-and-data
  - claude-code
---

arpe.io benchmarked six ways of exporting 150 million TPC-H rows from SQL Server 2025 to zstd Parquet on one laptop: DuckDB 2.0 with its community `mssql` extension, three FastBCP live-database configurations via its own ADBC driver, and two FastBCP configurations that read the striped `.bak` backup files directly with no SQL Server involved.

FastBCP reading the backup with `Physical` parallelism wins outright at 10.1 s — 18× DuckDB's 181.5 s — with SQL Server at 0% for the whole run. On a live database, `RangeId` splitting on `o_orderkey` beats physical row-location splitting 2.3× at the same 16 threads, likely because columnstore segment elimination helps key-range predicates. DuckDB is not slow but starved: a single TDS read stream keeps it at ~0.6 core for three minutes.

Every run was measured twice with a 1.3-second sampler; the sampler's CPU-seconds matched FastBCP's self-reported CPU within 4%, which the author argues validates the CPU figures for DuckDB too. All six outputs were checksum-verified to hold exactly the same 150 million rows.

The benchmark was designed, run and written up by Claude Opus 5.5 in Claude Code using the arpe.io `fastbcp` skill, which encodes the tool's option-combination rules; the post is also a demonstration of that skill and the skills distribution model (Claude Code plugin marketplace, plain folder copy for other agents).
