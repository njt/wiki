---
url: https://softwaredoug.com/blog/2026/07/29/just-brute-force-embeddings
title: "Just Brute Force Your Embeddings"
author: Doug Turnbull
date_fetched: 2026-08-01
date_published: 2026-07-29
topics:
  - databases-and-data
---

Doug Turnbull argues that teams with moderate embedding workloads — roughly a million documents, low query traffic, and up-front embedding writes — should skip vector databases and use brute-force NumPy dot products instead.

He benchmarks 384-dimension embeddings on an M4 MacBook Pro. At one million documents, a single thread achieves ~80 queries per second with 0.012s latency; ten threads push that to ~170 QPS at 0.058s. Even at nearly nine million documents, single-thread performance remains around 9 QPS at 0.106s — adequate for many use cases.

Turnbull notes this is naive NumPy and could be faster: per-thread batching, a top-n heap for collection, or loading vectors into memory with FAISS for larger scales. His core point echoes Raymond Chen — a simple O(n) scan can beat a complex O(log n) index at practical sizes.
