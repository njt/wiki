---
url: https://brooker.co.za/blog/2026/09/28/engineering-system-one.html
title: "Engineering System One: Building Hobson"
author: Marc Brooker
date_fetched: 2026-10-03
date_published: 2026-09-28
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Marc Brooker (AWS principal engineer, known for distributed-systems work) spends two weeks catching up on AI science by building his own small System One-style classifier, "Hobson" — a ~2B-parameter calibrated decision model trained at home on a single RTX 3090, benchmarked against TypeSafe's Jev on the public JevBench leaderboard.

The architecture: take a pre-trained Qwen3.5-2B torso, rip off the LM head (removing text generation entirely), and replace it with a ~1M-parameter *pointer head* that scores each offered option by comparing the hidden state at the `<answer>` position against each option's hidden state. Fine-tuned with a rank-16 LoRA over 115k rows (113k public data, 2k synthetic hard questions), plus self-distillation (KL to a frozen torso, and KL to earlier checkpoints on regressing tasks) to limit forgetting. Post-training, per-question-type temperatures are fit on held-out data for calibration.

Results: top of his size class on jevbench public, Brier 0.009 with 100% accuracy on the easy set, p50 latency just over 100ms and p95 under 300ms on a 3090 — with one or two forward passes regardless of question count. He's honest that generalization to unseen tasks is "useful but not great," that in-task accuracy moves far more easily than generalization, and that his own hold-out discipline is partly a matter of "how subconsciously intellectually honest I am. Science is hard."

Two meta-lessons: every line of code was written by an agent (Claude/Kiro), but he insisted the core ideas stay his — agents as a custom textbook with quizzing; and the "number go up" leaderboard loop is dangerously addictive ("people who had a bit of a *problem* with World of Warcraft... should probably find another way to spend their time").
