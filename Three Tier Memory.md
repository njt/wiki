# Three Tier Memory

A research paper presenting a concrete architecture for persistent agent memory, developed during construction of a 108,000-line C# system across 283 development sessions. The three tiers: a hot-memory constitution (660 lines, always loaded), 19 specialized domain-expert agents (9,300 lines total) invoked per task, and a cold-memory knowledge base of 34 specification documents (~16,250 lines) queried on demand via an MCP retrieval server.

---

## Key Quotes

> "LLM-based agentic coding assistants lack persistent memory: they lose coherence across sessions, forget project conventions, and repeat known mistakes."

## Key Themes

#agent-architecture #memory #context-management #mcp

This paper tackles the most practical problem in agent-assisted development: how do you make agents remember what they've learned? The three-tier approach mirrors how human organizations handle knowledge -- everyone knows the core rules (hot), specialists know their domains (warm), and reference docs exist for deep dives (cold).

The architecture maps directly to [[Components of a Coding Agent]]'s "structured session memory" component but goes further by providing scaling numbers. 660 lines always loaded is feasible; 26,000+ lines of context available on demand is not something you can stuff into a system prompt.

[[napkin]] solves a simpler version of this problem (one markdown file per repo), while this paper solves the full version (tiered retrieval across a large codebase). The MCP retrieval server for cold memory is particularly interesting -- it's the same pattern as RAG but purpose-built for codified project conventions rather than general knowledge.

## Critical Analysis

The numbers are the real contribution here. Lots of people wave hands about "memory systems for agents" -- this paper shows exactly how much context at each tier, across how many sessions, at what scale. The 108K-line C# system is big enough to be credible. The weakness is that it's a single case study from one developer's workflow, so the generalizability is uncertain. But as a reference architecture, it's solid -- and the open-source companion repo means you can actually try it.

---
*Sources: [[raw/three-tier-memory]]*
*Last updated: 2026-05-14*
