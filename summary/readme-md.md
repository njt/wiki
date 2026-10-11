---
url: https://huggingface.co/cross-encoder/ms-marco-MiniLM-L6-v2/blob/main/README.md
title: "cross-encoder/ms-marco-MiniLM-L6-v2 Model Card"
author: Sentence Transformers (UKPLab)
date_fetched: 2026-10-11
date_published: 2021
topics:
  - agent-memory-and-context
  - local-and-open-source-inference
---

This is the Hugging Face model card for `cross-encoder/ms-marco-MiniLM-L6-v2`, a compact cross-encoder trained on the MS MARCO passage ranking task. It is meant to be used as the re-ranking stage of a retrieve-and-rerank pipeline: a cheap first pass (e.g. ElasticSearch or a bi-encoder) fetches candidates, then this model scores each (query, passage) pair and re-sorts them.

The card is mostly usage and benchmarks. It shows drop-in code for both `sentence-transformers` (`CrossEncoder.predict` on query-passage pairs) and plain `transformers` (`AutoModelForSequenceClassification` over tokenized pairs), plus a performance table comparing the whole family of MS MARCO cross-encoders against third-party rerankers on TREC DL 2019 NDCG@10 and MS MARCO MRR@10. The headline trade-off: MiniLM-L6-v2 scores 74.30 NDCG@10 and 39.01 MRR@10 while reranking 1,800 docs/sec on a V100 — essentially matching its 12-layer sibling (74.31/39.02) at nearly double the throughput. The model is Apache-2.0, English-only, and distilled from the L12 version.
