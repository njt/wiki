---
url: https://github.com/AttemorySystem/attemory
title: Attemory — Attention-Native Memory Retrieval System
author: Lance Fang
date_fetched: 2026-08-06
topics:
  - agent-memory-and-context
---

# Attemory — Summary

Attemory is an **attention-native semantic retrieval engine** that replaces nearest-vector lookup with model attention as the core retrieval primitive. Rather than embedding similarity, BM25, or vector databases, it indexes raw corpora into reusable KV (key-value) cache state, then runs a local Qwen3.5 retrieval model that attends over both the indexed memory and the query. The result is a different retrieval paradigm: search through the same attention mechanism LLMs use to reason over context, not through compressed embeddings.

The system runs as a local service (C++ core with a Python HTTP client), splits large sessions into segments internally, and supports persistence via KV cache snapshots to disk. It operates at two levels: a general retrieval engine API (Python/HTTP) for memory, documents, and custom corpora; and a repository search tool (`atcode`) that indexes codebases into searchable chunked memory for coding agents, with a Claude Code plugin.

Benchmark results are the strongest signal: **98.72% session recall on LongMemEval-S**, **92.55% message recall on LongMemEval-M** (million-token scale where few systems test), **94.52% accuracy on LoCoMo**, and **0.9055 NDCG@10 on Semble** for code retrieval. On SWE-QA, a single Attemory code-search hint reduced Claude Code token consumption by **43.8%** (285M → 160M tokens) with near-identical judge quality, purely by giving the agent better file/line hints before exploration began.

The trade-off is clear: Attemory requires running a local retrieval model (inference cost), but gains retrieval quality through attention over raw context rather than compressed similarity. It's MIT-licensed, built on llama.cpp/ggml and Qwen, and published by Lance Fang in 2026.
