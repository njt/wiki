# MotherDuck Customer-Facing Analytics

MotherDuck's documentation page argues that customer-facing analytics (CFA) — analytics embedded in your product for *external* users rather than internal BI — breaks traditional data architecture, and that a DuckDB-based engine with per-customer isolated instances and browser-side execution answers it. It is marketing, but the marketing is load-bearing: it stakes a real architectural claim (single-node beats distributed for per-tenant workloads) with concrete numbers.

---

## What it argues

Three challenges define CFA: the OLTP/OLAP stack mismatch (data lives in Postgres; users live in JavaScript; neither tool serves the other), latency (users expect sub-second, distributed warehouses can't deliver it because of cold starts and coordination overhead), and multi-tenancy at scale (shared clusters mean noisy neighbors, overprovisioning, and all customers' data in one system).

The answers, both rooted in DuckDB's in-process design:

- **Hypertenancy** — one Duckling (dedicated DuckDB instance) per customer or per user. Zero coordination overhead, vertical scaling per tenant, ~100ms cold starts, per-second billing. The honest advisory note: "start simpler. Begin with a single Duckling per customer."
- **Dual Execution** — the same SQL engine in the cloud or compiled to WebAssembly in the browser, so read-heavy dashboards on <1GB per user query locally in ~5–20ms, with users supplying the compute.

It then lays out three implementation patterns with a comparison table: Embedded Dives (iframe dashboards, no frontend to build, Business plan), 3-tier (server-side auth, ~50–100ms, connection pooling saves ~200ms/request), and 1.5-tier DuckDB-Wasm (near-zero server cost, offline support, unlimited scalability). Practical optimizations follow: pre-aggregation, one multi-aggregate SQL statement over many round trips, batched writes ("MotherDuck is analytical, not transactional"), Parquet initial loads under 50MB, IndexedDB persistence.

## Key quotes

> "For CFA workloads that query one customer's data at a time, single-node execution is usually faster than distributed."

The load-bearing claim of the whole page, and it's correct as far as it goes — the counterpoint the page doesn't make is that this holds precisely because CFA shards by tenant. Distributed warehouses exist for workloads that *can't* fit one node; MotherDuck is arguing that per-tenant analytics never needs to be one of them.

> "Infinite scalability (users provide compute)"

The 1.5-tier pitch in six words. DuckDB-Wasm turns every customer's laptop into part of your serving infrastructure — the same move as [[Drilldown Dashboards from a Single Parquet File]], industrialized. The <1GB-per-user and <50MB-initial-load ceilings are where the trick's leverage ends.

> "MotherDuck is analytical, not transactional: if queries feel slow, set the right expectations and reshape OLTP-style write patterns into batches."

Rare candor in vendor docs: an explicit admission of what the engine will not do, framed as advice rather than a limitation buried in release notes.

> "start simpler. Begin with a single Duckling per customer"

A vendor resisting its own architecture's maximalism — one Duckling per *user* is sold in the headline and walked back in the body.

## Themes

#concept #tool #pattern

## Analysis

The page's real thesis is a bet against the distributed-warehouse consensus: that the CFA workload class — many small, isolated, latency-sensitive queries, one tenant at a time — is better served by fleets of single-node engines than by one big cluster. That's the composability camp's argument (see the compiled [[Databases and Data]] page's durable-layer/composable split) applied at the tenancy layer rather than the storage layer. It's persuasive for the stated workload and quietly silent about cross-tenant queries, fleet-wide joins, or tenants whose data grows past Duckling sizes.

The AI aside is thin but strategically telling: an MCP server with a "Dive Viewer" that renders analytics inline in chat clients positions the same dashboard artifact to be authored by natural language and consumed by agents. That makes this page partly an analytics-for-agents document — the charting layer becomes something an agent writes and an iframe renders, with `initial_state` seeding filters per session. Notably absent is any mention of agent *accuracy* over the data, which is exactly where the compiled page's evidence says agents fail hardest on real enterprise data.

Worth reading as a position statement, not a guide: the implementation patterns section is genuinely useful, the benchmark-flavored numbers (sub-100ms cold starts, ~50–100ms 3-tier queries) are unaudited vendor claims, and the comparison table omits the operational cost of running hundreds of per-tenant compute instances.

## Related pages

- [[DuckDB ADBC Extension]] — the same engine reaching outward to 30+ systems via Arrow-native connectors; this page shows DuckDB being pushed in the opposite direction, as the serving tier inside other people's products.
- [[Replace Athena with DuckDB (Lambda)]] — serverless DuckDB for ad-hoc analytics; the 3-tier pattern here is the always-on commercial sibling of that trick.
- [[Drilldown Dashboards from a Single Parquet File]] — the hand-rolled ancestor of the 1.5-tier pattern: a Parquet cube and 18KB JS reader where MotherDuck ships DuckDB-Wasm plus a managed backend.
- [[SQLite Is All You Need]] — the composable minimal-engine thesis; MotherDuck's hypertenancy is the managed-cloud version of "many small instances beat one big cluster."

---
*Sources: [[raw/customer-facing-analytics]], [[summary/customer-facing-analytics]]*
*Last updated: 2026-10-09*
