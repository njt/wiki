# RIP Vector Database

turbopuffer's announcement that v3 demotes the ANN index from primary key to secondary index — a storage-engine retrospective (SPANN/SPFresh, object storage, inverted indexes) and a diagnosis of the three amplifications that vector-primary layouts impose on everything else.

---

## Summary

turbopuffer launched as a "serverless vector database": object storage as source of truth for cheapness, NVMe/memory caches for speed, with a SPANN-style hierarchical clustering index as the primary key. Every document lived under its ANN address (`ClusterId` + `LocalId`), and every later feature — attribute filters, BM25, regex, aggregations, sparse vectors — was built as an index pointing into those addresses. That design delivered 100B+ vectors at 200 ms p99 for customers like Cursor and Notion, but it now caps the non-vector query shapes the company wants to serve. V3 flips the primary index: ANN becomes one secondary index among many. The post is deliberately day-zero — CI green, perf grinding starting, benchmarks promised in public.

## Key Quotes

> "We've pushed the vector-primary architecture as far as we can, and it's time to move on. We're in the process of moving to a new primary index, and making ANN 'just another' secondary index."

The whole thesis in two sentences. "Vector database" as a category is partly an artifact of one storage-layout decision; turbopuffer is outgrowing its own founding label.

> "Updating just one vector can move hundreds of attributes and their indexes."

The write-amplification story in one line. When the ANN index is the key, a purely statistical rebalancing operation (SPFresh re-clustering to protect recall) becomes a data-moving event for everything else. Physical layout leaking into logical cost is a classic storage-engine disease.

> "Every query plan has an optimal block size, but today they are all constrained by the ANN primary index. A plan that wants blocks of thousands of documents to keep the CPU saturated is still stuck at 100–200."

The vectorization argument, and the most technically interesting one. Their FTS v2 rewrite is the proof: decoupling posting blocks from ANN clusters made the index 10x smaller and queries up to 20x faster. Everything still keyed to clusters can't make the same jump.

> "Watching benchmark numbers go down is great fun, so we wanted to get you in at day zero of perf grinding."

A refreshing inversion of launch-culture: publishing *before* the performance numbers exist, with 100% CI passing as the milestone. Correctness first, speed second — stated as sequencing, not slogan.

## Key Themes

- #concept **Primary index choice shapes everything** — what you key your storage on determines block sizes, write fan-out, and which query plans can ever be fast. The ANN address was a great v1 decision and a v3 liability; that's not a mistake, it's how architectures age.
- #pattern **Physical layout is destiny** — the FTS v2 result (1.5 → 256 posting blocks, 10–20x gains) shows how much performance lives in layout rather than algorithms.
- #tool Vector databases as a *phase*, not a endpoint — the ANN index becoming "just another secondary index" is the general-purpose database reabsorbing a specialized one.

## Critical Analysis

This is the good kind of architecture post: it names its tradeoffs, admits the old design "works really, really well," and quantifies the pain (block sizes, amplification cascades) instead of hand-waving. The honesty about risk is real — they concede any change here "risks introducing regressions in ANN performance," and they haven't shown a single benchmark yet. So treat v3 as a promise with a correct blueprint, not a result.

Two things deserve scrutiny. First, the unstated tension: the ANN-primary layout is *why* the product was cheap on object storage; whatever the new primary index is, it must preserve those economics or the differentiation evaporates. Second, the strategy hiding in plain sight — "lay the foundation to move many more SQL queries to turbopuffer" is a quiet repositioning from vector-database-vendor to general search/query engine, which puts them on a collision course with different competitors entirely. Also worth noting: the market they helped define (RAG-era vector stores) is consolidating exactly this way — vector search commoditizing into a feature of general engines, mirrored on the embedded end by [[zvec]].

## Relations

- [[zvec]] — the opposite pole of the same market: where turbopuffer is dissolving the vector-database category upward into a general query engine, zvec dissolves it downward into an embeddable library; both suggest "vector database" as a standalone product is a transitional category.
- [[Just Brute Force Your Embeddings]] — that essay argued most teams don't need ANN at all; turbopuffer's own numbers (ANN demoted, non-vector plans starving under it) are the vendor-side confirmation that vector search is becoming one index among many, not the center of the system.
- [[Databases and Data]] — a textbook case study in primary-index choice and its three amplifications; strengthens the topic's storage-engine material with a rare public, mid-migration design narrative.
- [[Predicting the Future of Distributed Systems — Colin Breck (Tesla)]] — complements Breck's systems-forecasts with a concrete data-platform data point: at 1T+ documents and 10M+ writes/s, layout decisions (block sizes, write amplification) dominate algorithmic cleverness.

---
*Sources: [[raw/rip-vector-database]], [[summary/rip-vector-database]]*
*Last updated: 2026-10-02*
