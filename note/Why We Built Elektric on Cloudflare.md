# Why We Built Elektric on Cloudflare

Elektric, an AI infrastructure/routing layer built as a single Cloudflare Worker, explains its platform choice: the whole request lifecycle (auth, classification, routing, provider transport, fallback, streaming, billing, telemetry) lives in one runtime on Cloudflare primitives, with the routing intelligence kept firmly in-house — "Cloudflare transports the request. Elektric decides where it should go."

---

## What it argues

- One Worker (`elektric-api`) hosts the entire AI request path instead of a fleet of microservices; the platform's value is that its primitives compose around a single application runtime.
- Models are used *inside* the infrastructure: a Qwen classifier on Workers AI tags requests by category and difficulty and checks Knowledge relevance — the classifier and the answering model are two different layers.
- AI Gateway is transport, not routing. Candidate pools, eligibility rules, health checks and fallback decisions are Elektric's.
- Storage is split by shape: D1 for queryable structured state (with telemetry separating a request from its multiple provider attempts), R2 for objects, AI Search for Knowledge retrieval; Conversation/Memory/Knowledge are deliberately separate context systems.
- Durable Objects hold the state that shouldn't be stateless: per-project limits, provider-capacity coordination, realtime sessions.
- Streaming limits fallback: a model can only be swapped before the response stream is committed — a "small implementation detail" that illustrates why AI infrastructure is more than a provider proxy.
- Economics is a first-class architectural reason: free tiers, $5/mo paid plan, no Workers or R2 egress charges — "it is cheap to try things."
- Model Lab evaluates candidate models through the same adapters, producing evidence that never automatically changes production routing.

---

## Key quotes

> **"Cloudflare transports the request. Elektric decides where it should go."**

The sharpest line in the piece, and the thesis. It draws a clean line between commodity transport and proprietary judgment — the same boundary every "AI platform" vendor blurs when their gateway starts offering routing.

> "AI itself becomes an infrastructure primitive inside Elektric, not merely the destination at the end of the request."

A nice inversion: the internal classifiers consume inference to decide where the *real* inference goes. Cheap small-model classification as a routing input is the pattern [[Model Routing Is Simple Until It Isn't]] describes from the outside.

> "We did not want an AI platform that made every architectural decision for us. We wanted infrastructure that gave us strong primitives while leaving us free to build the intelligence layer ourselves."

The strategic core: platform lock-in fear inverted into a selection criterion. Elektric chose the vendor that least wants to own their brain.

> "It is cheap to try things."

When you're still discovering what the right architecture is, pricing shape (scale-to-zero, no egress) is itself an architectural property — it determines how much of the design space you can afford to explore.

---

## Themes

#concept #tool #pattern

- **Coordination as the product** — Elektric's self-description ("Elektric's job is ultimately one of coordination") is a useful definition of the AI-infrastructure layer: not the model, not the app, but the translation and policy between them.
- **Evidence vs. policy** — Model Lab's separation of evaluation from production routing is good discipline; benchmarks inform but never auto-deploy.
- **Single-runtime composition** — an implicit argument against microservice sprawl: the request path is one unit of deployment.

---

## Opinion

This is a vendor-adjacent architecture post (Elektric runs on Cloudflare and the pricing section reads like a soft ad), so some scepticism is warranted — there are no failure stories, no latency numbers, no costs shown in anger. But the load-bearing distinction survives scrutiny: they use Cloudflare the way you'd use AWS primitives, and the routing/evaluation intelligence stays proprietary. That's exactly the boundary [[Model Routing Is Simple Until It Isn't]] argues is the whole game. The telemetry design — recording requests and provider attempts separately so fallback chains are queryable — is the most quietly impressive detail and the least marketing-flavoured part of the piece.

It also nuances the "AI gateway" category: AI Gateway appears purely as plumbing, which complicates any story where gateways *are* routing products.

## Related

- [[Model Routing Is Simple Until It Isn't]] — this source is a concrete instantiation of that page's thesis: routing policy is the hard part, and here a vendor's gateway is deliberately kept out of it.
- [[Cloudflare OS]] — strengthens the case that Cloudflare is assembling a coherent primitives stack (Workers, D1, R2, DO, Workflows, AI Gateway) rather than a grab-bag of services; Elektric is a working demonstration of composing them.
- [[Managing AI Coding Costs at Scale]] — Databricks treats the AI Gateway and intelligent routing as cost infrastructure; Elektric complicates that by owning routing above the gateway and treating pricing shape as an experimentation enabler instead.

---
*Sources: [[raw/why-we-built-elektric-on-cloudflare]], [[summary/why-we-built-elektric-on-cloudflare]]*
*Last updated: 2026-10-24*
