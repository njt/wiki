# GraphRAG

Microsoft's structured approach to Retrieval Augmented Generation: instead of naive vector similarity search over text chunks, build a knowledge graph from your documents, cluster entities into communities, generate summaries at each level of the hierarchy, then use those structures for retrieval.

---

## Key Themes

#vector-db #graph-db #knowledge-management #agent-architecture

GraphRAG addresses two specific failures of baseline RAG:

1. **Cross-document synthesis** -- connecting information scattered across documents that share attributes but not surface-level text similarity
2. **Holistic summarization** -- answering "what is this corpus about?" rather than "find me the chunk most similar to this query"

The indexing pipeline is LLM-heavy: it uses language models to extract entities, relationships, and claims from raw text, then applies Leiden clustering to build community hierarchies, then generates bottom-up summaries for each community. This is expensive but produces a structured representation that naive chunking cannot match.

Four query modes cover different use cases: Global Search (corpus-wide questions using community summaries), Local Search (entity-specific exploration), DRIFT Search (hybrid of local + community context), and Basic Search (fallback to traditional vector retrieval).

Connects to [[NornicDB]] (graph + vector in a single engine -- the database GraphRAG could sit on top of), [[zvec]] (the naive vector search that GraphRAG explicitly improves upon), and [[Data Engineering for Large Models]] (the data pipeline that creates the documents GraphRAG indexes).

## Critical Analysis

The "substantial improvement" over vector-similarity RAG is real, but the costs are significant: LLM calls during indexing, the complexity of maintaining a knowledge graph, and the need for prompt tuning per corpus. The honest warning that "using GraphRAG out of the box may not yield the best possible results" is refreshing -- most RAG tools oversell their zero-config experience. The main limitation is that the knowledge graph is only as good as the LLM's entity extraction, which means errors in extraction propagate through the entire hierarchy. For private datasets where you can invest in tuning, GraphRAG is likely worth it. For general-purpose RAG, the complexity tax may not pay for itself.

---
*Sources: [[summary/graphrag]]*
*Last updated: 2026-05-14*
