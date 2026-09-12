---
url: https://github.com/Ladybug-Memory/icebug-format
title: "icebug-format"
author: Ladybug Memory
date_fetched: 2026-08-01
date_published: 2025
topics:
  - databases-and-data
---

A Python CLI and library that converts property graphs stored as relational tables
into CSR (Compressed Sparse Row) adjacency format, backed by Parquet files on disk
or Apache Arrow tables in memory.

The disk format (`icebug-disk`) uses DuckDB to read `nodes_*` / `edges_*` tables and
produces Parquet files for node data, CSR indices/index pointers, and a Cypher
schema file. The memory format (`icebug-memory`) converts Arrow tables directly
into an in-memory CSR graph via a Python API.

The design prioritizes scan efficiency from object storage: a graph engine can
mount the Parquet files directly without ingesting into a proprietary store. CSR
encoding gives O(1) neighbor lookup. The trade-off is write-time convenience — it
is a conversion target, not something authored by hand, and updates are full
replacement only (no incremental support).

Notable technical details: per-type dense integer node mapping (not globally
dense), DuckDB for all heavy SQL lifting, self-loop deduplication in reverse-edge
mode, and flexible source/target column name resolution across six aliasing
conventions. Also reads GraphAr format as input.
