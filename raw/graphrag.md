---
title: "GraphRAG"
url: https://microsoft.github.io/graphrag/
date_fetched: 2026-05-14
section: "Databases and Data"
---

# GraphRAG: Structured Retrieval Augmented Generation (Microsoft)

## Core Innovation
GraphRAG departs from traditional semantic-search RAG by constructing knowledge graphs from raw text, establishing community hierarchies, and generating community summaries to enhance language model reasoning.

## Problem Statement
Baseline RAG approaches struggle in two critical scenarios:
1. Connecting disparate information pieces requiring synthesis across shared attributes
2. Providing holistic understanding of summarized concepts across large datasets

## Competitive Advantage
Demonstrates "substantial improvement in answering" complex questions compared to vector-similarity-based retrieval methods, particularly for private datasets.

## Process

**Indexing Phase:**
- Converts input corpus into analyzable TextUnits
- Extracts entities, relationships, and claims using LLMs
- Applies hierarchical clustering via Leiden technique
- Generates bottom-up community summaries

**Query Modes:**
- Global Search: answers corpus-wide questions using community summaries
- Local Search: explores specific entities and neighbors
- DRIFT Search: combines local exploration with community context
- Basic Search: fallback to traditional vector retrieval

## Notable Points

"Using GraphRAG with your data out of the box may not yield the best possible results" -- recommends prompt tuning optimization. The project requires configuration updates between releases.
