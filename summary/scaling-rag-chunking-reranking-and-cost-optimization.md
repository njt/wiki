---
url: https://trpevski.com/blog/scaling-rag-chunking-reranking-and-cost-optimization
title: "Scaling RAG: Chunking, Reranking, and Cost Optimization"
author: trpevski.com (unattributed)
date_fetched: 2026-09-04
ingested: 2026-09-04
topics:
  - agent-memory-and-context
---

# Scaling RAG: Chunking, Reranking, and Cost Optimization — Summary

A practitioner's field guide to running RAG in production, written from the failure cases: systems that work on a 10-document test set and "fall apart" on real data. The core argument is that most teams treat RAG as "throw documents in a vector DB and ask questions" — which is a prototype, not a system. Production RAG is a hundred small decisions, and the post walks through the four that matter most.

**Chunking determines everything downstream.** Fixed-size chunks (512 tokens, 50–100 overlap) split sentences mid-thought and ignore document structure. The post presents three tiers: naive fixed-size, semantic chunking (embed sentences, split where adjacent-sentence cosine similarity drops below a threshold), and a production hybrid that splits on markdown headers first, chunks semantically within sections, and attaches metadata for filtering/ranking.

**Embedding costs leak money.** Teams pick expensive models and re-embed everything on every update. At OpenAI prices (text-embedding-3-small $0.02/M tokens, large $0.08/M), a local model that is 1–2% worse at retrieval is usually the right call — the loss is acceptable once a reranker is in the loop.

**Retrieval is where 73% of failures happen.** Hybrid retrieval (vector + BM25, fused by reciprocal rank with 60/40 semantic/keyword weighting) hit 82% recall vs 64% vector-only and 71% BM25-only on a real project. Reranking with a cross-encoder then orders candidates correctly — a 10–30% quality gain at a 50–200ms latency cost.

**Cost optimization is mostly about the stack, not the model.** On a 500k-document system, the "expensive" stack (text-embedding-3-large + Pinecone Pro + Cohere rerank) cost $684/month at 78% recall; the "optimized" stack (local embeddings + self-hosted Weaviate + local reranker) cost $120/month at 81% recall — better quality, 5.7× cheaper, with operational complexity as the tradeoff. The post closes with three debugging playbooks (irrelevant retrieval, >500ms latency, growing cost) and the note that LLM generation usually dominates cost (~60%), so shrinking context is the cheapest lever.
