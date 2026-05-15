---
url: https://openai.com/index/inside-our-in-house-data-agent/
title: "Inside Our In-House Data Agent"
author: Emma Tang (Data Platform Lead, OpenAI) and Venkat Venkataramani (VP of App Infrastructure, OpenAI)
date_fetched: 2026-05-15
date_published: 2026-04-17
---

Original URL returned HTTP 403. Content reconstructed from:

- Forbes / Yahoo Tech: "OpenAI Says Codex Agents Are Autonomously Running Its Data Platform" by Victor Dey (April 17, 2026). https://tech.yahoo.com/ai/chatgpt/articles/openai-says-codex-agents-autonomously-132131476.html
- Digital Watch Observatory: "OpenAI streamlines data analysis with in-house AI agent" (February 2, 2026). https://dig.watch/updates/openai-streamlines-data-analysis-with-in-house-ai-agent
- CDP Institute summary: https://www.cdpinstitute.org/news/inside-open-ais-in-house-data-agent/

## Summary

OpenAI's internal data platform — powering model training, safety pipelines, product analytics, and financial reporting across 600+ petabytes and ~70,000 datasets for 3,500+ users — is now run autonomously by Codex-powered AI agents built on GPT-5.2. The agents monitor pipelines in real time, trace anomalies, restart jobs, generate fixes, and validate them. Emma Tang (data platform lead) describes the central challenge: making "the company's data reality legible to the agent." Unlike coding agents with bounded repo contexts, data agents must reconstruct the entire company's data foundation — metadata, lineage, permissions, dashboards, query history, operational knowledge — before answering a question.

## Key Quotes

Emma Tang on the data agent's context:

> "It can draw on table definitions, ownership, documentation, query history, lineage, dashboards, permissions and the production code that generates the data."

Tang on the unified platform:

> "The lake, metadata, lineage, code, permissions, query execution, dashboards and notebooks are treated as one connected system."

Tang on the hard problem:

> "The hard part is making the company's data reality legible to the agent."

> "It still inherits the limits of the data foundation, i.e., missing metadata, pipeline definitions missing from code, siloed data across systems."

Venkat Venkataramani on the oral tradition risk:

> "Codex becoming an oral tradition with a better UI — where nobody knows why the workflow works or when it is stale."

> "A system can be effective and still become dangerous if humans can no longer reason about it."

Tang on treating agents as partners:

> "The risk is real only if teams treat agents as answer machines instead of reasoning partners."

## Specific Agents Described

1. **Release Agent** — Manages Apache Spark system updates. Gradual rollout, verifies stability over hours/days, generates PRs, notifies teams for review.
2. **On-Call Assistant** — Always-on agent retrieving context from past incidents (prior fixes, escalation paths, failure modes) and applying it to new issues in real time.
3. **Development Environment Agents** — Spin up local services, launch browser sessions, test UI changes, validate behavior before human review.
4. **Data Analysis Agent (Kepler)** — Natural-language data exploration using MCP, RAG, and vector search. Helps employees move from complex questions to reliable insights in minutes, replacing manual SQL-heavy workflows.

## Technical Architecture

- Foundation model: GPT-5.2 with built-in memory, self-validation, and continuous evaluation
- Context system: Combines metadata, human annotations, code-level insights, and institutional knowledge
- Memory: Stores "non-obvious corrections" so the agent improves over time and avoids repeated mistakes
- Validation: Self-validation against trusted "golden" sources (verified dashboards), plus automated evals comparing outputs to benchmarks
- Security: Strong access controls aligned with existing data permissions; transparent reasoning
- Scale: Event volumes grew ~50x year-over-year across streaming systems
- Self-validation where possible, and artifacts exposed for review: assumptions made, chain of thought, generated queries, citations, confidence levels

## Competitive Context

- Terminal-Bench 2.0: OpenAI agent 77.3% vs. Claude 65.4%
- SWE-bench: Anthropic leads mid-to-upper 60% range
- Anthropic leads long-context reasoning; OpenAI leads deterministic logic, factual retrieval, multi-tool orchestration
