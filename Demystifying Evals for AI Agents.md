# Demystifying Evals for AI Agents

Anthropic's engineering team lays out a comprehensive framework for evaluating AI agents -- the definitive guide to moving from "vibes-based testing" to rigorous, repeatable evaluation. The piece matters because agent evals are fundamentally harder than LLM evals: agents are non-deterministic, take actions in the world, and find valid solutions their designers never anticipated. This is the playbook for anyone shipping agents in production.

---

## Key Quotes

> "More rigorous evaluation may even seem like overhead that slows down shipping. But after the early prototyping stages, once an agent is in production and has started scaling, building without evals starts to break down."

> "There is a common instinct to check that agents followed very specific steps like a sequence of tool calls in the right order. We've found this approach too rigid and results in overly brittle tests, as agents regularly find valid approaches that eval designers didn't anticipate."

> "At k=1, they're identical (both equal the per-trial success rate). By k=10, they tell opposite stories: pass@k approaches 100% while pass^k falls to 0%."

## Key Themes

#evals #agents #testing #LLMs #production-engineering

The core insight is that agent evaluation requires a fundamentally different mindset from traditional software testing. You grade *outcomes*, not *pathways*. An agent that solves the problem via an unexpected route isn't wrong -- your eval is too brittle. This connects directly to the challenges described in [[Scaling Long-Running Agents]] and [[How Hightouch Built Their Long-Running Agent Harness]], where agent behaviors emerge unpredictably at scale.

Three grader types form the toolkit: code-based (fast, cheap, brittle), model-based (flexible but non-deterministic), and human (gold standard but expensive). The practical advice is to start with 20-50 tasks from real failures, not synthetic benchmarks.

The pass@k vs pass^k distinction is particularly sharp: pass@k tells you "can it ever succeed?" while pass^k tells you "can I trust it?" For customer-facing agents, pass^k is what matters -- and it plummets fast.

## Critical Analysis

This is the best single resource on agent evals available. The real-world examples (Claude Code, Descript, Bolt) ground the theory in practice. The Swiss Cheese Model framing -- no single evaluation layer catches everything -- is exactly right.

What's missing: the piece underplays the cost of eval infrastructure. Building and maintaining a robust eval harness is a significant engineering investment, and the "start with 20-50 tasks" advice, while correct, understates how quickly that grows. The eval saturation problem (SWE-bench going from 30% to 80% in a year) also raises the question of whether we're building evals fast enough to stay ahead of model capability. The connection to [[Gambit]] is natural -- Gambit tries to solve the "generate and maintain evals" problem specifically.

---
*Sources: [[raw/demystifying-evals-for-ai-agents]]*
*Last updated: 2026-05-14*
