---
url: https://www.databricks.com/blog/open-table-formats
title: "Open Table Formats"
author: Databricks
date_fetched: 2026-11-03
date_published: 2026 (undated)
topics:
  - databases-and-data
  - distributed-systems
---

Databricks' explainer on open table formats: metadata layers over Parquet/ORC in object storage that add ACID transactions, schema evolution, and time travel to data lakes. It profiles the big three — Apache Iceberg (Netflix-origin, catalog-coordinated, manifest-tree metadata), Delta Lake (Databricks-origin, transaction log, now adding catalog commits), and Apache Hudi (Uber-origin, record-level indexes for CDC-style upserts) — with a comparison table and per-format deep dives.

Key mechanics covered: the file-format vs table-format distinction (Parquet is a file format; the table format is the metadata layer above it), two-tier data skipping (file statistics cached in manifests/logs so engines skip whole files), partition evolution in Iceberg, deletion vectors and row lineage for cheap updates, and the emerging convergence — catalog commits aligning Delta with Iceberg's model, and a unified Delta/Iceberg metadata structure in preview.

The practical half is operations: manifest sprawl and small-file compaction, vacuum/snapshot-expansion retention windows, optimistic concurrency and shrinking writer overlap, format selection by engine stack rather than feature list, UniForm for cross-format reads, and a maintenance runbook (compaction thresholds, retention windows, monitoring file counts and commit failure rates). Unity Catalog governing 17+ exabytes is the closing proof point.
