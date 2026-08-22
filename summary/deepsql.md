---
url: https://github.com/DeepSQLAI/deepsql
title: "DeepSQL"
author: DeepSQL (DeepSQLAI)
date: 2026-08-22
date_fetched: 2026-08-22
---

DeepSQL is a self-hosted "database agent" for PostgreSQL and MySQL. You point it at a database and ask questions in plain English; it answers BI queries, analyzes slow queries, recommends indexes, and watches your schema. The pitch is that you bring your own model (OpenAI, Azure OpenAI, Anthropic, LiteLLM, or a local Ollama/vLLM/LM Studio server) while everything else — database credentials, the vault, the agent runtime — runs in your own infrastructure.

The stack is a monorepo: a Spring Boot 4 backend on Java 25 (~221K lines of Java), a React 19 frontend, a Node.js MCP server + CLI (`@deepsql/mcp`, 44 tools), and an "agent" layer that is a heavily customized Nous Hermes Agent runtime carrying a DBA persona (`SOUL.md`) plus six procedural skills. There are no prebuilt images — Compose builds everything from source, and the first build takes minutes.

Its defining technical posture is correctness-over-autonomy. Every LLM-generated SQL passes through deterministic fences: a schema whitelist that drops hallucinated tables and columns, an `EXPLAIN`-based validator, and a read-only session enforced by the database itself (`connection.setReadOnly(true)`) so parser gaps can't become data loss. Mutation is possible only through the hand-written SQL editor, only for admins, with two-step confirmation.

The project is an existence proof against the pessimism of [[Text-to-SQL in the Real World]]: rather than betting that an LLM can assemble a correct enterprise query, it constrains the LLM to drafting and lets deterministic machinery own the safety-critical decisions.
