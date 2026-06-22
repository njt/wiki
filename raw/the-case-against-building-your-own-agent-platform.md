---
url: https://www.oreilly.com/radar/the-case-against-building-your-own-agent-platform/
title: The Case Against Building Your Own Agent Platform
author: Pete Johnson
date_fetched: 2026-06-22
date_published: 2026-06-17
---

# The Case Against Building Your Own Agent Platform

**Author:** Pete Johnson
**Published:** June 17, 2026
**Source:** O'Reilly Radar

## Section Headers & Structure

1. **Build versus buy, flipped in a year**
2. **Most "agent platforms" aren't**
   - Memory
   - Governance
   - Eval
   - Orchestration
3. **The honest case for building**
4. **Five questions before you commit**
5. **What this looks like in 2 years**
6. **Sources** (primary and secondary)

## Key Arguments

### The Pendulum Pattern

Johnson argues this build-vs-buy cycle has repeated across tech history (app servers, CMS, container orchestration). "When a category is new, the components look deceptively simple. Early adopters build their own." He notes that in 2024, 47% of enterprise AI solutions were built in-house, but by late 2025 that "collapsed to 24%."

### Workflows vs. Agents (Core Distinction)

Drawing on Anthropic's guidance, Johnson separates workflows (LLMs "orchestrated through predefined code paths") from agents (LLMs that "dynamically direct their own processes"). He warns that teams building for workflows later get "asked to support agents" and discover the jump is far from incremental.

### The Four Underestimated Components

**Memory** — "sounds like a database problem. It isn't." Real production memory splits into episodic, semantic, and procedural systems with distinct retention policies, plus temporal reasoning and deduplication. Evidence of category maturity: Mem0 raised $24M, Letta $10M, and the competitive landscape includes 21 frameworks across three hosting models.

**Governance** — Extends far beyond "RBAC plus audit logging." Requires "action authorization, not just data authorization," decision-chain auditability, behavioral drift detection, and tiered autonomy. Johnson flags that Grant Thornton found 78% of executives lack confidence they'd pass an AI governance audit within 90 days. The EU AI Act's full enforcement for high-risk systems arrives August 2026. OWASP now lists "excessive agency" as a top vulnerability class.

**Eval** — "Agent evaluation is qualitatively different from traditional software testing." For multi-agent systems, you evaluate "system dynamics, including coordination patterns and collective invariants." Google Vertex AI standardized `trajectory_exact_match`, `trajectory_precision`, and `trajectory_recall` — metrics that "didn't exist 18 months ago." Gartner projects 60% of software engineering teams will adopt AI evaluation platforms by 2028, up from 18% in 2025.

**Orchestration** — "hasn't converged." LangGraph, CrewAI, OpenAI Agents SDK, AutoGen, Google ADK, Claude SDK, and Microsoft's Agent Framework each represent different bets on state management and coordination; migration between them means "rewriting most of your agent logic." The Model Context Protocol and agent-to-agent (A2A) protocols are "moving targets."

### The Honest Case for Building

Johnson acknowledges valid reasons: proprietary data as a competitive moat, regulated industries needing full-stack control (HIPAA, GxP, SOX, etc.), and avoiding vendor lock-in. But his key distinction: "Those are arguments for building agents on top of platform components, not arguments for building the platform components themselves."

## Five Specific Recommendations (Questions to Ask)

1. **Workflow or agent platform?** — "They're not the same scope, and conflating them is where most of the cost overruns originate."
2. **Can you define "done" for each component?** — Memory, governance, eval, orchestration, each in three sentences. "If you can't, you don't have requirements. You have a vibe."
3. **What happens when you swap the underlying model?** — Menlo data shows Anthropic went from 12% to 40% of enterprise LLM spend while OpenAI fell from 50% to 27%. Hardcoded assumptions mean "simultaneous rewrites across memory, eval, and orchestration."
4. **What happens when techniques shift?** — 18 months ago the default was "RAG with flat vector retrieval"; now it's "just-in-time context strategies, agent-managed memory tiers, and trajectory-based evaluation."
5. **What happens when the platform team leaves?** — "Agent platforms are a particularly bad candidate for this pattern because the talent pool is both small and mobile."

## Key Data Points Cited

| Stat | Source |
|---|---|
| Enterprise in-house AI builds dropped from 47% to 24% in one year | Menlo Ventures, 2025 |
| 57% of organizations have agents in production | LangChain, 2026 |
| 32% cite quality as top deployment barrier | LangChain, 2026 |
| 78% lack confidence in passing AI governance audit within 90 days | Grant Thornton, 2026 |
| 40%+ of agentic AI projects projected to be canceled by 2027 | Gartner, June 2025 |
| Anthropic went 12%→40% of enterprise LLM spend (2023→2025) | Menlo Ventures |
| OpenAI fell 50%→27% over same period | Menlo Ventures |
| Eval platform adoption: 18% (2025) → projected 60% (2028) | Gartner |

## Notable Quotes

- "The cost of building this is almost always estimated before anyone has a clear picture of what 'this' actually is."
- "Within 18 months, building becomes the expensive path."
- "Memory sounds like a database problem. It isn't."
- "RBAC was designed for humans with predictable intent. Agents don't have predictable intent."
- "Your eval system needs its own eval system."
- "If you built your own orchestration layer in 2024, you're rewriting it in 2026."
- "The only durable bet in this space is the one that assumes the bet will change."
- "Build the things that are specific to your business. Buy the things that are specific to the technology category."
