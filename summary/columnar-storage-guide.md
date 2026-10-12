---
url: https://motherduck.com/learn/columnar-storage-guide/
title: "What Is Columnar Storage? Column vs Row Databases Explained"
author: MotherDuck team
date_fetched: 2026-10-12
date_published: unknown
topics:
  - databases-and-data
---

MotherDuck's explainer on columnar versus row storage: what the layout is, why it wins on analytics, and when it loses. The core mechanism is column pruning — a query touching 3 of 50 columns reads roughly 6% of the bytes — stacked with data skipping (zone maps, Bloom filters), per-column compression (dictionary, RLE, delta, FOR, FSST), vectorized execution, and late materialisation. Each mechanism multiplies the others.

The guide is grounded in measured numbers rather than hand-waving: on ClickBench's 99.9-million-row benchmark with stock configs, the same data takes 106 GB in PostgreSQL (median query 4.3 minutes) versus 20.5 GB in DuckDB (0.35 seconds). On their own NYC taxi benchmark (11.2M rows, DuckDB v1.5.5), Parquet-Zstd beat CSV 22–60× on real queries, with `count(*)`'s ~160× called out honestly as a metadata trick rather than a headline.

It also carefully separates concepts that usually get muddled: columnar *storage* (the layout, e.g. Parquet) versus a columnar *database* (the engine around it, e.g. DuckDB); file formats (Parquet, ORC) versus in-memory exchange (Arrow) versus mutable internal storage (DuckDB's own format); and columnar OLAP engines versus wide-column stores like Cassandra, which are row-oriented under the hood despite the name. The anti-advice is as useful as the advice: skip columnar for OLTP, single-row writes, `SELECT *` lookups, tiny tables, and trickle ingest — keep the application database on Postgres and batch rows into Parquet for analytics.
