---
url: https://infino.ai/blog/search-index-on-s3/
title: "What changes when your search index lives on S3?"
author: Infino team
date_fetched: 2026-10-24
date_published: 2026-10
topics:
  - databases-and-data
  - agent-memory-and-context
---

Infino, a retrieval engine for agent questions, describes how its full-text search index works when it lives on object storage (S3, GCS, Azure Blob) rather than local disk. The core argument: on local SSD a random read is nearly free, but on object storage every random read is a billed HTTP GET taking tens of milliseconds — so round trips, not CPU or bytes, are the scarcest resource, and nearly every design decision follows from minimising them.

Key design choices: data is organised into *superfiles* — single Parquet objects with full-text and vector indexes spliced in as byte regions before the footer, readable by any standard Parquet tool. A per-supertable *manifest* stores every superfile's Parquet footer and index headers inline, so opening a superfile costs zero GETs. A *term index* maps each term to the superfiles containing it, with document counts, BM25 score bounds, and postings byte offsets — letting a query skip thousands of superfiles without opening them, and in one test answering a single-term count across 141 superfiles without opening any. Beyond that: ranged GETs fetch only the needed byte regions, nearby postings are coalesced into one request (gaps up to 64 KB), and superfiles are downloaded to a local disk cache in the background, after which queries hit memory-mapped pages with no GETs at all.

The author's closing point is notable: the search engine itself did not change — postings, BM25, block-max pruning are Lucene ideas. Only the storage layer changed, because that is where the cost model changed. This is part 1 of a series; later posts cover the index data structures and parallel query execution.
