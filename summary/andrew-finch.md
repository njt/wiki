---
url: https://ai.engineer/code/2026/talks/andrew-finch
title: "The Safe Software Factory"
author: Andrew Finch
date_fetched: 2026-11-26
date_published: 2026
topics:
  - guardrails-and-feedback-loops
  - ai-code-review
---

Andrew Finch, a staff engineer at Sonar working on Guitar (Sonar's AI code review product), argues that agentic coding has automated the old bottleneck — writing code — and shifted it to review and verification. The question is no longer how to build the software factory but how to make it safe: "Code is abundant, confidence is not."

His answer is defense in depth through four loops. **Loop 1, the PR checks loop**, is the familiar PR → CI → review cycle, but he argues it can be widened well past tests and code review: static analysis, functional validation against the ticket, coding-convention rules, accumulated knowledge from past reviews and outages, cross-repo and cross-PR analysis, security review, end-to-end browser testing, and risk assessment that grades blast radius and decides whether a human needs to look at all. Merge gates should decide on recorded evidence, not on "a test ran," which lets teams graduate to auto-approve and automerge for low-risk changes and free human reviewers for the risky ones.

**Loop 2, post-deploy verification**, is early-stage: agents reading production logs, traces and metrics, agentic rollout control (widen/rollback/feature flags), and AI-assisted outage triage. **Loop 3, the maintenance loop**, runs in parallel on a schedule over the whole codebase, catching things a single PR cannot: rules or vulnerabilities published after merge, and slow drift like accumulating duplication or complexity — opening remediation PRs for what it finds. **Loop 4, the knowledge loop**, curates company documentation, product decisions, and lessons from past reviews and outages, then feeds it to every other loop's agents so coding agents get things right the first time and need fewer review laps.

Humans stay in the loop, but relocated: the factory "runs dim, not dark." People audit risky code, verify evidence, handle exceptions like rollbacks and incidents, review architecture (where "agents are not very good at this yet"), supply the why — product intent, trade-offs, business decisions — and, above all, engineer the factory itself.
