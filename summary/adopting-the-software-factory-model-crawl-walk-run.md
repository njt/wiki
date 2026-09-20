---
url: https://www.warp.dev/blog/adopting-the-software-factory-model-crawl-walk-run
title: "Adopting the software factory model: crawl, walk, run"
date_fetched: 2026-09-20
topics:
  - agent-coding-workflow
  - agent-orchestration
---

Warp lays out a three-stage adoption path for moving engineering organisations from local, interactive coding agents to cloud-hosted "software factories" — closed agentic loops that run autonomously in the cloud. The **crawl** stage is point automations on triggers: agentic issue triage, PR review, self-healing CI, doc updates, browser-based QA verification. These are low-risk but hit predictable limits: no shared context between automations, no global productivity metrics, and per-tool maintenance and security burden.

The **walk** stage is deciding on a factory architecture and deploying one end-to-end on a deliberately simple surface — Warp automated ~75% of changes to its own marketing site this way. The loop is triage → spec → implement → review → verify → monitor, with humans at decision points. Warp argues this is not a build-vs-buy decision: every team will build org-specific skills, MCPs and context integrations, and the real question is which commodity layers (agent infrastructure, steering, measurement) to partner on.

The **run** stage scales the factory to complex codebases, where bottlenecks emerge: remote dev environments, config churn, model routing costs, security, and PR backlogs. The end state is a closed-loop, measurable, self-improving system — "software engineering is becoming factory engineering." The post is also, transparently, a pitch for Warp Factories (early access, $10k usage credit).
