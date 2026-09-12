---
url: https://nikola-petkovic.com/blog/2026/08/07/observability-pain-cycle/
title: The Observability Pain Cycle
author: Nikola Petkovic
site: nikola-petkovic.com
date_fetched: 2026-08-25
date_published: 2026-08-07
topics:
  - software-engineering-craft
---

Nikola Petkovic names a pattern he argues is endemic to mainstream observability platforms: the **Observability Pain Cycle**. Vendors push a "store-everything-up-front" model — ingest all telemetry and charge from minute one, regardless of how much of it ever yields value. A large share of what's stored is noise (an average vanilla Kubernetes cluster emits almost 100,000 time series out of the box), yet nothing tracks which series actually feed a dashboard or alert, so the only signal that triggers cleanup is the invoice. When the bill crosses the pain threshold, an engineer hunts for telemetry to trim, applies coarse filters (drop log levels, sample traces, exclude prefixes), and a few months later the volume silently regrows and the cycle repeats.

The article traces how the model became the default — reasonable when telemetry was low-volume and hand-instrumented, never revisited after cloud, containers, microservices, and auto-instrumentation made emission nearly free — and then dissects five "tactics to avoid the change": "you never know which signal the next incident will need" (true only for a thin slice), "storage is cheap" (ignoring ingestion, indexing, query, and egress costs), AI features (which sit downstream of collection and are themselves hurt by noise), classic cost controls (which decide *how much* to keep, not *which* data matters), and "transparent billing" (which itemizes accurately but never answers which signals are worth paying for).

The one exception Petkovic grants is tail-based trace sampling, because traces carry a value marker (latency and error status) the pipeline can read. Metrics and logs have no such marker — their value is whether anything downstream uses them, which pipelines never inspect. The coda argues the problem is worsening in the AI era, as agent and AI workloads emit telemetry an order of magnitude faster than human teams ever did — and that it won't be fixed by those who profit from the noise.

---
*Source: [[raw/observability-pain-cycle]]*
