---
url: https://gist.github.com/f79da993aa0a786d0b1045651c8931f6
title: "Vedana — Domain Models as Agent Context"
author: Olga Tataranova
date_fetched: 2026-09-13
topics:
  - agent-memory-and-context
  - mcp-and-tool-protocols
---

Olga Tataranova (co-founder of Epoch8) presents Vedana, their product for giving LLMs correct, traceable answers to domain-specific questions — the kind where a wrong answer is expensive: legal, e-commerce, manufacturing, automotive. The diagnosis is blunt: dumping documents into ChatGPT produces prose that sounds right but is wrong, because the model has no business logic.

The fix is not better chunking but a structured domain model. Epoch8 uses "minimal modeling" (Alexey Mahotkin's notation): **anchors** (nouns), **attributes**, and **links** (relationships) that describe how a domain works. The model is expert-crafted, stable, and maintained by domain experts rather than engineers — it changes only when a business process changes.

Grist is the documentation and collaboration layer, not the query engine. It was chosen because it combines a business-user-friendly UI, typed columns, open-source licensing, API access, Python friendliness, and role-based access — requirements that databases, spreadsheets, and CMSs each fail individually. The model lives in Grist and is fed to LLMs two ways: (1) as a prompt to extract structured entities and links from documents, and (2) as a context file — the whole Grist SQLite database, or the Grist MCP — so the agent can explore the schema, craft Cypher/SQL queries, and fetch exact answers from Memgraph (the production graph database) instead of guessing from text chunks.

This shifts answers from "sounds right" to "is right, and here's the source." Text chunks are kept only as a fallback layer whose quality drops sharply. The admitted bottleneck: expertise doesn't scale, and the model itself "can't be generated" — it must be built with domain experts, then reused as playbooks across similar businesses.
