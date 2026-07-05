---
title: "qmd"
url: https://github.com/tobi/qmd
date_fetched: 2026-05-14
section: "Random"
---

# QMD: Query Markup Documents

Mini CLI search engine for your docs, knowledge bases, meeting notes, whatever. All local, tracking current SOTA approaches.

## Core Features
- **Full-text search (BM25)**: Fast keyword-based retrieval
- **Vector semantic search**: Understanding meaning beyond keywords
- **Hybrid search with re-ranking**: Combines both plus LLM-powered relevance scoring

## Architecture
1. Query Expansion: LLM generates variations for broader matching
2. Parallel Retrieval: Each variant searches both BM25 and vector indexes simultaneously
3. Ranking Fusion: Reciprocal Rank Fusion (RRF) with position-aware blending
4. LLM Re-ranking: Cross-encoder model scores final candidates with confidence metrics

Smart chunking preserves semantic units -- headings, code blocks, and sections stay together.

## Technical Requirements
- Node.js >= 22 or Bun >= 1.0
- Three GGUF models auto-download on first use (~2GB total):
  - EmbeddingGemma (300M) for vector embeddings
  - Qwen3-Reranker (0.6B) for relevance scoring
  - Custom QMD query-expansion model (1.7B)

## Key Features
- Configurable collections from any directory with custom glob patterns
- "Context" layers that provide metadata to improve search relevance
- Multiple output formats (JSON, CSV, Markdown, XML) for agent integration
- MCP server for Claude and other LLM clients
- SDK for embedding in Node.js/Bun applications
- GPU acceleration automatic, can be overridden

## Quick Start
```
npm install -g @tobilu/qmd
qmd collection add ~/notes --name notes
qmd embed
qmd query "search term"
```
