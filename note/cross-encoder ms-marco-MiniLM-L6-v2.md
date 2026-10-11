# cross-encoder/ms-marco-MiniLM-L6-v2

The model card for a small (6-layer MiniLM) cross-encoder trained on MS MARCO passage ranking — the canonical open-source reranker for retrieve-and-rerank pipelines. It documents usage with sentence-transformers and transformers, and publishes a benchmark table where L6-v2 matches its 12-layer sibling's accuracy (74.30 vs 74.31 NDCG@10) at roughly twice the throughput (1,800 vs 960 docs/sec on a V100).

---

## What it is

A cross-encoder takes a *pair* — query and candidate passage — and outputs a relevance score, so unlike a bi-encoder it cannot be pre-indexed. It exists only to reorder candidates someone else fetched. The card says this plainly: encode the query against passages "retrieved with ElasticSearch", then "sort the passages in a decreasing order" — the classic SBERT.net Retrieve & Re-rank architecture.

## Key quotes

> "Given a query, encode the query with all possible passages (e.g. retrieved with ElasticSearch). Then sort the passages in a decreasing order."

The division of labour is stated as architecture, not marketing: cheap recall, expensive precision. This is exactly the two-stage retrieval pattern that modern RAG and agent-memory systems still use.

> cross-encoder/ms-marco-MiniLM-L6-v2 | 74.30 | 39.01 | 1800
> cross-encoder/ms-marco-MiniLM-L12-v2 | 74.31 | 39.02 | 960

The table's most interesting row is the one-step difference. Doubling depth buys 0.01 NDCG and costs half the throughput. It is a concrete instance of a pattern that recurs across small-model research: past a threshold, layers buy almost nothing on this task. It also strengthens the case in [[When Smaller Models Win]] — here the smaller model is not just cheaper, it is effectively free of accuracy loss.

> "The model can be used for Information Retrieval: Given a query, encode the query with all possible passages."

A warning disguised as an instruction. A cross-encoder scoring "all possible passages" is the naive strawman; the honest usage is behind a first-stage retriever. Anyone bolting this onto a corpus without a recall stage will discover why MRR@10 is measured at 10.

## Key themes

#concept (cross-encoder vs bi-encoder — pair scoring vs pre-computed embeddings) #tool (the model itself, Apache-2.0, distilled from L12) #pattern (retrieve-then-rerank as the default retrieval architecture)

## Opinion

This card is old (the v2 models date to ~2021) and its numbers are stale relative to modern rerankers — yet the model remains the default choice in countless RAG tutorials and production stacks, which is itself informative. The reason is the table: it makes the cost-accuracy frontier legible in one glance, and L6-v2 sits at the knee of that curve. The benchmark table also quietly dates the field — MRR@10 of 39 looks weak next to today's rerankers, but the *shape* of the trade-off (depth buys ~nothing, latency buys a lot) has not changed. What has changed is where the reranker sits: increasingly it is not a dedicated model but an LLM-as-judge call, which is far more expensive and not obviously better on this narrow task. The card's unglamorous code samples are a useful corrective to that.

## Related pages

- [[Scaling RAG — Chunking, Reranking, and Cost Optimization]] — this model is the concrete artifact behind that page's reranking stage; the card's docs/sec column gives the cost numbers that page argues about.
- [[Building a RAG Pipeline for Semantic Code Search (JetBrains)]] — JetBrains' semantic code search uses the same retrieve-then-rerank shape; this card supplies the canonical instance of the second stage.
- [[Just Brute Force Your Embeddings]] — that page argues exhaustive embedding search beats clever indexing; this card's cross-encoder costs cut the other way, showing why pure brute force needs a precision stage.
- [[RIP Vector Database]] — complicates the "reranker is dead" narrative: the two-stage pattern this model embodies survives the vector-database commoditisation intact.

---
*Sources: [[raw/readme-md]], [[summary/readme-md]]*
*Last updated: 2026-10-11*
