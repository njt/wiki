---
url: https://addyo.substack.com/p/agentic-verification
title: "Agentic Verification"
author: Addy Osmani
date_fetched: 2026-10-18
date_published: not stated (2026)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Addy Osmani's essay argues that verification, not code generation, is the bottleneck in agentic software development: today's models are good enough that what limits quality is the ability to check their own work. Because agents make writing tests, benchmarks and specs cheap, the human's job shifts from reviewing every line to building loops the agent can run itself — and to "make taste compound" by writing every agent mistake down as shared config, a hook, or a test.

He surveys a spectrum of oracles. E2E tests are the one to pick if you can pick only one, because they are hard to trick — either the whole flow works or it doesn't — and they define "done" as a target the agent can chase alone. Property-based tests (fast-check, Hypothesis, proptest) counter the agent's habit of inventing its own examples: a rule that must always hold, hammered by thousands of generated inputs. Differential testing makes the old system the oracle during rewrites and migrations, closing the gap between official specification and actual behaviour.

A final section insists the loop must be fast and deterministic, or the agent learns the wrong lesson — that retrying a flaky test beats fixing the code. Good loop hygiene reads like a checklist: subsecond core suites, seeded randomness, flaky tests removed immediately, short specific failure output. The essay closes with embedded comment-style replies (Lauren on giving agents hands and eyes on the running system, Boris Cherny on Lean/TLA+ for state machines, a commenter on who writes the E2E test) that add practical nuance.
