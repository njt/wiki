---
url: https://db.zizka.ai/
title: "ZizkaDB — Two Ways to Run ZizkaDB"
author: Zizka
date_fetched: 2026-10-18
date_published: unknown
topics:
  - guardrails-and-feedback-loops
  - coding-agents-and-frameworks
---

ZizkaDB is a vendor product page for an agent-monitoring database aimed at "teams building with agents": production agents need operational data, not just logs, and ZizkaDB "monitors behavior so your team can correct it." It logs from Python, TypeScript, LangChain, CrewAI, LiveKit voice agents, MCP, or REST — one API key per agent — and can be self-hosted via Docker Compose or run as managed cloud, with a single-tenant VPC "Fleet" offering on top.

The pitch is built around incident debugging: when a prompt change breaks agent behavior, replay the full session and trace the decision chain to root cause. Out of the box the dashboard shows Activity, Behavior, Reports, and Suggestions.

The most substantive part is a self-scored comparison table against Laminar, LangSmith, Langfuse, and Arize Phoenix. ZizkaDB claims differentiation on causal lineage (a "why()" query rather than a span tree), time travel over logged state, native drift baselines, first-party EU AI Act mapping, and AGPL self-hosting — scoring itself 5/5 while competitors score 1–2. The table itself carries a caveat: "~ = partial support. Verify competitor docs before external debates."
