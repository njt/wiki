# CodeMira

A proof-of-concept adapting Mira OS's memory architecture for programming contexts, built for OpenCode. A Python daemon monitors coding sessions during idle periods, extracts patterns from tool-call transcripts via LLM, stores them in SQLite with vector embeddings (hnswlib) and full-text search (FTS5), and serves retrieval over HTTP. A TypeScript plugin injects relevant memories into future conversations.

---

## Key Quotes

> "Concepts are adapted from Mira and not ported 1:1 as a tool-call laden coding session is different than an ongoing conversation."

## Key Themes

#memory #vector-search #opencode #pattern-extraction #hybrid-retrieval

The honest acknowledgment that coding sessions are different from conversations is the key design insight. The extract-during-idle pattern (wait for the developer to stop, then process what happened) avoids the latency hit of real-time extraction. Hybrid retrieval (BM25 + vector similarity + Reciprocal Rank Fusion) is the right approach for code-related queries where both keywords and semantic meaning matter.

Project-scoped storage (`.codememory/memories.db` per project) keeps different projects' contexts separate, which is essential for developers who switch between codebases.

## Critical Analysis

This is the OpenCode equivalent of [[Claude-Mem]] -- same problem (persistent memory across sessions), different platform, different architecture. CodeMira's daemon-based approach is heavier than Claude-Mem's hook-based one, but potentially more thorough since it processes full session transcripts rather than individual tool calls.

The multi-provider support (Ollama, OpenRouter, vLLM, llama.cpp) is notable -- you can run the extraction and retrieval with local models, avoiding API costs for the memory overhead. Compare with [[Planning With Files]] for a simpler file-based approach to the same persistence problem. The tradeoff is clear: Planning With Files is simpler and more portable; CodeMira is richer and more automatic.

Early release, active development -- worth watching but not production-ready.

---
*Sources: [[summary/codemira]]*
*Last updated: 2026-05-14*
