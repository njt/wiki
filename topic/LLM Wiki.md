# LLM Wiki

Andrej Karpathy's pattern for AI-maintained personal knowledge bases: a three-layer architecture where LLMs incrementally build and maintain persistent markdown wikis from raw sources. The human curates sources and asks questions; the LLM does the grunt work of summarizing, cross-referencing, filing, and bookkeeping. The key distinction from RAG: knowledge is compiled once into structured pages and kept current, rather than re-synthesized from raw documents on every query.

---

## Key Quotes

> "You never (or rarely) write the wiki yourself -- the LLM writes and maintains all of it. You're in charge of sourcing, exploration, and asking the right questions."

## Key Themes

#knowledge-management #personal-wiki #obsidian #rag-alternative #llm-patterns

This is the pattern this wiki itself implements. Three layers: raw sources (immutable), the wiki (LLM-owned markdown pages), and a schema document (CLAUDE.md) defining structure and workflows. Three operations: ingest (new source triggers multi-page updates), query (LLM searches wiki, synthesizes answers), lint (periodic health checks).

The insight that makes it work: LLMs don't tire or forget, so the tedious maintenance that kills human-maintained wikis (updating cross-references, maintaining consistency, catching contradictions) becomes tractable. The human does the interesting work (choosing what to read, asking questions); the LLM does the boring work (filing, linking, formatting).

The community response -- dozens of implementations addressing contradiction detection, token compression, hierarchical routing, provenance tracking -- validates the pattern's generativity. See [[Elements of Agentic Systems Design]] for the formal framework (particularly Memory and Artifacts elements).

## Critical Analysis

Strong: the pattern works. This wiki exists because of it. The three-layer separation is clean and the operations (ingest/query/lint) are intuitive. Using Obsidian's graph view alongside the LLM agent is a practical workflow.

Weak: cascading updates from single changes can be expensive (tokens and time). Referential integrity without rigorous infrastructure is fragile -- "dead links" are a real risk as the wiki grows. The pattern assumes a single user with a single LLM maintaining everything; multi-user or multi-agent scenarios need transactional guarantees the markdown-file approach doesn't provide.

The deepest limitation: the LLM's synthesis is only as good as the sources. Garbage in, well-formatted garbage out. The human's curation role is understated in the original proposal.

---
*Sources: [[summary/llm-wiki]]*
*Last updated: 2026-05-14*
