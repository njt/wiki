---
url: https://gist.github.com/njt/34460de457cb2f63069845db0e3f7d4d
title: "Application Performance Optimisation in Practice"
author: Steve Gordon (Microsoft MVP, Pluralsight author, Engineer at Elastic)
date_presented: 2026-01
published: 2026-08-07 (gist)
tags: #performance #dotnet #profiling #benchmarking #software-craft
topics:
  - software-engineering-craft
---

Steve Gordon's NDC Copenhagen 2026 talk is a practitioner's field guide to performance engineering as a proactive discipline. The centerpiece is the **Performance Optimization Loop**: start with production monitoring data → identify the problem → profile CPU and memory → ensure tests exist → benchmark for a baseline → apply small, targeted changes → re-benchmark and re-test → document → deploy and validate with production data. Gordon applies this loop to a real OpenTelemetry SQL sanitizer in .NET, reducing allocations by 86% and CPU time by 46% across seven stages of incremental optimization.

The talk's framing is built on a precise reading of Knuth's "premature optimization" quote: forget small efficiencies 97% of the time, but don't pass up the critical 3%. Performance is contextual — it always matters, but the bar varies by business. Gordon makes the case that performance work should be shared ownership, not a specialist silo, with SLOs and alerts set before deployment and regression detection built into the pipeline.

The demo walks through distinct optimization stages: pre-sizing `StringBuilder` capacity, reusing a static `StringBuilder` with `Interlocked.CompareExchange`, replacing `Hashtable` with `ConcurrentDictionary` plus an approximate count, eliminating `Substring` allocations via `Span<T>` slicing, replacing `StringBuilder` entirely with rented `char[]` and span APIs, converting state classes to `ref struct` with `ref`/`in` parameters, using `SearchValues<T>` for keyword scanning, adding keyword metadata for algorithmic shortcuts, and returning the original string when sanitization is a no-op.

The AI-assisted optimization section is a notable practical data point: Gordon asked Copilot for 10 possible CPU/memory optimizations, got two insane suggestions and one hallucinated API, but seven were plausible and worth testing. Each AI-suggested change was manually verified with benchmarks and tests. The talk predates the post-AI-skepticism era — Gordon describes himself as "AI skeptical" when he first presented it in January 2026.

Unaddressed threads include: setting SLO thresholds, the engineering-cost vs. infrastructure-savings calculus, scaling the loop to distributed systems, and the cultural barrier of convincing management to invest in proactive performance work.
