---
url: https://web.archive.org/web/20260925153131/https://momo5502.com/posts/2026-08-22-mw2-decompilation-audit/
title: "Evaluating My Multi-Agent MW2 Decompilation Setup"
author: momo5502
date_fetched: 2026-10-11
date_published: 2026-08-22
topics:
  - agent-orchestration
  - agent-memory-and-context
---

momo5502 follows up on a multi-agent MW2 decompilation project by auditing ~2GB of session logs to find where the tokens and hours actually go. Pay-per-token-equivalent spend would have been $85,207 ($70k workers, $15k overseer); real cost ~€200 on a subscription, with 89% cache reads since agents run nonstop.

The biggest finding is waste that isn't intelligence: CI polling, sleep/wait, and subagent duplication. 47 subagent runs re-read 54.7% of the data they ingested from large files — `STATUS.md` alone was read 28 times by 21 different subagents, usually within 35 minutes of another subagent's identical read. Disabling subagents raised commits/day from 244 to 311.

Context management is the other lever: sessions ran to 93–100% of the 1M-token window, meaning every turn paid near-full cache-read cost on mostly volatile data (decompiled functions nobody needs again). Compacting at 60% instead should save tokens at no performance cost. Model comparison shows Sonnet 5 out-commits Opus 5 (6.1 vs 1.6 commits/hour) with fewer build failures, and — the sharpest result — the overseer's model matters more than the worker's: the reviewer catches the bugs, so a Sonnet reviewer over an Opus worker beats the reverse. The overseer flagged 354 distinct bugs across 2,177 commit-referencing messages, justifying its existence.

Takeaways: cheap fast workers + a good reviewer beat expensive workers; compact early; mechanical blockers beat soft instructions; polling is a bug, webhooks are the fix.
