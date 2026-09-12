---
url: https://xcancel.com/cerebras/status/2077822555159945507
title: "How We Built Our Knowledge Base"
author: Cerebras (@cerebras, authors: Isaac, Daniel, Zenghao)
date_fetched: 2026-07-25
date_published: 2026-07-16
topics:
  - agent-memory-and-context
---

Cerebras announced Cerebras Knowledge, an internal RAG system handling 15,000+ employee queries per day. The thread links to their full technical blog post on the architecture.

The announcement drew substantial technical discussion. Kevin Simback argued retrieval-based approaches can't reliably answer questions like "why was this invoice paid when it didn't match the PO?" — they locate text but don't resolve claims. He outlined a four-layer alternative: Evidence → Facts → Judgement → Access, where a derived, citable fact layer sits over raw communications.

Kirk Patrick described convergent evolution: an almost identical architecture built a month earlier on Cloudflare's stack (Workers, R2, D1, Vectorize, Workers AI) for ~$5–10/month, with additional guarantees — zero LLM calls in the query path, formal provenance on every derived artifact, and an "empty-never-nearest" contract that makes hallucination inexpressible.

Other replies touched on ACL-aware retrieval (balancing permission scope against cross-team value), the absence of structured domain/feature/workflow artifacts, the same context-gap problem one layer down for coding agents, and predictions of eventual migration to graph-based approaches.
