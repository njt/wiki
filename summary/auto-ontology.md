---
url: https://github.com/NVIDIA/auto-ontology
title: "NVIDIA Auto Ontology"
author: NVIDIA CORPORATION & AFFILIATES
date_fetched: 2026-11-03
date_published: 2026 (ongoing open-source project, Apache-2.0)
topics:
  - coding-agents-and-frameworks
  - agent-memory-and-context
---

NVIDIA Auto Ontology is an open-source library (Apache-2.0, ~120K lines of Python plus a Next.js frontend) that builds a **semantic/ontology context layer** over enterprise databases so that analytics agents answer questions correctly, not just quickly. Its thesis: agents can already retrieve documents and query databases, but what gates reliability is knowing where data lives, what it means, whether to use it, and where it came from.

**What it does.** From ingested metadata it compiles an ontology — business terms, SQL attributes, semantic relationships, foreign-key suggestions — stored in Postgres/pgvector alongside the catalog it describes. The "Learn" loop folds certified corrections and usage back in, so shared context improves with every question. The layer is exposed three ways: an MCP server for harnesses like Cursor and Claude Desktop, a REST API with hashed API tokens, and a chat web UI.

**Stack.** Next.js 16 frontend (Better Auth, Prisma, SSO, RBAC), FastAPI backend with LangGraph driving the text-to-SQL pipeline, a separate ingestion worker, Postgres+pgvector, optional HashiCorp Vault for credentials. Models default to NVIDIA NIM triplets (nemotron-3-nano-30b-a3b reasoning, nemotron-3-embed-1b, llama-nemotron-rerank), configured per role via `*_ENDPOINT/_API_KEY/_MODEL` triplets falling back to `DEFAULT_MODELS_*`.

**The retrieval flow.** A large LangGraph state machine (`retrieval/text_to_sql/text_to_sql_graph.py`, ~25 nodes) runs intent classification → candidate retrieval (vector search over semantic objects, LLM-filtered) → evidence refinement → SQL construction from certified terms → validation/parse/value checks → execution, with up to 7 reconstruction attempts and "soft" post-construction checks that stop blocking once too many attempts accumulate. A KumoRFM predictive-query branch is wired in only when configured. Conversation grounding rewrites contextual follow-ups using the five most recent turns (capped at 12K chars).

**Agent-facing design.** The MCP server is a deliberately curated allow-list of ~a handful of tools from an 87-operation OpenAPI spec — no raw schema browsing, because tool that lists columns invites the agent to bypass the semantic layer. Tool names and descriptions are hand-written as prompts, generated from the spec with CI enforcing drift. Three skills (`nvidia-ontology-setup/-management/-query`) teach coding agents how to deploy and drive it; the repo ships its own AGENTS.md/CLAUDE.md split.

Not accepting external contributions; benchmark via the separate ontology-sql-eval repo (BIRD).
