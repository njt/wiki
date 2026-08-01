---
url: https://huggingface.co/blog/ibm-research/model-routing-is-simple-until-it-isnt
title: "Model Routing Is Simple. Until It Isn't."
author: Yara Rizk, Eyal Shnarch, Jason Tsay, Merve Unuvar (IBM Research)
date_fetched: 2026-07-18
date_published: 2026-07-15
---

IBM Research argues that model routing — sending easy queries to cheap models and hard ones to expensive models — is not a classification problem but a systems optimization problem. Three dimensions complicate routing beyond the naive approach.

**Cost is more than model pricing.** The authors benchmarked GPT-4.1 against Claude Sonnet 4.6 on 417 AppWorld tasks and found Sonnet cheaper overall ($79 vs. $155) despite higher per-token pricing and longer reasoning trajectories, because Sonnet's lower cache-read pricing dominated in agent workloads with heavy context reuse. A router that looks only at pricing sheets optimizes against the wrong numbers.

**Complexity is more than task difficulty.** Difficulty is often invisible at routing time — a simple-seeming request may cascade into retrieval, compliance checks, and tool use. Routers must also juggle cost, latency, specialization, reliability, and enterprise constraints (compliance, data residency, approved model lists) simultaneously.

**Latency is more than model speed.** Infrastructure factors — hardware, cache warmth, endpoint load — often dominate response time. A theoretically faster model can produce a slower experience under suboptimal serving conditions. Routing granularity matters: per-task routing adds minimal overhead, but per-step routing compounds latency at each decision point.

The authors reframed routing as an optimization problem balancing cost, quality, and latency. Their lightweight algorithm (6 ms, 2 kB per task) found operating points on a cost-accuracy frontier that a standard difficulty-based router could not reach — achieving 84% accuracy for $93 and 83 seconds, a 21% cost and 9% latency reduction versus running Opus alone with only a 4% accuracy drop.

The core lesson: routing is about optimizing the entire system, not picking the best model for a single task. Models are one variable among many — caching behavior, infrastructure state, compliance constraints, and workload patterns all matter.
