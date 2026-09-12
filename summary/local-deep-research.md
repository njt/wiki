---
url: https://github.com/LearningCircuit/local-deep-research
title: Local Deep Research
author: LearningCircuit
date_fetched: 2026-08-21
date_published: 2025
topics:
  - agent-architecture
---

# Local Deep Research — Précis

## What it is

Local Deep Research (LDR) is an open-source, MIT-licensed Python web app for "deep, agentic research": you ask a complex question, and it searches the web, academic databases, and your own documents, then synthesizes a cited report. Its defining promise is **privacy and control** — it runs entirely locally (Ollama + SearXNG), stores every user's data in a per-user AES-256-encrypted SQLCipher database, and makes no telemetry calls. It ships as Docker images, `pip install local-deep-research`, or a Flask web UI on `localhost:5000`.

The headline result: the first open-source project to report **~95% SimpleQA (n=500) and 77% xbench-DeepSearch on a single RTX 3090** running Qwen3.6-27B fully local, using its `langgraph-agent` strategy.

## How it works

- **Research engine** — `AdvancedSearchSystem` (`search_system.py`) coordinates a pluggable **strategy** (`BaseSearchStrategy`). Five user-facing strategies: `source-based`, `focused-iteration` (×2), `topic-organization`, and `langgraph-agent` (autonomous tool-calling).
- **`langgraph-agent`** — the flagship: LangChain's `create_agent()` builds a tool-calling agent whose tools are *search engines themselves*. It spawns parallel **subagents** via a `research_subtopic` tool, each with its own search/fetch tools, and a thread-safe `SearchResultsCollector` dedupes sources across subagents.
- **Search engines** — a plugin system over `BaseSearchEngine`, ~25 engines (arXiv, PubMed, Semantic Scholar, Wikipedia, SearXNG, Tavily, Brave, GitHub, Wayback…), plus local collections and LangChain retrievers.
- **Knowledge base** — research findings feed an encrypted "library": download sources, extract text, chunk + embed (sentence-transformers + FAISS), then search collections as engines next time ("your knowledge compounds over time").
- **Security** — a DLP-style egress guardrail (ADR-0007) classifies every source and sink on two axes (Sensitivity × Exposure) and forbids sensitive data from reaching exposing sinks. Per-user encrypted databases, CSRF, SSRF/metadata-IP blocking, and 22+ CI security scanners.

~189K lines of Python. Python ≥3.12, requires AVX-capable CPUs for the bundled scientific wheels.

## Distinctive angles

- **Egress as a first-class architecture concern** — an in-process PDP (`security/egress/policy.py`) with PEPs at every boundary, a PEP-578 audit hook that classifies raw `socket.connect` targets, and honest documentation that it is "a correctness guardrail, NOT a hard security boundary."
- **Per-user encrypted DB threading** — one SQLAlchemy `QueuePool` per user (not one global pool), because each user has their own SQLCipher file with a unique key; the FD-budget analysis and cleanup layers are documented in `docs/architecture.md`.
- **Benchmark-driven** — a community-maintained HuggingFace leaderboard tracks accuracy across models, engines, and strategies, so users can compare local models before downloading weights.

*Source: [[raw/local-deep-research]] — verbatim README. Analysis from full repo clone.*
