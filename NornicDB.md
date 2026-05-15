# NornicDB

A database that unifies graph traversal, vector search, and temporal queries in a single engine. Neo4j-compatible (Bolt/Cypher) with native vector search and built-in memory decay -- designed for AI agent memory, knowledge graphs, and Graph-RAG systems.

---

## Key Quotes

> "6,389 ops/sec vs Neo4j's 518 (12x faster)" -- message content lookup, LDBC benchmark

## Key Themes

#graph-db #vector-db #memory #agent-architecture

NornicDB's most interesting feature isn't the performance (though 12-52x faster than Neo4j on benchmarks is notable). It's the memory decay system. Using Ebbinghaus-Roynard four-layer decomposition, knowledge fades over time unless reinforced -- exactly how human memory works, and exactly what AI agent memory systems need. Instead of an ever-growing context that eventually overwhelms retrieval, old memories naturally fade unless they prove their relevance through repeated access.

The hybrid retrieval story (vector + 1-hop graph traversal in sub-millisecond) matters because real-world queries are rarely pure vector search or pure graph traversal. "Find me things semantically similar to X that are also connected to Y" is a natural question that requires both capabilities working together.

The Neo4j compatibility (Bolt protocol, Cypher queries) is a smart adoption strategy -- teams can try NornicDB without rewriting their application layer. The multiple deployment sizes (148MB minimal to 1.1GB full with embeddings) show thoughtful packaging for different use cases.

Connects directly to [[GraphRAG]] (structured retrieval that benefits from graph + vector), [[zvec]] (pure vector search -- simpler but less powerful), and the agent memory discussion in [[Process-Based Concurrency BEAM OTP]] (where isolated stateful processes need persistent memory).

## Critical Analysis

The benchmarks are impressive but self-reported on an M3 Max -- production workloads on commodity hardware will tell a different story. The MIT license with defensive patent non-assertion is unusual and worth understanding before production adoption. The Ebbinghaus-based memory decay is the genuine innovation here; most graph databases treat all data as equally important forever, which doesn't match how knowledge actually works. The Go implementation (90.4%) suggests good performance characteristics but limits the contributor pool compared to a Rust or C++ engine. The feature surface is vast -- 950+ APOC functions, Heimdall AI assistant, GDPR erasure tracking -- which raises the question of whether this is trying to do too much for a young project.

---
*Sources: [[raw/nornicdb]]*
*Last updated: 2026-05-14*
