---
url: https://github.com/arm/metis
title: "Metis — AI-Powered Security Code Review"
author: Arm Product Security Team (open-source-office@arm.com)
date_fetched: 2026-07-08
date_published: 2025
topics:
  - guardrails-and-feedback-loops
---

Metis is an open-source, agentic AI security framework from Arm's Product Security
Team for deep security code review. It detects subtle vulnerabilities in large,
complex, or legacy codebases using a hybrid of deterministic static analysis and
LLM reasoning.

The core insight is the architecture's split by language tier. C and C++ files go
through a tree-sitter–based reachability analysis: the tool builds a call graph,
finds paths from untrusted sources to dangerous sinks, and only then asks the LLM
to confirm whether each reachable path is actually exploitable. This filters what
the LLM sees, avoiding the cost and noise of dumping full files into context. All
other languages (19 plugins, from Python and Java to Solidity and SystemVerilog)
get a generic LLM review with language-specific prompts.

Under the hood, Metis is a Python 3.12+ CLI app built on LangChain and LangGraph.
The review and triage pipelines are deterministic StateGraphs (3-node for review,
2-node for triage) — not open-ended agent loops. The triage system applies
deterministic adjudication rules after the LLM decision, downgrading verdicts to
inconclusive when evidence coverage is insufficient. This guards against
overconfident model output.

Findings flow in and out as SARIF, making Metis composable with existing SAST
tools. Providers and language plugins are discovered via Setuptools entry points,
allowing third-party extensions without modifying core code. The default vector
store is ChromaDB (local), with PostgreSQL+pgvector available for multi-project
deployments. Repository-level memory uses LangGraph's BaseStore protocol backed by
SQLite with fingerprint-based deduplication.
