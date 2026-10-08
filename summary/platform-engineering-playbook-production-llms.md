---
url: https://www.infoq.com/articles/platform-engineering-playbook-production-llms/
title: "A Platform Engineering Playbook for Production LLMs"
author: "Praveen Shipurapu (InfoQ)"
date_fetched: 2026-10-08
date_published: 2026
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

A retail-platform engineer's field report on running a multi-agent LLM inventory-recommendation system in production: hallucination rate went from 15% to 1.5% in six months, and the lever was not a better model but moving LLM concerns from each application into a shared platform layer.

The architecture is a single gateway (auth, JWT with role claims, metrics, admin prompt-rollback routes) in front of a root coordinator agent built on Google's ADK, which classifies intent and delegates to specialist agents, each bound to a registry-resolved prompt and a Pydantic structured-output schema, reaching models through LiteLLM and tools through per-team MCP servers built with FastMCP. The companion repo runs end-to-end on Ollama's llama3.1 at zero API cost, deliberately with "no hidden abstractions."

The primitives: schema enforcement at the platform boundary with typed retry-on-validation-failure; failure classification (schema violation / hallucination signal / infrastructure error) routing to different retry strategies with a capped retry budget; a history-preserving prompt registry with runtime resolution so rollback takes effect on the next request without redeploy; an intent-validation gate that returns "unclassified" instead of defaulting to the highest-scoring agent; RBAC with default-deny enforced at each MCP resource server, not just the gateway; per-user/per-team cost attribution set at request ingress because "the team dimension must exist in the data from the first call"; LLM-specific observability (hallucination rate, prompt drift, token spend by team) that standard APM cannot provide; and a golden-set evaluation suite gating prompt promotions in CI.

The unifying argument: LLM systems fail differently from traditional systems — they don't crash, they "quietly produce slightly worse output" — so the platform primitives must be built before the second application, not retrofitted after the tenth under incident pressure.
