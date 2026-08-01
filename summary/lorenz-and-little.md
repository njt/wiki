---
url: https://brooker.co.za/blog/2026/07/29/lorenz-and-little.html
title: "Lorenz and Little: How Much Does Your Tail Cost?"
author: Marc Brooker
date_fetched: 2026-08-01
date_published: 2026-07-29
---

Marc Brooker adapts the Lorenz curve — an economics tool for measuring income inequality — to latency analysis. For any percentile P, 1 − L(P) gives the fraction of mean latency contributed by requests at or above that threshold. A worked example shows p50 requests contribute ~99% of mean latency, p90 ~93%, p99 ~52%, and p99.9 ~10%.

Through Little's law, contribution to mean latency equals contribution to concurrency: if 1 − L(p99) = 0.5, then half of all busy server threads are serving p99+ requests. Brooker notes this often exceeds 0.5 or even 0.75 in real services.

The core insight is that tail requests are disproportionately expensive to serve — they drive concurrency, capacity demand, and lock contention. Rather than trimming tails, the post argues for optimizing them to reduce costs.

Includes Python code for computing 1 − L from measured quantiles with power-law interpolation, an interactive widget, and honest caveats about interpolation error, estimation uncertainty with heavy tails, and the post-hoc nature of the Little's law mapping.
