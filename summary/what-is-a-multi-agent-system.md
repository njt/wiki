---
url: https://www.media.mit.edu/articles/what-is-a-multi-agent-system/
title: "The Identity Crisis (What Is a Multi-Agent System?)"
author: Ayush Chopra
date_fetched: 2026-09-25
date_published: unknown
topics:
  - agent-orchestration
  - agent-architecture
---

Ayush Chopra (MIT PhD candidate) argues that the multi-agent systems field is having an identity crisis: as LLMs dominate AI discourse, researchers rebrand orchestrated applications as "multi-agent systems" while missing what made classical MAS genuinely multi-agent — that how agents interact matters more than how smart each agent is.

Most current frameworks follow an agent-centric design: build individual agents as objects with sophisticated capabilities (reasoning, planning, memory), then compose them through orchestrator workflows. Chopra calls this "object-oriented programming with natural language interfaces" — Action Agents, Planning Agents, Orchestrator Agents — where interaction becomes an afterthought implemented as APIs between independent components. Real distributed coordination (supply chains, financial markets, social movements) instead emerges from interaction patterns that no central planner designed: the intelligence is in the patterns themselves, not any individual decision-maker.

His proposed inversion is "interaction-first design": make the interaction pattern the primary abstraction, with agents as participants in patterns rather than independent entities. This enables emergent rather than orchestrated coordination and naturally handles multi-scale dynamics (local interactions aggregate into network patterns that feed back to constrain local decisions, across heterogeneous protocols like APIs, message queues, and blockchains). He points to his AgentTorch framework as a practical demonstration, treating interaction patterns as differentiable, composable primitives.

Key takeaway: current MAS frameworks build "smart objects with workflows"; genuine distributed intelligence requires "interaction patterns that enable emergent coordination." The right abstraction for multi-agent systems isn't the agent — it's the interaction.
