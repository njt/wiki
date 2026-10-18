---
url: https://clickhouse.com/docs/chdb/index
title: "chDB"
author: ClickHouse team
date_fetched: 2026-10-18
date_published: 2026-10-18
topics:
  - databases-and-data
  - developer-tools
---

chDB is ClickHouse's embedded, in-process OLAP engine: the full ClickHouse query engine as a library, no server to install, with language bindings for Python, Go, Rust, NodeJS, Bun, and C/C++. It reads and writes 70+ formats (Parquet, CSV, JSON, Arrow, ORC…) and claims "zero dependencies".

The headline addition is **DataStore**, a pandas-compatible API (630+ methods: 209 DataFrame methods plus accessors like `.str`, `.dt`, `.geo`) that runs operations lazily and pushes filters and aggregations down to ClickHouse — the pitch is "one-line migration" from pandas with up to 20× speedup on 10M-row benchmarks.

The rest of the page is navigation: getting-started guides per language, DataStore API reference, SQL API guides (querying S3, Parquet, Arrow, remote ClickHouse), and benchmark links (ClickBench of embedded engines, DataFrame benchmarks).
