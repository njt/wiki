---
title: "NornicDB"
url: https://github.com/orneryd/NornicDB
date_fetched: 2026-05-14
section: "Databases and Data"
---

# NornicDB: Graph + Vector + Temporal Database

## Main Purpose
NornicDB is a distributed, low-latency database combining graph, vector, and temporal capabilities in a single system. Implements Neo4j-compatible Bolt/Cypher protocols while adding vector search, historical reads via MVCC, and AI-native features.

## Core Value Proposition
Targets workloads requiring simultaneous graph traversal, vector retrieval, and historical truth -- particularly AI agent memory, knowledge graphs, and Graph-RAG systems. Rather than bolting vector functionality onto a graph database, NornicDB unifies these as co-equal execution paths.

## Key Features

**Protocol & Compatibility:**
- Neo4j Bolt protocol and Cypher query language support
- REST, GraphQL, and gRPC (Qdrant-compatible) interfaces
- Drop-in replacement for existing Neo4j applications

**Intelligent Capabilities:**
- Native vector search with GPU acceleration (Metal/CUDA/Vulkan)
- Memory decay systems using Ebbinghaus-Roynard four-layer decomposition
- Auto-relationship generation via embedding similarity, co-access patterns, and temporal proximity
- 950+ APOC functions

**Data Management:**
- Snapshot Isolation with MVCC for repeatable reads
- Canonical graph ledger model for temporal validity and audit trails
- Historical reads and as-of queries
- Conflict detection at commit time

**Advanced Features:**
- Heimdall AI assistant for natural language database interaction
- Knowledge-layer scoring with profile-driven decay and promotion
- Retention policies with GDPR erasure tracking
- Plugin system for custom extensions

## Performance

LDBC Social Network Benchmark (M3 Max, 64GB):
- Message content lookup: 6,389 ops/sec vs Neo4j's 518 (12x faster)
- Recent messages (friends): 2,769 ops/sec vs 108 (25x faster)
- Avg friends per city: 4,713 ops/sec vs 91 (52x faster)

Hybrid Retrieval (vector + 1-hop graph traversal): ~859 microseconds P50 (HTTP)

## Deployment
Multiple Docker image variants from minimal (~148 MB) to full with embeddings + AI assistant (~1.1 GB ARM64).

## Technology
Go (90.4%), MIT licensed with defensive patent non-assertion grant.
