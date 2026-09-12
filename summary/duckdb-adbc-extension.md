---
url: https://columnar.tech/blog/announcing-duckdb-adbc-extension/
title: "Announcing the DuckDB ADBC Extension"
author: Sam Arch
date_fetched: 2026-07-11
date_published: 2026-07-08
topics:
  - databases-and-data
---

Sam Arch announces the DuckDB ADBC extension, a community extension that lets DuckDB connect outward to any database with an ADBC (Arrow Database Connectivity) driver — including Snowflake, Databricks, BigQuery, PostgreSQL, MySQL, and 25+ others. Built on Apache Arrow, it enables fast columnar data transfer and avoids the row/column conversion overhead of ODBC and JDBC.

The extension offers two interfaces: `read_adbc` for one-shot remote queries returning DuckDB tables, and `ATTACH` for persistent connections that support SELECT, INSERT, COPY, and CTAS statements as if the remote database were local. Connection profiles are stored as TOML files, and drivers are installed via the `dbc` CLI tool.

Key features include streaming bulk ingest (batched inserts keeping memory low), automatic connection pooling, and metadata caching with a manual `adbc_clear_cache()` refresh. Current limitations: autocommit-only mode (no multi-statement transactions) and no predicate or projection pushdown on attached tables (workaround via `read_adbc` with a macro to push filters to the remote side).

The extension is open source and the team plans to contribute it to the Apache Arrow project as an official ADBC library.
