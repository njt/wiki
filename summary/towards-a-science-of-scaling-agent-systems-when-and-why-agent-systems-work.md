---
url: https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/
title: "Towards a Science of Scaling Agent Systems: When and Why Agent Systems Work"
author: Yubin Kim and Xin Liu, Google Research
date_fetched: 2026-09-25
date_published: 2026-01-28
topics:
  - agent-orchestration
  - ai-research-and-models
---

Google Research reports on a controlled evaluation of 180 agent configurations across three model families (OpenAI GPT, Google Gemini, Anthropic Claude) and four benchmarks (Finance-Agent, BrowseComp-Plus, PlanCraft, Workbench), testing one single-agent architecture and four multi-agent variants (independent, centralized, decentralized, hybrid). The headline: "more agents" is not a universal win.

Key findings:

- **Parallelizable vs sequential tasks.** On parallelizable tasks like financial reasoning, centralized coordination improved performance by 80.9% over a single agent. On strictly sequential tasks (planning in PlanCraft), *every* multi-agent variant degraded performance by 39–70% — communication overhead fragments the reasoning and leaves insufficient "cognitive budget" for the actual task.
- **Tool-coordination trade-off.** As tasks require more tools (e.g. 16+ for a coding agent), the coordination "tax" of multiple agents grows disproportionately.
- **Error amplification.** Independent (non-communicating) multi-agent systems amplified errors 17.2×; centralized systems with an orchestrator contained amplification to 4.4×. The orchestrator acts as a "validation bottleneck" that catches errors before they propagate.
- **Predictive model.** Using measurable task properties (tool count, decomposability), a model with R² = 0.513 picks the optimal architecture correctly for 87% of unseen task configurations — a step toward principled rather than heuristic architecture choice.
- **Smarter models don't remove the need for multi-agent systems**; they change when it pays, and only with the right architecture.

The paper is framed as the first quantitative scaling principles for agent systems, moving the field from heuristics ("more agents are better") to task-property-driven architecture selection.
