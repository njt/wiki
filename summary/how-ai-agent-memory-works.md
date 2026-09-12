---
url: https://memory.cobanov.dev/
title: How AI Agent Memory Works
author: Mert Cobanov
date_fetched: 2026-05-15
date_published: unknown
topics:
  - agent-memory-and-context
---

Interactive illustrated technical essay on how memory systems work for AI agents — the orchestration layer around LLMs that carries information forward across turns. Covers context windows, working vs. long-term memory, memory lifecycle governance, vector embeddings, four memory types (episodic, semantic, procedural, working), the RAG loop with production tricks (HyDE, RRF), retrieval pipelines, six architecture tradeoffs, multi-agent memory, and production considerations. Includes live interactive demos (embedding visualization, timeline navigation, drag-and-drop memory cards, latency sliders, multi-agent permission lab).

Core premise: "A language model on its own is stateless" — it forgets after every reply. An agent's memory is "the part of that loop that carries information forward." The central design question is: "what should we put in the prompt this time?"

Key quotes:
- "A frustrating agent forgets everything. A dangerous one remembers wrong."
- "Memory governance is what separates a one-off demo from a production agent."
- "In multi-agent memory, sharing buys collaboration, and grows the attack surface."

Memory types mapped from cognitive science: Episodic (time-stamped events), Semantic (facts and relations, vector search), Procedural (learned skills, tool invocation), Working (current scratchpad, in-prompt).

Production tricks: HyDE (embed a hypothetical answer instead of the question), Reciprocal Rank Fusion (combine dense, sparse/BM25, and graph retrievers by merging rankings).

Six architecture tradeoffs compared: Simple buffer, Rolling summary, Vector store, Knowledge graph, Hierarchical (MemGPT-style), Self-editing (Letta-style).

Multi-agent memory recommendations: private by default, shared memory explicit. Six failure modes enumerated.

Production: 800ms p95 latency target, three storage tiers (hot/warm/cold), minimum API surface (POST /memory/events, POST /memory/search, DELETE /memory/{id}).
