---
url: https://arcprize.org/results/deepseek-v4-flash-0731
title: DeepSeek V4 Flash 0731
author: DeepSeek / ARC Prize
date_published: 2026-07-31
date_fetched: 2026-08-08
---

# DeepSeek V4 Flash 0731 — ARC-AGI Results

DeepSeek V4 Flash 0731 is a reasoning model with three effort variants (Max, High, Low), submitted to the ARC Prize leaderboard. At max effort, it scores **89.0% on ARC-AGI-1** (Semi-Private) at $0.02 per task and **61.4% on ARC-AGI-2** (Semi-Private) at $0.04 per task. No ARC-AGI-3 scores are reported.

## Verified Scores

| Variant | ARC-AGI-1 | ARC-AGI-2 | ARC-AGI-3 |
|---------|-----------|-----------|-----------|
| Max     | 89.0%     | 61.4%     | —         |
| High    | 87.0%     | 56.0%     | —         |
| Low     | 84.0%     | 46.0%     | —         |

The gap between effort levels is modest on ARC-AGI-1 (5pp from Low to Max) but substantial on ARC-AGI-2 (15.4pp), suggesting the harder benchmark is where reasoning depth matters most. The cost is extremely low by frontier standards — $0.02–0.04 per task makes this one of the most cost-efficient high-scoring entries on the leaderboard.

The public eval tables show per-task pass/fail across all three variants for 400 ARC-AGI-1 tasks and 120 ARC-AGI-2 tasks. The pattern is consistent: Max passes more tasks than High, which passes more than Low, with no task ever passing at a lower effort level but failing at a higher one (the reasoning-effort monotonicity holds).
