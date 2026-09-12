---
url: https://coding-is-like-cooking.info/2026/06/i-stopped-coding-and-started-architecting-agents-and-why-you-should-too/
title: "I Stopped Coding and Started Architecting Agents (And Why You Should Too)"
author: Emily Bache
date_fetched: 2026-07-18
date_published: 2026-06-25
topics:
  - agent-coding-workflow
---

Emily Bache argues that the same code-quality skills she has always taught — working in small steps with frequent feedback — apply directly to working with agentic AI, under the banner of **Harness Engineering**.

A harness constrains an AI agent toward good code through two mechanisms: **Guides** (feed-forward advice given before the agent acts) and **Sensors** (feedback scripts that check output afterward, e.g. linting for file-length smells). Together they create a flywheel: better harness → better agent output → better codebase → better guide examples → better harness.

She recommends teams build their own harnesses rather than adopt someone else's, so they understand and can evolve them. Harnesses should also be pruned over time as models and codebases improve — an overgrown harness can be worse than none. She treats the harness as a first-class part of the codebase, updated with every task.

Bache has turned off AI line-completion ("a huge distraction") but sees agentic AI — which iterates with tools toward compiling results — as a genuine lever, particularly for legacy codebases where steady design improvement avoids risky rewrites.
