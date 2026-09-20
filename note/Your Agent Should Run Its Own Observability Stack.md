# Your Agent Should Run Its Own Observability Stack

Kin Lane's piece argues that agents generating real work should run their own observability stack — Prometheus, OpenTelemetry, Tempo inside the agent's own boundary — and that the reason is not the one usually given. It is a short post with two distinct payloads: a governance argument about agent telemetry, and an empirical discovery about API design for agents that undercuts his own catalog's scoring.

---

## The argument in one paragraph

Agent telemetry is not application performance data; it is a record of the agent's reasoning — what it considered, rejected, and decided — and shipping it to an observability vendor is therefore a governance decision that teams are making by default rather than deliberately. The claim that could be wrong: that self-hosted observability is now *easier for agents to use* than commercial platforms, because a single query endpoint plus an expression language beats a large REST surface, and because self-hosted systems expose their schema for machine discovery. If Lane is right, the observability industry's agent-readiness is worst exactly where its API surface is richest, and the default "buy monitoring" reflex is actively harmful for agentic workloads — not merely expensive.

## Key quotes

> Trace spans from an agent are a record of what it considered, what it rejected, and what it was working with when it decided. That is not application performance data. It is closer to a transcript.

This is the load-bearing reframe. Everything else in the piece — the self-hosting recommendation, the boundary argument — only follows if you accept that agent traces have a different sensitivity class than web-server traces. He is right that most teams haven't noticed the category change.

> An agent that learns PromQL can ask anything. An agent facing 290 REST endpoints has to first solve the problem of finding the right one, which is a retrieval problem layered on top of the actual question.

The most falsifiable and most interesting claim in the piece. It inverts the usual "more endpoints = more capable" assumption and suggests that the best agent-facing API is a small surface plus a language — which is, uncomfortably for tool builders, an argument against the sprawling REST conventions the API industry has spent a decade standardising.

> An agent can discover the entire schema before it writes a query.

The mechanism behind the previous claim. Prometheus's `/api/v1/metadata`, `/api/v1/labels`, and `/api/v1/status/tsdb` endpoints let an agent ground its queries in what actually exists — the same move retrieval-grounded agents make over documents, applied to infrastructure. Most commercial APIs offer nothing equivalent; you read the docs or you guess.

> That is a self-hosted system being *more* agent-legible than the commercial alternative, and it is legible because the project was designed for operators who introspect rather than for a UI.

A genuinely non-obvious causal claim: agent-legibility here is an *accident* of serving human operators who curl endpoints. That suggests a research direction — design APIs for introspectability and agents benefit for free — and an uncomfortable corollary: APIs designed primarily for dashboards and sales demos may be structurally hostile to agents.

> Nobody is publishing a license-aware, deployment-aware index of the self-hostable stack.

The confession-turned-roadmap. Lane admits his own catalog scores self-hosted projects badly because the rubric measures hosted-SaaS virtues (pricing pages, onboarding flows), and that nothing tells an agent which of four functionally similar projects it is *permitted* to run. Naming the gap is half the contribution.

## Critical analysis

The strongest move is refusing the cost argument. The observability-cost literature (see [[Observability Cost Saving Strategies]] and [[Wide Events vs. Three Pillars — AI Observability Costs]]) treats vendor pricing as the problem to solve; Lane says cost is "the weak version" and relocates the issue to data governance. That is a sharper framing, and it is falsifiable in a useful way: if agent traces really are transcripts, then the per-gigabyte pricing debate is a distraction from a consent question.

The weak points are the ones he half-acknowledges. The Prometheus-vs-Datadog comparison is a single anecdote against one commercial vendor, and Datadog's 290 endpoints may include exactly the schema-discovery surfaces he praises — he scored them from a catalog record, not by calling them. The claim "fewer endpoints and a language is a better agent affordance" deserves the same empirical treatment he gave Prometheus's discovery endpoints applied to the commercial alternative. There is also an unexamined tension: he wants agents to *run* infrastructure, but his own enrichment-pipeline examples show how much implicit knowledge that requires (Loki, Tempo, and Prometheus "assume each other"). An agent that can query PromQL is not yet an agent that can stand the stack up, and the piece's practical advice — "you will spend an afternoon on it" — quietly concedes a human is still in the loop.

What is left out: retention and cardinality cost on self-hosted storage. The wide-event argument that agent telemetry balloons storage is not dissolved by self-hosting; it is relocated to your own disk, and Lane says nothing about who pays that operational cost or how it is bounded. He also doesn't address the failure mode where the agent observing itself creates a feedback loop — the observer is inside the observed system, which is exactly the situation observability discipline usually tries to avoid.

## Related

- [[Observability Cost Saving Strategies]] — complicates: Shpilt's seven strategies all assume the vendor relationship is worth optimising, whereas Lane argues the per-volume billing conflict is better exited than mitigated for agent workloads.
- [[Wide Events vs. Three Pillars — AI Observability Costs]] — nuances: both agree agent telemetry breaks conventional observability economics, but Travaglini fixes it with a data model while Lane fixes it with a deployment boundary; the two are complementary rather than competing.
- [[The Three Pillars of Observability]] — strengthens: Greptime's closing question about agents as first-class consumers of observability data is answered here with a concrete mechanism — the agent queries its own telemetry through a schema-discovery API.
- [[Open Source Agent Toolkit 2026]] — extends: Perrone's seven-layer open-source stack is exactly the catalogue of projects Lane audits, but Lane adds the dimension Perrone's guide lacks — license awareness and deployment legibility for agents choosing between them.

---
*Sources: [[raw/your-agent-should-run-its-own-observability-stack]], [[summary/your-agent-should-run-its-own-observability-stack]]*
