---
url: https://awflow.io/blog/ai-agent-teams-that-dont-loop
title: "AI Agent Teams That Don't Loop"
author: AWFlow team
date_fetched: 2026-10-24
date_published: unknown
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

AWFlow (a browser-based automation tool) shipped multi-agent "teams" and wrote up the coordination runtime that made them usable. The diagnosis: free-for-all agent chatrooms — the default of multi-agent demos — produce loops, duplicate work, and runaway token bills in production.

Their answer is structure enforced in code, not prose. A team thread snapshots its members, roles, handoff map and rules at start, so a running conversation can't have its rules changed underneath it. Teams pick one of three ways of working: lead routes (a manager plans, delegates via a per-agent `delegate_to` tool, and merges results), pipeline (fixed order, no planning, friendliest to small local models), and open table (human steers via @mentions). The handoff map is compile-time reality: an agent with no permitted targets doesn't even see the `delegate_to` tool — cheaper and more reliable than asking the model in a prompt to only delegate to X.

Handoffs carry the task and needed context, not the transcript — the difference between working and not working on 2–4k-token local models. Every thread has budgets (steps, handoffs, cloud tokens) checked *before* each handoff runs, and team rules merge so a permissive agent joining a careful team only ever gets stricter. All of it is enforced in code around tool calls, "so a web page can't talk an agent out of it." Out-of-team `ask_teammate` is capped at one level deep, so asks can't recurse. The runtime is pure orchestration with `runMember` injected, so the whole thing unit-tests against fake agents without calling a model.
