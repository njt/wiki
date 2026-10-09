---
url: https://motherduck.com/docs/getting-started/customer-facing-analytics/
title: "Customer-Facing Analytics Overview"
author: MotherDuck (vendor documentation)
date_fetched: 2026-10-09
date_published: unknown
topics:
  - databases-and-data
  - ai-product-and-business
---

MotherDuck's overview of customer-facing analytics (CFA) — embedded analytics for external users inside a SaaS product — and how its DuckDB-based architecture is positioned to serve it. It is vendor documentation, but a well-argued one: it opens with a comparison table of traditional BI vs CFA (external audience, milliseconds-to-low-seconds latency, thousands to millions of users, per-customer isolation, JavaScript/embedded SDK stack), then names three challenges: OLTP/OLAP stack mismatch, latency requirements that distributed warehouses (BigQuery, Snowflake, Databricks) miss due to cold starts and coordination overhead, and multi-tenancy on shared clusters (noisy neighbors, overprovisioning, contention).

The two architectural answers are **Hypertenancy** — every customer (or even every customer's user) gets a dedicated DuckDB instance, a "Duckling," with vertical scaling and read-scaling Ducklings for heavy tenants, sub-100ms cold starts and per-second billing — and **Dual Execution**, where the same DuckDB engine runs in the cloud (3-tier) or in the browser via WebAssembly (1.5-tier), offloading compute to the client for sub-10ms local queries on datasets under ~1GB per user.

The bulk of the page is implementation guidance: three patterns (Embedded Dives — iframe-embedded, natural-language-authored dashboards on the Business plan; 3-tier with server-side auth and ~50–100ms queries; 1.5-tier DuckDB-Wasm with ~5–20ms local queries and offline support), a comparison table across them, per-pattern performance optimizations (pre-aggregation, single multi-aggregate SQL statements, batched writes, Parquet-compressed initial loads under 50MB, IndexedDB persistence), and an FAQ. An aside covers AI-driven analytics: natural-language questions over data, the MotherDuck MCP Server with a Dive Viewer rendering inline in MCP-capable chat clients, and a pointer to building analytics agents.
