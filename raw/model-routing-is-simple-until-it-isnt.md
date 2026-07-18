---
url: https://huggingface.co/blog/ibm-research/model-routing-is-simple-until-it-isnt
title: Model Routing Is Simple. Until It Isn't.
author: Yara Rizk, Eyal Shnarch, Jason Tsay, Merve Unuvar (IBM Research)
date_fetched: 2026-07-18
date_published: 2026-07-15
---

Model routing — sending easy queries to cheap models and hard ones to expensive models — seems straightforward but quickly becomes a systems optimization problem rather than a classification problem. IBM Research identifies three dimensions that make routing hard: cost is more than model pricing, complexity is more than task difficulty, and latency is more than model speed.

## Cost Is More Than Model Pricing

The authors tested GPT-4.1 against Claude Sonnet 4.6 on 417 tasks from the AppWorld Test Challenge using the same CodeAct agent. Despite GPT-4.1 having lower token pricing and Sonnet needing about three times the reasoning steps, Sonnet was cheaper overall: $79 total ($0.19/task) vs. GPT-4.1 at $155 ($0.37/task). The reason was caching. Agent workloads often reuse large context blocks across steps, and Sonnet's lower cache-read pricing gave it a major advantage that outweighed both its higher base pricing and longer trajectories. The authors conclude that "actual cost depends on the interaction between the model, the workload, and the serving infrastructure" — a router looking only at pricing sheets is optimizing against the wrong numbers.

## Complexity Is More Than Task Difficulty

The article identifies two problems with difficulty-based routing. First, "difficulty is often invisible at routing time." A request that seems simple (e.g., summarizing a contract) may require retrieval, compliance checks, tool use, and multiple refinement rounds, while a technically complex prompt might be handled well by a smaller specialized model. Second, even with perfect difficulty estimation, routers in production must balance cost, latency, specialization, and reliability simultaneously, plus enterprise constraints like compliance, data residency, privacy, and approved model lists. The authors state that "routers aren't solving one problem" — they juggle multiple competing priorities at once.

## Latency Is More Than Model Speed

User experience depends on more than model size. Routing itself adds overhead, and infrastructure factors (hardware, cache warmth, endpoint load) often dominate response times. A theoretically faster model can still produce a slower user experience if serving conditions are suboptimal. Additionally, routing granularity matters — routing once per task adds minimal overhead, but routing at every step introduces more latency and complexity with each decision point. The key insight: "a router that ignores the serving system is optimizing against the wrong reality."

## The Optimization Approach

The team shifted from treating routing as classification to treating it as "an optimization problem." Rather than asking which model is best for a task, their algorithm optimizes across cost, quality, and latency simultaneously while staying lightweight. Results on AppWorld Test Challenge demonstrate a cost-accuracy frontier with multiple operating points. Configuration 1 (latency-optimized) achieved "84% accuracy for $93 and 83s" — a 21% cost reduction and 9% latency reduction versus running Opus alone, with only a 4% accuracy drop. A standard difficulty-based router landed at similar accuracy but higher cost, unable to explore the full tradeoff space. The optimization itself is lightweight — roughly 6 ms and 2 kB of memory per task.

## The Bigger Picture

The authors' core lesson: "routing isn't really about choosing models. It's about optimizing systems." Models are one variable among many — caching behavior, infrastructure state, compliance constraints, and workload patterns all matter. When routing works well, it's because it found "the best operating point for the entire system" rather than the best model for a single task.

## Acknowledgement

The post was influenced by conversations with colleagues whose questions, feedback, and insights helped refine the authors' thinking.
