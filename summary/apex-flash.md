---
url: https://www.cantina.security/apex-flash
title: "Introducing Apex Flash-1"
author: Cantina Security (with Yeta Labs)
date_fetched: 2026-10-05
date_published: 2026-10-05
topics:
  - ai-research-and-models
  - security-and-sandboxing
---

Cantina Security, with Yeta Labs, released apex-flash-1, an open-weights security-research model fine-tuned from GLM-5.3-Flash using GRPO reinforcement learning on 150 tasks built from 50 real vulnerability cases. It scores 66.7% Pass@1 on their 60-task held-out eval versus 60.0% for the base model and 71.7% for Claude Opus 5 High, at $2.38 per run against Opus's $74.68 — a Pareto-frontier move on cost, not raw capability.

The release doubles as a policy argument: cybersecurity is adversarial, so restricting defender access to capable models restricts defenders, not attackers. Cantina draws the analogy to the 1990s disclosure debate that normalized Nmap and Metasploit, and argues defenders need models they can run, control, and improve inside their own systems. They also ship an "abliterated" variant for researchers who want more behavioral control.

The training detail is the interesting part. Environments are functioning open-source applications (e.g. a Forgejo workflow-artifact authorization bug) with realistic seeded data, three information tiers (guided whitebox, focused whitebox, blackbox), and binary verification that the agent reached the objective *through the intended bug*. Because GRPO needs contrast within rollout groups, difficulty is calibrated so tasks are solvable some of the time; task wording alone can swing solve rates. Full-parameter RL on all MoE experts collapsed the model — they suspect misattributed gradients on a small dataset — so they used rank-256 LoRA across experts plus full updates to 16 activation-selected experts.

Apex-flash-1 is explicitly positioned as a *worker* model to be orchestrated by a larger planner: focused, tenacious investigation (reading code, using tools, verifying findings) rather than autonomous breadth.
