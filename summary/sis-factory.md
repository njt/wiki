---
url: https://github.com/sis-factory/sis-factory
title: "SIS: the 3D Self-Improving Software Factory"
author: sis-factory (GitHub organisation, MIT license)
date_fetched: 2026-11-09
date_published: 2026 (active development)
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

SIS is an open-source (MIT) "self-improving software factory": label a Jira ticket `sis` and a coding agent picks it up, works in a locked-down container, and opens a draft pull request. The pitch is enterprise-grade and fully local — no cloud, no external API keys, with a stated deadline of 1 November 2027 for a 100% locally runnable stack. A live 3D factory floor (React Three Fiber) renders every task as a robot you can click to read logs, answer its questions or stop it.

The architecture is a fleet of ~17 small Python components (FastAPI services plus Temporal workers) wired through a shared pure-Rules library: jira-connector (sole holder of Jira credentials) → orchestrator-worker (a Temporal TaskWorkflow per ticket, with planner/intent/architect/developer agent roles) → injection-guard (scans everything going in and out) → sandboxed agent-runner → sandbox-gateway (the sandbox's only network egress, holding model keys) → git-broker (GitHub App with short-lived installation tokens) → portal (Next.js dashboard + 3D view) over PostgreSQL. Everything durable runs on Temporal; Podman containers, gVisor-ready, Helm charts included.

The distinctive piece is the lessons engine. Review feedback, CI failures and human fix-up commits are mined into candidate "lessons" (`@sis remember …`), which humans approve by role; machine-proposed lessons are withheld from a deterministic random share of tasks (a sha256 hash of task+lesson), and their effect on merge rate is measured A/B against the held-out group with a one-sided z-test — lessons that hurt are retired automatically. Security posture is unusually thorough: untrusted text wrapped in tagged blocks, canary tokens in system prompts to detect leaks, outbound secret-pattern scanning, PII redaction before indexing, and a scripted no-model demo agent so the stack runs dry by default.
