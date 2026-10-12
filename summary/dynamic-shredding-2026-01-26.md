---
url: https://pydantic.dev/articles/dynamic-shredding-2026-01-26
title: "Dynamic Shredding: Making Logfire Queries 1000x Faster"
author: Pydantic team
date_fetched: 2026-10-12
date_published: 2026-01-26
topics:
  - databases-and-data
---

Pydantic's Logfire observability product stores OpenTelemetry spans as Parquet on object storage. Span attributes are semi-structured key-value pairs, originally kept as a single JSON column parsed with Jiter. The post walks through why that layout was slow (extra IO dominating query time on object storage, poor compression, mandatory JSON parsing) and the fixes: first static shredding of a hardcoded set of common attributes into typed columns, now dynamic shredding.

Dynamic shredding automatically extracts the most frequently-seen or largest attribute keys into their own strongly typed columns at ingestion/compaction time, per Parquet file, capped at 128 columns (data shows 99% of files have fewer than 128 unique keys, median 32). Type widening follows Int→Float→JSON, Bool→JSON, String→JSON; a mix of types collapses the column to JSON (parseable but still columnar), while homogeneous strings stay as plain Utf8View with zero parsing. Queries on `attributes->'key'` get rewritten to the shredded column, so users keep a single logical JSON column while the physical layout is an internal detail (the `_lf_attributes_*` prefix now hidden from `select *`).

The engineering cost was substantial: six months of upstream DataFusion work on expression and predicate pushdown into the Parquet reader for JSON accessors, landing in DataFusion 52.1.0. Rollout is staged — rewrite the query path first while keeping static shredding in the write path, then flip the write path once confident. Future work: Parquet Variant for the remainder, dynamic indexing (inverted indexes, bloom filters), and usage-based shredding informed by query patterns.
