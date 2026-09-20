---
url: https://martinfowler.com/articles/making-data-ready-for-agentic-ai.html
title: "Making Your Data Ready for Agentic AI"
date_fetched: 2026-09-20
date_published: 2026-08-27
topics:
  - databases-and-data
  - mcp-and-tool-protocols
---

A Martin Fowler article (27 August 2026) arguing that three decades of data architecture were built for human analysts — consumers who supply implicit context, tribal knowledge, and skepticism for free — and that autonomous agents supply none of it. When data looks wrong, a human double-checks; an agent confidently acts. The article's central claim is that every bit of that implicit human labor must be engineered into the data itself, as five attributes: trusted, contextual, traceable, governed, and operational.

It works through four topics in build order. Data contracts as code (with freshness SLAs keyed to last successful load, not last change), a quarantine pattern routing contract violations to dead-letter queues, and a medallion architecture where agents see only Gold tier and above — extended to vector indexes, whose freshness clock measures re-indexing, not content change. Traceability via agentic lineage (traces and spans capturing why, not just what), staged autonomy from shadow mode to full autonomy, and delegated per-user access with just-in-time credentials to break Willison's "lethal trifecta." A context layer of three models — domain (nouns), semantic (numbers), capability (verbs) — all in version control, with retrieved text informing but never gating actions. And an access spectrum from retrieval through real-time query to governed write-back, designed as capabilities rather than naive API-to-MCP wrappers.

The closing argument: data architecture *becomes* AI architecture when agents are the primary consumers, and unowned data artifacts drift at machine speed.
