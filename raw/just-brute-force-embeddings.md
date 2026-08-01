---
url: https://softwaredoug.com/blog/2026/07/29/just-brute-force-embeddings
title: Just Brute Force Your Embeddings
author: Doug Turnbull
date_fetched: 2026-08-01
date_published: 2026-07-29
---

# Just Brute Force Your Embeddings

By Doug Turnbull, July 29, 2026

Many teams with ~1M documents, low query traffic, and up-front embedding writes don't need a vector database. Turnbull recommends brute-forcing embeddings with NumPy until the scale no longer permits it.

## Benchmarks

384-dim embeddings, M4 MBP, single dot-product line:

- 1,000,000 docs: 79.7 QPS, 0.012s avg latency (1 thread); 170.5 QPS, 0.058s (10 threads)
- 8,841,823 docs: 9.34 QPS, 0.106s (1 thread); 18.34 QPS, 0.106s (10 threads)

The code: `scores = self.doc_vectors @ query_vector.astype(np.float32, copy=False)`

## Key Quotes

- Raymond Chen: "My O(n) algorithm can run circles around your O(log n) algorithm"
- Jo Kristian Bergum: "an exhaustive search may be all you need"

## Footnotes / Future Directions

Turnbull notes this is naive NumPy and could be faster:
- Andreas Erickson on improving throughput by giving threads more than one query per scan
- A top-n heap collection would beat NumPy's approach
- Past this scale, consider a database or just loading vectors in memory with FAISS

## Promotion

The post advertises "Vectors Week," a series of events on vector retrieval, hybrid search, and building your own vector database.
