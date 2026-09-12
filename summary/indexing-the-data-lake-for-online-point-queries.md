---
url: https://engineering.atspotify.com/2026/7/indexing-the-data-lake-for-online-point-queries
title: Indexing the Data Lake for Online Point Queries
author: Spotify Engineering
date_fetched: 2026-08-21
date_published: 2026-07
topics:
  - databases-and-data
---

Spotify describes **Random Access Parquet (RAP)**, a technique for serving fast point queries (look up a key and its rows) directly from the Parquet files already sitting in the cloud data lake — no copy into a KV store like Bigtable or DynamoDB.

The motivating problem: online services and AI agents need per-user data at interactive latency, but the interesting datasets live in exabytes of lake storage while only petabytes fit economically in Bigtable. Distributed SQL engines (Trino, BigQuery) add seconds of scheduling and planning overhead even for a single-row lookup. RAP bridges the gap with an **external index** mapping each key to its exact file, rows, and pages, so a lookup becomes an O(1) index hit plus a few precise ranged reads issued in parallel — no dependent read chain.

The article's core is a catalog of write-time optimizations to Parquet files — sorting by key, co-grouping, one-page-per-key, ZSTD frame resets, blob/variant columns, interleaving, covering indexes with hoisted values — that shrink a point query to a single ranged read of a few kilobytes, or eliminate the storage read entirely. Every optimization has a tradeoff, and RAP shifts which tradeoffs matter: in-file discovery gets cheaper to skip, while minimizing the final read becomes paramount.

The payoff is economic: one dataset, two access patterns. The same files BigQuery scans for weekly reports are the files an AI agent reads for context retrieval — no copy, no ETL, no second storage bill.
