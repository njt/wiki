---
url: https://martinalderson.com/posts/making-agents-fast/
title: "Making agents fast - what I've learnt over the last 12 months"
author: Martin Alderson
date_fetched: 2026-11-09
date_published: 2026-11
topics:
  - agent-architecture
  - agent-coding-workflow
---

Martin Alderson, a performance engineer by background, applies web/API optimisation instincts to agentic workflows. His core claim: inference speed matters, but it is the *obvious* lever and often not the biggest one — halving inference time only takes ~20% off total task time once provisioning, tools, and suspend/resume are counted.

On inference, he splits the problem into TTFT (time to first token, analogous to TTFB, includes prefill and benefits hugely from prompt caching) versus tok/s (analogous to bandwidth). Which one matters depends on the workload shape: short back-and-forth turns favour TTFT; long code or document output favours tok/s. Provider choice matters enormously even for the same model, but providers vary wildly with time of day and load — a serious "noisy neighbour" problem that is much more opaque than VM `steal`. His single biggest overlooked lever is *thinking budgets*: a fast tier at max thinking is vastly slower than a slow tier at low thinking.

The rest is classic performance engineering applied to agents: tool call traces (including `strace`, with the caveat that it has its own overhead), tool output size (uncached tokens hit TTFT; SQL-style data sources let agents pull exactly what they need), and per-command shell startup cost when agents spawn a fresh shell each time (nvm in `.bashrc` becomes a real tax). His most distinctive idea: read the model's thinking traces (easy with open weights) to discover what tools the agent *wants* but is working around — then build those tools, e.g. replace repeated `xargs` over markdown with a database. Polling loops with `sleep` are a red flag for missing async completion; parallelisation helps but weak models botch it, so build parallelism into the tools. Finally: faster hardware (real CPU cores, local NVMe vs networked disks, sandbox copy-on-write overhead), careful suspend/resume memory management, and basic network hygiene like broken IPv6/AAAA lookups.
