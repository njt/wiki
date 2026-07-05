# zvec

Alibaba's lightweight, in-process vector database. No server, no configuration -- just `pip install zvec` and start searching. Handles billions of vectors in milliseconds, supports both dense and sparse embeddings, and runs anywhere from Jupyter notebooks to edge devices.

---

## Key Themes

#vector-db #database

zvec is the "SQLite of vector databases" -- an embeddable library, not a service. This matters because most vector search use cases don't need a distributed system. If your vectors fit on one machine (and with modern hardware, that's billions of vectors), an in-process library eliminates network overhead, deployment complexity, and an entire class of operational concerns.

Key engineering decisions: WAL for durability (data survives process crashes), multi-process read concurrency with single-writer exclusivity (simple and correct), and hybrid search combining semantic similarity with structured filtering. The multi-vector query support is useful for systems that combine different embedding models or representations.

Part of Alibaba's broader data stack alongside [[AliSQL]] (MySQL + DuckDB OLAP + vector search). The two tools target different niches: zvec for embedded use cases, AliSQL for when you need vector search integrated into a full relational database.

Also contrast with [[NornicDB]] (graph + vector + temporal -- much heavier) and [[GraphRAG]] (which needs a vector store as part of its pipeline and could use zvec as a lightweight option).

## Critical Analysis

The "billions of vectors in milliseconds" claim is plausible for an in-process C++ engine with HNSW indexing, but the README doesn't provide detailed benchmarks. The Python + Node.js bindings cover the most common use cases. Battle-tested at Alibaba is a meaningful credential -- production scale tends to surface bugs that benchmarks miss. The main limitation is single-writer exclusivity, which means zvec isn't suitable for high-write-throughput applications. For read-heavy search workloads (the common case), this is fine.

---
*Sources: [[summary/zvec]]*
*Last updated: 2026-05-14*
