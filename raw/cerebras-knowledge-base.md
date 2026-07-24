---
url: https://xcancel.com/cerebras/status/2077822555159945507
title: How We Built Our Knowledge Base
author: Cerebras (@cerebras, authors: Isaac, Daniel, Zenghao)
date_fetched: 2026-07-25
date_published: 2026-07-16
site: X/Twitter (via XCancel)
---

# How We Built Our Knowledge Base

Cerebras announcement thread for their technical blog post on building Cerebras Knowledge, an internal RAG system handling 15,000+ employee queries per day. The full technical blog is available at https://www.cerebras.ai/blog/how-we-built-our-knowledge-base

Thread date: July 16, 2026. 2.5M views, 4,577 likes, 447 retweets.

## Key Reply Threads (selected)

### Kevin Simback on ontology vs retrieval

Kevin Simback argues the system is well-designed but surprised Cerebras took a retrieval approach over an ontology-based system. His follow-up thread defines a "shared context layer" / "company brain" as a derived, citable fact layer over all communications and documents — distinct from systems of record. His four-layer architecture: Evidence (raw data) → Facts (entities and claims) → Judgement (reasoning across evidence) → Access (what AI can reason over). Argues retrieval-based systems will not be reliable on questions like "why was this invoice paid when it didn't match the PO?" — they find text but don't solve claims.

### Kirk Patrick on convergent evolution

Landed on almost exactly the same architecture a month prior: unified Postgres embeddings, hybrid retrieval, reranker, MCP serving LLM-free retrieval primitives. Went further with: append-only canonical store before indexing, formal provenance on every derived artifact, zero LLM calls in the query path, a verified output layer that makes hallucination inexpressible (empty-never-nearest contract, abstention typed as Found|Gap), and governance via the ARSIA Protocol aligned with EU AI Act and GDPR. Full stack on Cloudflare (Workers, R2, D1, Vectorize, Workers AI) for ~$5-10/month.

### Other notable replies

- Prashanth Chandrasekar (Stack Overflow): Stack Internal has the same multi-source ingestion philosophy with a strong human validation layer. Bidirectional MCP is particularly interesting.
- Stanislav Sorokin: Retrieval is only the start; useful systems must maintain context through real work. Links to his Kimi K3 coding agent review.
- Brandon Kase: Curious about exposing MCP tools as composable CLI/code-mode APIs for more token-efficient agent interactions.
- Nisarg: How to do ACL-aware retrieval while keeping context broad for cross-team value — "tricky balance."
- Kushal: Surprised not to see structured artifacts around domains, features, workflows, teams — creates those today with update loops.
- Nav: Same problem one layer down for coding agents — Claude Code only sees repo and current session, not Slack decisions or teammate context. Built basethread.ai as shared context layer.
- SurgicalAI: Building a surgical knowledge base with 4-hour surgery context maintenance — asking about real-time VLM knowledge base architecture.
- Andrew Altshuler: Predicts eventual migration to graph — "you pretty much throw away accuracy you already have for free."
- Saanvi Mehra: Curious about score-based methods vs RRF at different k values.
- Vansh Mittal: What's the daily cost of ingestion and maintaining the search index?
- Deepak Gupta: How do you handle permissions on an index with data from so many tools?
