---
url: https://thenewstack.io/building-ai-agent-harness/
title: "Your AI agent is only as good as the harness around it"
author: The New Stack (author not stated)
date_fetched: 2026-09-16
topics:
  - agent-architecture
  - guardrails-and-feedback-loops
---

A production-agent essay arguing that the model is only one component of an agent service; the harness — the scaffolding that feeds inputs, checks outputs, and contains failures — is the rest. Demos succeed because everything behaves; production fails at the seams between the model and the systems around it, which is precisely where the harness lives.

The article walks through five harness concerns. Tools need contracts with schemas, timeouts, idempotency keys, and a retryable/terminal error taxonomy, because "an error message is a prompt" the model will act on. Permissions are product decisions, not hygiene: prompt injection means you cannot trust the model to resist embedded instructions, so scoped credentials and verified identity tokens (never model-filled parameters) are the real security boundary. Context needs a defined build order and provenance records. Traces — following OpenTelemetry's genAI conventions — make failures investigable. And testing should be built from real user failures, run repeatedly with exact assertions on the invariants and rubrics for the prose.

It closes on designed stop conditions: refusal, escalation, and approval paths are product features, not apologies. The piece is framed generically but is effectively sponsored content — the context section pivots to Oracle AI Database's integrated vector search and its LangChain/LangGraph packages as the answer to permission-slipping across retrieval boundaries.
