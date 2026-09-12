---
url: https://handbook.sutro.sh/
title: "Analytical AI — What It Is and Why It Matters"
author: Sutro
date_fetched: 2026-09-04
site: handbook.sutro.sh
topics:
  - guardrails-and-feedback-loops
---

Sutro's handbook introduces **Analytical AI** as a distinct usage pattern that emerged alongside the 2022 "ChatGPT moment" but got less attention: data, research, ops, and product teams using foundation models to process unstructured data and make scaled operational decisions. The one-sentence definition: **if the AI's job is to decide something rather than create something, it's analytical AI.**

## Why it matters

The distinction isn't semantic pedantry — best practices genuinely diverge from generative use cases, for three reasons:

1. **Tasks are measurable.** You can build a ground-truth dataset from expert annotations and validate against it for correctness. Other generative outputs aren't directly measurable, which is why you must build evals to measure them — and evals are themselves a special case of analytical AI.
2. **Tasks are discriminative, not generative.** You use an LLM's autoregressive reasoning and instruction-following to make decisions, but reduce "creativity" in favor of consistency. So you run the smallest possible model that's been evaluated for task accuracy, rather than reaching for the largest.
3. **Latency is tolerated.** No user transaction means batch and flexible workload processing is acceptable, often saving enormously on cost and processing time — analogous to OLTP vs. OLAP / map-reduce data processing.

## Audience and structure

Written for data/ML/analytics teams transforming unstructured datasets into structured ones; AI engineers and PMs building evals; ops teams scaling domain-expert decisions; and research teams building judges and verifiable reward functions. The handbook is organized into four sections — **Primitives** (core workload types), **Patterns** (implementation best practices), **Architectures** (end-to-end system guides), and **Deployment** (production operations) — framed by Sutro as an evolving FAQ gathered from the trenches with customers, independent of tooling choice.
