# Towards a Science of Scaling Agent Systems (arXiv Paper)

The full arXiv paper (v3, April 2026) behind the January 2026 Google Research blog post: 260 controlled configurations across six agentic benchmarks, five architectures and three LLM families, yielding a predictive scaling model (R²=0.373) whose headline lesson is that architecture-task alignment — not agent count — decides whether multi-agent coordination helps or hurts.

---

## What it does

The team (Yubin Kim and colleagues) runs controlled evaluations across 260 configurations spanning six agentic benchmarks, five canonical architectures — Single-Agent plus four Multi-Agent topologies (Independent, Centralized, Decentralized, Hybrid) — and three LLM families. Tools, prompts, and compute are standardized so the architectural effect is isolated rather than confounded with scaffolding differences. The result is fit as a quantitative scaling model: cross-validated R²=0.373 across all six benchmarks, R²=0.413 with a task-grounded capability metric.

## Findings that matter

1. **Capability saturation.** Coordination yields diminishing returns once the single-agent baseline exceeds a threshold. If one model already does the task well, adding agents buys coordination overhead, not capability.
2. **Tool-heavy tasks pay a multi-agent tax.** Tasks dense in tool calls appear to incur multi-agent overhead — parallelism doesn't amortize the coordination cost when the bottleneck is the tool loop, not reasoning.
3. **Decentralized topologies propagate errors.** Architectures without centralized verification leak errors downstream; centralized coordination contains them.
4. **The spread is enormous and task-dependent.** Relative performance versus the single-agent baseline ranges from **+80.8%** on decomposable financial reasoning to **−70.0%** on sequential planning. Architecture is not a free choice; it's a bet on task structure.
5. **The model is predictive.** It identifies the best architecture for 87% of held-out configurations, and architecture preferences transfer consistently to unseen frontier models.

## Opinion

This is the paper version of a result the wiki already holds from the blog write-up — and the paper is more honest than most "multi-agent systems" marketing. The R²=0.373 is modest by physics standards and the authors don't pretend otherwise; that a model with that much variance still picks the right architecture 87% of the time tells you the *ranking* of architectures is far more learnable than the exact performance. The deepest implication is unflattering to the multi-agent hype cycle: the single-agent baseline is the strongest predictor of whether you should build a multi-agent system at all. The negative results (−70% on sequential planning) are the finding most teams will ignore, and the finding most worth internalizing.

---

This paper is the longer, more general sibling of the blog version captured in [[Towards a Science of Scaling Agent Systems]] — it strengthens that page by widening from 180 configurations/three benchmarks to 260/six, at the cost of a slightly lower fit (R²=0.373 vs. 0.513), which itself nuances how much to trust the model. It complicates [[Silo-Bench — The Communication-Reasoning Gap]] by quantifying the same coordination-overhead erasure of parallelization gains across many more architectures. It gives empirical teeth to [[Structural Backpressure Beats Smarter Agents]]'s argument that structure, not raw capability, governs outcomes — here centralized verification is precisely a backpressure mechanism. And it sharpens [[Agent Orchestration for the Timid]]'s cautionary posture with numbers: mismatched coordination doesn't just waste effort, it can destroy 70% of single-agent performance.

---
*Sources: [[raw/2512-08296]], [[summary/2512-08296]]*
*Last updated: 2026-09-25*
