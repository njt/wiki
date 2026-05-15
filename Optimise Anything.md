# Optimise Anything

"If it can be serialized to a string and its quality measured, an LLM can reason about it and propose improvements." A universal optimization API (optimize_anything) that combines LLM reasoning with structured diagnostics and Pareto-efficient search. Extends GEPA (Genetic-Pareto) from prompt optimization to any measurable text artifact -- code, agent architectures, configurations, algorithms.

---

## Key Quotes

> "If it can be serialized to a string and its quality measured, an LLM can reason about it and propose improvements."

## Key Themes

#optimization #llm #pareto #agent-evolution #concept

Two core mechanisms make this work:

**Actionable Side Information (ASI)**: Instead of collapsing all feedback to a single score, evaluators return structured diagnostics -- error messages, execution traces, rendered images. The LLM proposer gets targeted feedback, not just "this scored 0.7."

**Pareto-Efficient Search**: Maintains a frontier of candidates excelling at different metrics. This prevents averaging from hiding strengths. A solution that's best at memory usage but mediocre at speed survives alongside one that's fastest but memory-hungry.

The results across eight domains are the argument: coding agent skills to near-perfect pass rates (47% faster), ARC-AGI from 32.5% to 89.5%, CUDA kernels where 87% match or beat baseline, circle packing outperforming AlphaEvolve. The breadth is the point -- this isn't tuned for one domain.

## Critical Analysis

The "serialize to string, measure quality, let LLM improve" framing is deceptively simple. The hard part is the evaluator -- you need a function that reliably measures quality for your specific artifact. The API handles the optimization loop; you provide the judgment.

The Pareto search is genuinely better than single-objective optimization for most real problems. When you're optimizing a prompt, you care about accuracy AND cost AND latency. Single-objective optimization forces you to collapse those into a weighted average; Pareto search gives you the tradeoff frontier.

This connects to [[Feedback Loop is All You Need]] -- both argue that the system around the agent matters more than the agent itself. Optimise Anything automates the improvement loop; Feedback Loop automates the quality enforcement loop. Together they'd create a system that both improves and guards against regression.

See also [[agent-pr-replay]] for empirical measurement of agent performance gaps.

---
*Sources: [[raw/optimise-anything]]*
*Last updated: 2026-05-14*
