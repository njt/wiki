---
url: https://www.dgt.is/blog/2026-10-07-deepseek-freek-out/
title: "Why Isn't The Industry Freaking Out About DeepSeek 4.1 Flash?"
author: Nat Torkington (dgt)
date_fetched: 2026-10-09
date_published: 2026-10-07
topics:
  - local-and-open-source-inference
  - ai-product-and-business
---

# Why Isn't The Industry Freaking Out About DeepSeek 4.1 Flash?

A month of heavy use of DeepSeek 4.1 Flash across a dozen projects has convinced Nat that it behaves like a frontier model — he can't tell it from Opus mid-session — while costing orders of magnitude less. He thinks the frontier labs should be panicking because Chinese distilled models are "a month or two behind" and can handle the same workloads.

## Key points

- **Subjective indistinguishability.** In daily coding sessions he cannot tell whether he's talking to DeepSeek or Opus; he treats it "like a frontier model because it behaves like one."
- **Cheapness changes behaviour.** On a $10/month OpenCode Go subscription DeepSeek is effectively unlimited: exploratory UI monkey-testing, mindless tasks, all-day sessions "rarely exceed $1 in expected costs." Reorganising a desktop "will cost $0.003 instead of $1."
- **The tiered pattern.** DeepSeek 4.1 Flash does planning, research, and execution; he occasionally pulls in Opus 5.5 for a final code review to catch edge cases, then has DeepSeek execute the fixes. The point of calling in Opus or GLM is "getting new eyes on a problem," not superior capability.
- **Cache magic.** DeepSeek shrank the KV cache ~437× versus their V1 model. Since holding KV cache in GPU memory is one of the biggest costs of long coding sessions, that's how all-day sessions stay under a dollar — and it must also mean less water and electricity. The same optimisation is how Opus 5.5 "quietly got its own efficiency boost."
- **"Good enough" reframes the market.** Chasing the latest and greatest is silly when models are good enough for high-quality unattended tasks; industry status-spending (FAANG buying the most expensive intelligence, people load-balancing a dozen Claude Max subs) misses the real story.
- **Self-hosting implications.** At these economics, self-hosting is not worth it for cost reasons — you'll never recoup hardware. Only privacy justifies it, and the cache optimisations are "coming to you" locally: "Even now, 4.1 Flash is technically self-hostable, even if not practically so. Any day now."

## Take

A practitioner's ground-truth report that cheap Chinese distilled models have crossed the "can't tell the difference" threshold for real coding work — and that cost, not capability, is now the thing that changes how you work.
