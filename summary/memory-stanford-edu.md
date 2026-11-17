---
url: https://memory.stanford.edu/
title: "Memory beyond the context window (MEMORILLA)"
author: Fallahpour, Idehpour, Kothapalli, Mounier, Wu, Pardis, Leskovec (Stanford)
date_fetched: 2026-11-17
date_published: 2026
topics:
  - agent-memory-and-context
  - ai-research-and-models
---

Stanford's project page for **MEMORILLA** ("Latent Semantic Memory for LLMs", arXiv:2610.21375) presents a memory architecture that sits between full context and RAG. A frozen encoder stores each document (conversation, book chapter) as an embedding; a small trained module reads the question *and the entire store* and distills it into a fixed handful of memory tokens — sixteen in the supervised experiments — that the frozen LLM reads alongside the prompt.

The design is framed around four requirements: collection-wide access (each query reads the whole store, not pre-selected passages), query-dependent recall, a fixed token budget regardless of collection size, and reusable document embeddings (each document is encoded once; the store grows without re-encoding). Only the memory module is trained, staged from paraphrasing Wikipedia through QA and summarization to benchmark training splits.

Results on eight benchmarks (personalization, knowledge-intensive QA, long-document QA) with the same frozen Qwen3-8B behind every method: highest mean score (49.68), leading on five of eight, using 394× fewer document tokens than full context on average. The sharpest wins are where answers require connecting weakly-similar facts — PersonaMem-v2 (44.60 vs 30.90 for xRAG) and FactKG fact verification (84.3% vs 80.4% for *full context*, which sees every fact, using 108× fewer tokens). On TriviaQA it hits the best exact match with 743× fewer tokens. RAG still wins where the answer is a verbatim name; on short sources like PubMedQA, reading the text directly remains better.

In an agent setting, MEMORILLA condenses a Qwen3-4B TextWorld agent's growing history into eight memory tokens, trained with RL on game reward: held-out success rises from 18.6% to 24.2% and three-attempt completions from 25.0% to 37.5%, against a full-history baseline that sees the same transcript — so the gain is attributed to learned compression, not extra tokens. Untrained memory falls below baseline.

Stated limits: fine details like exact names can be lost; memory tokens are written directly into input embeddings, requiring open weights and a per-model trained module; all experiments pair a Qwen encoder with a Qwen decoder on text alone; the TextWorld agent is trained and evaluated on one game suite. The authors position learned memory as complementary to retrieval — 15.9% of FactKG claims are solved by MEMORILLA alone, 10.0% by retrieval alone — with hybrid combination as the next step.
