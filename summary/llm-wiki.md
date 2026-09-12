---
title: "llm-wiki"
url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - agent-memory-and-context
  - databases-and-data
---

# LLM Wiki: Pattern for AI-Maintained Personal Knowledge Bases

By Andrej Karpathy. Proposes a three-layer architecture where LLMs incrementally build and maintain persistent markdown wikis as a synthesis layer between raw sources and the user.

## Key Distinction from RAG

Rather than re-retrieving and re-synthesizing from raw documents on every query, the system compiles knowledge once into structured wiki pages, then keeps that compilation current as new sources arrive.

## Architecture: Three Layers

1. Raw sources (immutable): Articles, papers, files the user curates
2. The wiki (LLM-owned): Generated markdown pages with summaries, entity pages, cross-references
3. Schema document (configuration): Defines wiki structure and workflows (e.g., CLAUDE.md)

## Key Operations

- Ingest: New sources trigger multi-page updates -- LLM reads source, extracts key info, writes summaries, updates indices, revises related pages
- Query: Users ask questions; LLM searches wiki pages, synthesizes answers, optionally files answers back as new pages
- Lint: Periodic health checks identify contradictions, stale claims, orphaned pages, missing cross-references

## Why This Works

Offloads tedious maintenance -- updating cross-references, maintaining consistency across dozens of pages -- to LLMs, which don't tire or forget. Humans curate sources and ask questions; LLMs handle bookkeeping.

## Implementation Details

- index.md: Content-oriented catalog of all wiki pages with one-line summaries
- log.md: Append-only chronological record of ingests, queries, and maintenance
- Optional tooling: local full-text search (qmd), Obsidian integration, version control via git

## Community Response

Spawned dozens of implementations: contradiction detection (sigma-guard), token compression (sqz), hierarchical routing (synthadoc), typed knowledge graphs with provenance, multi-agent support, MCP server integrations.

## Acknowledged Limitations

Transactional overhead from cascading updates, referential integrity challenges, temporal blindness in large archives, risk of "dead links" without rigorous maintenance infrastructure.

## Practical Workflow

Users typically run Obsidian on one side for browsing the wiki and its graph view, while an LLM agent operates on the other side making edits in real time.

Human role: source curation, question direction, synthesis review
LLM role: reading, extraction, cross-referencing, page maintenance
