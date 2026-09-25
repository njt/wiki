---
url: https://github.com/Parcha-ai/agentrun
title: "agent.run() — AgentRun"
author: Parcha Labs, Inc.
date_fetched: 2026-09-25
date_published: 2026
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

AgentRun (Parcha Labs, v0.1.0-beta.4, Apache-2.0) is a workflow language for agents you already run: repeatable steps defined as inspectable documents, Jev typed decisions for focused judgments, and an agent call only when investigation is needed. Your application keeps its tools, model access, permissions, and budgets — the workflow engine is embedded, not a platform.

Three npm packages: `@parcha/agentrun-dsl` (define, validate, inspect, execute), `@parcha/agentrun-jev` (the TypeSafe Jev adapter), and `@parcha/agentrun-pi` (a Pi editor extension with a packaged skill so a coding agent can author workflows). Workflows are JSON documents with 17 node kinds, or authored via a TypeScript builder with Zod contracts; the interpreter validates intermediate state paths at runtime. A support-workflow example shows the shape: search for an answer, a `judge` node checks it with a confidence threshold of 0.8, one agent attempt is allowed on failure, and the workflow escalates for human review if the check fails again.

The engine is deliberately host-owned. Tools arrive through a `runEffect` adapter, the agent through `runNode`, decisions through `runJudge` — nothing in a workflow document can name or reach host policy. Escalation is a first-class terminal state (`status: "escalated"`), checkpoints and idempotency-keyed effect memos support recovery, and a Lean model in `spec/lean` proves properties about validation and execution semantics against a shared conformance corpus with the TypeScript interpreter.

Jev decision nodes compile flat schemas (string enums = choice, boolean = yes/no, integer with level criteria = score) into typed question sets answered in one request with per-answer confidences — the same System One interface covered by [[System One Models and Jev]].
