---
url: https://opentelemetry.io/blog/2026/otel-native-by-design/
title: "OTel-Native by Design — Building Products That Export to Any Observability Stack"
author: OpenTelemetry project (with contributions from Dan Gomez Blanco, New Relic)
date_fetched: 2026-10-11
date_published: 2026 (opentelemetry.io blog)
topics:
  - guardrails-and-feedback-loops
  - distributed-systems
---

An OpenTelemetry project blog post arguing that software products — whether self-hosted binaries or managed platforms — should let users export their logs, traces, and metrics to any OTel-compatible backend rather than locking them into built-in dashboards or per-vendor integrations. Its core prescription: push-based export over OTLP as the default, not custom polling APIs.

The post frames the design question by deployment model. Self-hosted software (Kuma, Keycloak) should ship pre-instrumented with the OTel SDK and expose endpoint configuration through policies or startup flags. Platforms (Cloudflare Workers, Heroku) should offer a platform-level "drain" or "destination" feature where the operator's infrastructure forwards telemetry to the customer's OTLP endpoint. Four properties define a good export story: vendor-neutrality, no deep custom development, preserved context (trace/span correlation), and adherence to Semantic Conventions.

It then makes the architectural argument: custom polling APIs shift pagination, retry, backfill, and state-tracking burden onto users and make near-real-time delivery hard; OTLP push is one vendor-neutral protocol for all three signals and should be the default for new designs. Practical guidance follows — a single OTLP endpoint for most users with per-signal endpoints (`OTEL_EXPORTER_OTLP_<SIGNAL>_ENDPOINT`) as a power-user option, per-signal toggles to control cost, and a choice between one Collector per tenant (strong isolation, resource-hungry) and a shared Collector with static per-tenant pipelines (simpler ops, growing config file). It closes with a checklist of seven design steps and a challenge to end users: ask your vendors for OTLP export.
