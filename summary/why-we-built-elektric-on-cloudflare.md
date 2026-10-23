---
url: https://elektric.ai/blog/why-we-built-elektric-on-cloudflare
title: "Why We Built Elektric on Cloudflare"
author: Elektric team
date_fetched: 2026-10-24
date_published: unknown
topics:
  - ai-infrastructure-and-hardware
  - ai-product-and-business
---

Elektric — an AI infrastructure layer that sits between applications and a shifting ecosystem of model providers — explains why most of its server-side logic runs on Cloudflare. A single request touches authentication, classification, routing, context, provider selection, fallback, streaming, billing and telemetry before the answer reaches the developer, and the whole orchestration lives inside one Worker (`elektric-api`) rather than a fleet of microservices.

The architecture is layered deliberately: Workers AI runs internal classifiers (request category/difficulty, Knowledge relevance) that route but never answer; Cloudflare AI Gateway is a common transport to OpenAI/Anthropic/Google, with Elektric — not the gateway — owning routing policy; D1 holds structured state (with telemetry separating requests from individual provider attempts so fallback behaviour is queryable); R2 holds objects; AI Search backs Knowledge retrieval; Durable Objects coordinate per-project limits, provider capacity and realtime sessions. Conversation, Memory and Knowledge are kept as separate context systems so the underlying model can change while application context stays stable.

The economics argument is prominent: near-zero starting cost, generous free tiers, no Workers egress charges, R2 with free Internet egress — "it is cheap to try things" while the right AI infrastructure is still being figured out. Model Lab, the evaluation system, runs candidates through the same provider adapters and produces evidence, but evaluation never automatically changes production routing. The closing distinction: Cloudflare provides primitives, not the system — "Cloudflare does not build Elektric for us"; the intelligence layer is Elektric's.
