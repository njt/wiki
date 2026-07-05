---
title: "CodeMira"
url: https://github.com/taylorsatula/CodeMira
date_fetched: 2026-05-14
section: "LLMs"
---

# CodeMira: Developer Memory System

Proof-of-concept adapting Mira OS memory architecture for programming contexts. Learns from coding sessions and surfaces relevant context when needed.

## Architecture

**Python Daemon:**
- Monitors OpenCode sessions during idle periods
- Extracts patterns from tool-call transcripts via LLM
- Stores in SQLite with vector embeddings (hnswlib) + full-text search (FTS5)
- Serves retrieval over HTTP

**TypeScript Plugin:**
- Integrates with OpenCode's message transformation pipeline
- Analyzes conversation intent via LLM
- Queries daemon for relevant memories
- Injects context into message stream

## Features
- **Project-Scoped Storage**: `.codememory/memories.db` per project
- **Hybrid Retrieval**: BM25 + vector similarity via ANN + Reciprocal Rank Fusion
- **Multi-Provider**: Ollama, OpenRouter, vLLM, llama.cpp, OpenAI-compatible
- **Deduplication**: Fuzzy matching + vector similarity

## Origin
"Concepts are adapted from Mira and not ported 1:1 as a tool-call laden coding session is different than an ongoing conversation."

Early release, active development.
