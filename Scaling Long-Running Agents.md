# Scaling Long-Running Agents

Cursor ran hundreds of AI coding agents simultaneously for weeks, generating over a million lines of code on single projects. The key finding: flat self-coordination fails catastrophically (agents hold locks forever, avoid hard work, slow to a crawl), but a planner/worker/judge hierarchy works. This is one of the most concrete accounts of multi-agent coding at scale.

---

## Key Quotes

> "Agents would hold locks for too long, or forget to release them entirely...Twenty agents would slow down to the effective throughput of two or three."

> "A surprising amount of the system's behavior comes down to how we prompt the agents."

> "The best system is often simpler than you'd expect."

## Key Themes

#agents #multi-agent #coding-agents #scaling #coordination

The planner/worker/judge pattern keeps recurring across agent systems. Planners map the codebase and generate tasks recursively, workers execute without needing to coordinate with each other, and judges evaluate whether work is done. This is almost identical to the architecture described in [[How Hightouch Built Their Long-Running Agent Harness]], where separated planning and execution was also the breakthrough.

The concrete results are staggering: a web browser from scratch (1M+ lines in a week), a Solid-to-React migration (266K additions/193K deletions in three weeks), a Windows 7 emulator (1.2M LoC ongoing). These are not toy demos.

The flat coordination failure is instructive: self-coordinating agents using shared files and locks reproduced every pathology of poorly-managed human teams -- bottlenecks, risk aversion, diffusion of responsibility. Hierarchy solved it.

## Critical Analysis

The results are impressive but raise questions about code quality. A million lines of generated code in a week is meaningless if the code is unmaintainable or architecturally incoherent. The piece focuses on throughput metrics (lines of code, files touched) rather than quality metrics. This is exactly where the eval framework from [[Demystifying Evals for AI Agents]] becomes essential.

The finding that GPT-5.2 outperforms competitors for extended autonomous work is interesting but model-specific -- this ranking shifts with every release. The more durable insight is that different models suit different roles (planning vs execution vs judgment).

The "prompt engineering matters more than system architecture" conclusion is both surprising and unsurprising. Surprising because you'd expect the coordination infrastructure to dominate at this scale. Unsurprising because, at the end of the day, these are still language models following instructions.

---
*Sources: [[raw/scaling-long-running-agents]]*
*Last updated: 2026-05-14*
