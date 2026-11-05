---
url: https://www.opendata.dev/blog/introducing-vector
title: "Introducing Vector"
author: OpenData
date_fetched: 2026-11-06
date_published: 2026-11-06
topics:
  - databases-and-data
  - distributed-systems
---

OpenData announces *Vector*, an MIT-licensed stateless vector search engine built on SlateDB (an LSM store over object storage), pitched as filling the gap between running `pgvector` yourself and paying a vendor multiples of hardware cost to operate search for you. The headline claim: simple enough to run yourself, efficient enough to serve 100M vectors for roughly $350/month.

The post frames a three-generation evolution of online systems on object storage: tiered (cold data on S3), disaggregated (durable state on S3, stateful shard assignments on nodes — Chroma, Milvus), and stateless (all state, including metadata, in object storage, so any node can serve any data). Vector is the third generation; Turbopuffer is cited as the only proprietary system with a similar architecture, and Vector claims to be the only OSS one generally available.

Three architectural decisions make this work. First, IVF/SPANN-style inverted indexing (adapted from SPFresh) rather than graph indexes like HNSW: graph traversal requires sequential GETs to object storage with up-to-100ms first-byte latency, while IVF batch-loads posting lists per round trip and is cheaper to update incrementally. Second, LIRE compaction on an LSM tree: object storage can't do read-modify-write, so ingestion is append-only with reconciliation during compaction, using SlateDB merges for a lazy, low-write-amplification adaptation of LIRE. Third, share-everything state: nodes never talk to each other; readers work against SlateDB snapshots.

Tradeoffs are stated plainly: warm queries run ~10ms versus sub-millisecond for HNSW, write acknowledgement can take upward of a second (batched to amortize PUT costs), and cold queries are sub-second where a cold HNSW graph can take dozens of seconds to load. Benchmarks on a single `c6id.4xlarge` (~$346/mo) show warm latencies in the low single-digit to low-teens milliseconds at 90% recall, ~1K–12K vector writes/second, and S3 costs of tens of dollars for PUTs and $5–12/month for SIFT100M storage. Roadmap: smaller vector types, quantization, flattened centroid index, and full-text search.
