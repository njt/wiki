---
url: https://jamiemaguire.net/index.php/2026/08/01/rag-in-net-the-complete-series/
title: "RAG in .NET: The Complete Series"
author: Jamie Maguire
date: 2026-08-01
ingested: 2026-08-07
---

# RAG in .NET: The Complete Series — Summary

Jamie Maguire's index page collecting seven posts on building and operating production RAG pipelines in .NET, spanning February 2024 through July 2026. The series covers the full lifecycle: integrating RAG into Microsoft Agent Framework agents via `TextSearchProvider`, the chunking/embedding/production gotchas that tutorials skip, diagnosing and fixing 18–25-second query latency caused by duplicate-page context bloat, building a local-first RAG workbench (`DocIngestion`) to shorten the retrieval debugging loop, building an Elasticsearch-backed administration panel for managing index drift and "zombie" vectors, 26 hard-won lessons from the trenches across architecture/retrieval/operations, and implementing 100% local RAG with Phi-3 and ONNX embeddings via Semantic Kernel.

The series is unified by a single thesis: **the gap between a RAG tutorial and a RAG pipeline that stays healthy in production is substantial.** Maguire argues that most RAG tooling makes answering basic diagnostic questions ("why did this chunk retrieve?") too slow, and that the real work is in observability, administration tooling, and operational hygiene — not model selection or vector database choice.

Key technical contributions: the `GroupBy(c => c.DocumentUri)` deduplication fix that cuts token counts ~40%, the `IVectorStore` abstraction that defers infrastructure decisions until retrieval quality is understood, the zombie document problem in two-index architectures, deterministic chunk IDs from content hashes, and the case for JSON-file-backed vector stores during early iteration. The series also candidly discusses Claude Code as a development accelerator, crediting it with materially speeding up implementation while emphasizing that domain understanding remains essential.
