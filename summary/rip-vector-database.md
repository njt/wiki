---
url: https://turbopuffer.com/blog/rip-vector-database
title: "RIP Vector Database"
author: turbopuffer
date_fetched: 2026-10-02
date_published: unknown (recent, pre-2026-10)
topics:
  - databases-and-data
  - ai-infrastructure-and-hardware
---

turbopuffer announces v3, an architectural rework that demotes the ANN vector index from primary index to "just another" secondary index. The post is a day-zero engineering narrative: CI fully green on v3, performance parity still ahead, benchmarks promised in public.

The story runs v1 → v2 → v3. V1 was a serverless vector store built on object storage with SPANN/SPFresh hierarchical clustering, where every document is keyed by its **ANN address** (`ClusterId` + `LocalId`). Attribute filtering and BM25 full-text search (v2) were bolted on as inverted indexes that point *into* ANN addresses, plus other plans (aggregations, regex, sparse vectors) all built around the same layout. This served them well — 100B+ vectors, 200 ms p99 at 1k+ QPS, customers like Cursor and Notion — but three structural costs come with keying everything on the ANN index:

- **Storage amplification**: multi-vector documents (nesting, late interaction) duplicate non-vector content per vector.
- **Write amplification**: SPFresh rebalancing moves vectors between clusters, dragging the full document and all its inverted indexes along — updating one vector can move hundreds of attributes.
- **Limited vectorization**: every query plan inherits the ANN cluster size (~100–200 docs) as its block size, far below the 2k–65k blocks vectorized engines like DuckDB and ClickHouse prefer. Their FTS v2 rewrite — posting blocks of ~256 instead of ~1.5 — got a 10x smaller index and up to 20x faster queries, proving the point.

The fix is to stop keying on the ANN address, making vector search one secondary index among many and opening the door to fast SQL-style queries (GROUP BY, aggregations). Scale context: 1T+ documents, 10M+ writes/s, 25k+ queries/s.
