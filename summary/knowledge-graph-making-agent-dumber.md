---
url: https://praveenvijayan.substack.com/p/your-knowledge-graph-is-making-your
title: "Your Knowledge Graph Is Making Your Agent Dumber"
author: Praveen Vijayan
date_fetched: 2026-07-25
date_published: 2026-07-24
topics:
  - agent-memory-and-context
---

Praveen Vijayan put Graphify — a tool that converts a codebase into a knowledge graph for AI agents — through a realistic test on a 605-file TypeScript monorepo. He asked three real development questions and compared graph-based queries against plain ripgrep + manual file reads.

The graph was impressively built: 1,842 nodes, 3,722 links, LLM-labeled communities. But it fell apart on query time. A question about OAuth provider registration returned 170 nodes, only 8 on-topic. Barrel files dominated the graph's degree rankings, and a BFS from any entry point flooded the results with irrelevant imports. Community clustering was barely better than random (~0.03–0.08 cohesion) despite confident-sounding LLM labels.

Graphify did have wins: it clustered plan documents into named workstreams (the standout feature), detected import cycles cleanly, and served as a useful onboarding artifact via its `GRAPH_REPORT.md`. Query cost was near-zero since no API calls were involved.

The core problem is structural: barrel-export monorepos produce hub poisoning in import-graph traversal. The author's closing thesis: "Graphify is an excellent document knowledge graph and a mediocre code knowledge graph." His recommended workflow treats the graph as an onboarding tool, then switches to `rg` + reading for actual code queries.

*Sources: [[raw/knowledge-graph-making-agent-dumber]]*
*Last updated: 2026-08-01*
