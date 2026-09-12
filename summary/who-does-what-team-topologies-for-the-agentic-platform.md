---
url: https://blog.owulveryck.info/2026/06/22/who-does-what-team-topologies-for-the-agentic-platform.html
title: "Who Does What? Team Topologies for the Agentic Platform"
author: Olivier Wulveryck
date_fetched: 2026-06-24
date_published: 2026-06-22
tags: [AI, agents, architecture, platform, team-topologies, agentic-engineering]
topics:
  - agent-coding-workflow
  - ai-product-and-business
---

# Who Does What? Team Topologies for the Agentic Platform

Olivier Wulveryck extends Team Topologies (Skelton & Pais) to the age of AI agents. He argues that cognitive load in agentic production isn't merely a *quantity to distribute* across teams but a *throughput to regulate over time* — an "anticipation burden" compressed onto single humans in short time windows. The agentic platform absorbs this burden so business teams can drive production via agents, while developers shift to building the platform itself.

## Full Content

### The real problem: the cognitive load of agentic production

Cognitive load does not disappear with AI — it transforms. It first becomes an "anticipation burden": everything a human must foresee before launching an agent, or the output will fall short. The developer who carries the anticipation burden knows which questions the agent will not ask.

The load also becomes a throughput problem. Agents produce continuously without human rhythm, creating a sustained flow of decisions over time that no single human can regulate.

### Team Topologies, an answer to the load

Wulveryck keeps the fundamental insight from Skelton & Pais: a team can only be effective if it carries no more complexity than it can absorb. But he adapts the model: the agentic platform defines *what* needs to be provided, and Team Topologies defines *who* provides it.

### Four team types, one objective

1. **Stream-aligned teams** — business/product teams that define intent and supply dynamic context (specifications, product-specific guardrails, domain knowledge). They no longer need developers and are *not* end-to-end operational — the platform absorbs operations.

2. **The platform team** — industrializes systemic context (instructions, roles, shared business knowledge, memory, examples, patterns), systemic guardrails (security, reliability, brand consistency, conventions), and tooling as self-service X-as-a-Service. The orchestrator itself is provided by the platform; its *configuration* stays with the product team.

3. **Enabling teams** — temporary function that trains and bridges the gap between business teams and production quality. Structurally compensated by the platform since stream teams remain non-technical. "Enabling disappears because it succeeds, not because it fails."

4. **Complicated subsystem teams** — deep technical specialists (models, KV cache, inference, evaluation) whose work flows through the platform, never directly to product teams.

### Three interaction modes

- **Facilitating** — enabling team teaches stream-aligned teams (temporary)
- **X-as-a-Service** — platform delivers consumable capabilities (target state)
- **Collaboration** — complicated subsystem teams co-build with the platform team (transitions to X-as-a-Service at maturity)

### Making the model last

**Journey toward autonomy**: The article describes a maturity progression where platform coverage grows and stream teams gain independence.

**Application governance**: Ease of production must be matched by ease of oversight. Shadow IT is the risk of ungoverned application production when non-technical teams can deploy; mitigated by platform's systemic visibility.

**Graduation path**: A guardrail discovered by one product team can be systemized into the platform. Governed by a "rule of three" — three teams needing the same guardrail triggers its integration. This prevents premature abstraction while ensuring discoveries propagate.

### Operational synthesis

**Who owns what**: Specifics (dynamic context, product guardrails, business knowledge) belong to stream-aligned teams. Systemics (systemic context, organizational guardrails, tooling, orchestrator) belong to the platform team.

**Platform maturity criteria**: Guardrail coverage, pipeline reliability with measurable SLA, majority of deployments self-service, complete documentation, and decision traceability (audit trail for blocked deployments).

### The Open Points of Debate

Wulveryck candidly includes counterarguments raised on Hacker News:

- **Agentic Waterfall**: Rigid role hand-offs between agents could mirror outdated Waterfall methodology
- **Over-engineering**: Topologies might be premature for a technology still in flux
- **Baseline quality dilemma**: Whether efforts should go into better defaults for future models rather than complex harnesses for current mediocre AI
- **Code degradation / "ghetto code"**: Risk of AI-generated codebases no human fully understands or maintains over time
- **Platform product owner bottleneck**: A single owner deciding what goes into the platform becomes a bottleneck
- **Porous what/how boundary**: The line between "what to build" (stream teams) and "how to build it" (platform) is inherently blurry

The article was originally written in French and AI-translated; Wulveryck acknowledges HN criticism of the translation as "word-salad" but leaves it unedited to "preserve the context of the HN discussion."
