# OTel-Native by Design

The OpenTelemetry project's design guide for products that export telemetry to *any* observability backend: ship OTel instrumentation (self-hosted) or a configurable drain feature (platform), push over OTLP instead of making users poll a custom API, and treat Semantic Conventions as the contract that keeps exported data interpretable.

---

The post starts from a customer reality: users of your software or SaaS will eventually demand their logs, traces, and metrics in their own stack — for compliance, cost, or consolidation. Locking them into your dashboards or per-vendor exports is friction. The answer is to be "OTel-native by design": configure an OTLP endpoint, push, done.

## The design matrix

Two deployment contexts, two approaches:

- **Self-hosted software** (Kuma, Keycloak): instrument the product itself, expose the OTLP endpoint via policies or startup flags. Kuma does it with per-signal mesh policies (`MeshAccessLog`, `MeshTrace`, `MeshMetric`); Keycloak with a single `--telemetry-endpoint` flag and per-signal toggles (logs still in preview).
- **Platforms** (Cloudflare Workers, Heroku): your infrastructure collects from the user's workload and forwards to their destination. Cloudflare's Observability Destinations auto-pushes traces and logs; Heroku's Telemetry Drains let users pick signals, protocol, and headers.

Four properties of a good export story: vendor-neutral, no deep custom integration, rich context preserved (trace IDs on logs), Semantic Conventions compliance.

## Push vs poll

The central architectural argument: **custom polling APIs should not be the default for new designs**. Polling makes users build scalable pollers — pagination, retry, backfill, state tracking — and makes near-real-time delivery hard, which matters for traces and metrics. OTLP push is one protocol for all three signals, preserves structure and correlation natively, and is proven at scale (the project recently graduated from CNCF).

## Practical machinery

- Single endpoint for most users; per-signal endpoints (`OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`) for advanced ones — "technically optional, but a major value-add."
- Per-signal enable/disable toggles for cost control (Heroku's `--signals`).
- Collector topology: one per tenant (strong isolation, expensive at hundreds of instances) vs shared Collector with static per-tenant pipelines and a routing connector (cheap to operate, config-file sprawl).
- Weaver for keeping your own attribute schemas in lockstep with Semantic Conventions.

The closing "golden rules": **default to push, stay vendor-neutral, and document the contract.** And a direct appeal to end users: ask your vendors for OTLP export — this post is the thing to point at.

---

## Key quotes

> "Locking them into your built-in dashboards or limiting exports to certain vendors creates unnecessary friction."

The framing is deliberately anti-lock-in from the vendor's own mouth. Notably, this comes from the OTel project with New Relic contributing — a backend vendor arguing users should be able to leave. That's standards-body self-interest in the good sense: the ecosystem benefits when exit is cheap.

> "You shift a significant operational responsibility onto your users. They must now build scalable polling systems that can manage polling intervals, paginate API responses, retry on failures, and backfill missing data."

The strongest passage. The polling model doesn't just cost users latency; it transfers an entire unbounded engineering project — a mini data pipeline — onto every customer. The CloudWatch-style pseudoloop in the article makes the hidden complexity legible: `state: last_end_time` is the tip of a very cold iceberg.

> "For new designs, however, it should not be the default."

Honest hedge. The authors concede polling is "a reasonable starting point if you already have a mature, well-tested API" — the migration-cost argument in disguise. Most real products have exactly such an API, which is why the industry is still full of pull-based telemetry despite everyone agreeing push is better.

> "Default to push, stay vendor-neutral, and document the contract."

Twelve words that compress the whole post. "Document the contract" is the underrated clause: OTLP gets the bytes there, but without documented attribute semantics and signal-to-trace correlation the data arrives as noise.

## Analysis

This is a standards-body design essay, and it argues like one: the strawman (per-vendor integrations) is real but the harder competitor — your existing pull API — is handled with a concession rather than a kill. The honest question the post under-argues is **who pays**: OTLP push moves cost from the user's poller to the vendor's Collector fleet, and the multi-tenancy section quietly admits the shared-Collector path means the vendor now operates a telemetry routing layer per customer base. That's a genuine operational commitment, not free vendor-neutrality.

The most transferable idea isn't about observability at all — it's the pattern of **expose a standard protocol endpoint and let users pull the data out**, which is the same shape as MCP for tools and OTLP for telemetry: interop as a design axiom rather than a feature. For agent infrastructure it maps directly onto how harnesses should emit their own telemetry (see [[Your Coding Agent Needs Observability Too]]).

It also pairs neatly with [[AI Handles Incidents, Engineers Lose Touch with Their Systems]]: portable telemetry is what makes "your stack" a choice rather than a habit, and it's the substrate on which any incident-analysis agent must read. And the Collector-per-tenant vs shared-Collector tradeoff is a textbook isolation-vs-economics scale decision, sibling to arguments in [[Durable Actors — Open-Source Durable Objects]] about tenancy isolation as the primitive.

Weaknesses: the comparison table is thin (four examples, all volunteers), metrics support is patchy even among the exemplars (Cloudflare lacks it), and profiles — the fourth OTel signal — is waved through. The post is a checklist in search of a counterexample.

**Tags:** #concept #tool #pattern

---
*Sources: [[raw/otel-native-by-design]], [[summary/otel-native-by-design]]*
*Last updated: 2026-10-11*
