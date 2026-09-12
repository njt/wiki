---
url: https://aistack.imec-int.com/blog/gpu-self-hosting
title: "How many devs can you fit on a GPU? — self-hosting explained"
author: Steven L., Wouter V.d.B., Michaël M., Ioana F., Jan F., Christian M., Maxim C., Arathy U., Bohdan D., Sepideh P., Baptist V., Jeroen B., Robbert V.C., Mathias F., Hendrik M., Jannick V. (aistack / imec)
date_fetched: 2026-08-01
date_published: 2026-07-23
topics:
  - local-and-open-source-inference
---

The aistack (imec) team benchmarks self-hosting open-weight LLMs for coding
agents across four hardware tiers — DGX Spark, single H200, 4×H200, and 8×B200
— pairing each with a best-fit model. They measure token throughput, task
completion time, and resolution rate at increasing concurrency on SWEBench Pro
tasks, then compare costs against frontier API pricing.

Key findings: Qwen3.6-35B-A3B on a single H200 comfortably serves 32 concurrent
developer sessions. DeepSeek-V4-Flash on 4×H200s matches that concurrency with
higher quality. GLM-5.2 on 8×B200s delivers near-frontier quality but saturates
at ~8 users. Kimi K3 outperforms all tested models on quality (86.4% resolution
rate) but is 8× slower than the Claude Code API baseline and requires B300s to
fit in memory.

On cost: renting GPUs can be up to 35× cheaper than APIs for smaller models,
but more expensive for larger ones. A 4×H200 must stay 89% busy to beat
DeepSeek API pricing; an 8×B200 rack needs only 15% utilisation. Real-world GPU
utilisation in enterprise inference typically runs 15–25%, so owned hardware
often idles. The authors recommend measuring your own workload peaks and
concurrency patterns before committing — and note that overnight automated-agent
sessions are emerging as a way to soak up idle capacity.

An update (29 July 2026) adds Kimi K3 results: 1.4TB weights, requires 8×B300
node (~20% pricier than B200), 30% lower throughput than GLM-5.2 but markedly
better task resolution.
