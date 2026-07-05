---
url: https://clickhouse.com/blog/postgres-cdc-year-in-review-2025
title: "Postgres CDC in ClickHouse, A Year in Review"
author: Sai Srirampur
date_fetched: 2026-05-14
date_published: 2025-12-03
---

# Postgres CDC in ClickHouse, A year in review

**Author:** Sai Srirampur — Founder of PeerDB, now at ClickHouse following the acquisition
**Published:** December 3, 2025 · 18-minute read

## The Goal

The core objective was giving customers a simple method to sync transactional data from Postgres to ClickHouse, enabling them to offload analytics workloads onto a purpose-built analytical database.

## The Roots – PeerDB Acquisition

ClickHouse acquired PeerDB in July 2024. Within four months it was integrated as the engine behind ClickPipes' Postgres CDC connector.

**Key decision:** Keeping PeerDB "free and open" as a distinct modular component, with ClickPipes CDC as the managed-service implementation. The author notes this was "critical for engineering velocity and management."

The article includes a GitHub activity comparison chart showing PeerDB vs. Debezium over 12 months.

## Customer Growth Post-Acquisition

- **Growth:** Nearly 100× since acquisition
- **User base:** From a handful pre-acquisition to "more than 400 using it today through ClickPipes"
- **Data volume:** "Collectively replicating over 200 TB of Postgres data to ClickHouse every month"
- **Notable customers:** AutoNation, Seemplicity, Ashby, Vapi, SpotOn, Cyera, LC Waikiki

## Use Cases

### 1. Real-time customer facing analytics

Customers hit Postgres analytical limits sooner due to AI-driven workloads. The author speculates that "the time it takes for Postgres deployments to grow beyond terabyte-scale has shrunk from multiple years to just a few months."

A chart shows the top 10 companies using the connector experienced over 1,000% average data increase across six months, adding more than 85 TB, with most being AI-native companies.

### 2. Data warehousing

Customers consolidate Postgres data into ClickHouse, combining it with other supported data sources for internal BI and analytics.

## Five Favorite Features

### 1. Avoid Reconnecting Replication Connections (Reliability)

On reconnect, Postgres would read WAL from the restart_lsn rather than the last processed position, causing massive lag. The team made infrastructure changes to ensure the replication connection is never dropped. Before/after graphs show dramatic reduction in replication lag during long-running transactions.

### 2. Validate ClickPipes Before Creation (Usability)

Over 50 pre-flight checks implemented, including verifying replication configuration, primary key presence, CDC role permissions, duplicate table detection, version compatibility, and publication existence.

### 3. Initial Load Faster Than Ever (Performance / Community Contribution)

**Key contribution from Cyera employee Alon Zeltser.** Previously, the system ran heavyweight COUNT and window-function queries to partition tables for parallel snapshotting—on multi-terabyte tables these could take hours. The fix replaced this with a "block-based partitioning strategy on the CTID column." Partition generation went from "7+ hours to under a second."

### 4. Bucketized User-Facing Alerts (Usability, Reliability)

The team initially built internal alerts for every action/error (Y Combinator "do things that don't scale" mindset). Beyond 100 customers, on-call load became unmanageable. They grouped errors into 10+ categories and delivered actionable messages via Slack and email. Result: "on-call load dropped by orders of magnitude." The team remains in close contact with 10–20 customers at any time.

### 5. ClickPipe Configurability (Usability)

Hundreds of options added: PrivateLink, SSH tunneling, ordering keys, table engines, hard delete support, adding/removing tables before and after pipeline creation, switching connectivity methods.

**Key learning from the author:** "understanding the true complexity of data-movement / ETL systems" — that enterprise-grade CDC reliability depends on "hundreds, sometimes thousands, of smaller capabilities and edge cases working together."

### Special Mentions

- Faster ingestion via chunking and parallel replicas
- Prometheus/OTEL endpoint for native CDC metrics (replication slot growth, commit lag)
- Reliability through disk spooling and Go channel management improvements

## The Big Gaps – What's Next

### Data Modeling Is Still an Overhead

Most overhead traces to deduplication via `ReplacingMergeTree`. Migrations take "a few weeks" for smaller customers and months for larger ones. Goal is to bring that down to "just a few days."

**Solutions being worked on:**
- Lightweight UPDATE support in Postgres CDC (leveraging recent ClickHouse improvements)
- Unique index support on ReplacingMergeTree for synchronous deduplication
- A Postgres-compatible layer/extension to reduce query migration effort
- JOIN performance improvements for relational/normalized schemas
- Better onboarding and observability for Materialized Views

### Maturing the Platform

- OpenAPI support: currently in beta
- Terraform support: planned for next quarter
- GCP expansion: underway
- BYOC: temporary solution via PeerDB Helm charts

### Reducing the Footguns

- Partial schema change support (ADD/DROP COLUMN supported, not CHANGE TYPE)
- Earlier bugs with nullability changes not propagating when adding tables (now fixed)
- Operational issues like long-running table additions that couldn't be interrupted
- Strengthened unit-testing framework, "close to full code coverage"
- Evaluating a data-consistency view feature

### Scaling Postgres Logical Replication

- Early signals from largest customers needing deeper improvements
- Adding support for Postgres Logical Replication V2, which "allows reading changes from a replication slot before a transaction commits"
- Investigating simultaneous consumption of replication slots during initial snapshots and resyncs
- Plans to increase upstream Postgres contributions (e.g., fixing a logical replication bug related to replicating from standbys)

## Conclusion – Reflections

The author reflects that CDC looks straightforward from the outside—"just read the WAL"—but real workloads expose a far more complex reality involving long-running transactions, replication slot backpressure, schema changes, and edge cases discovered only when a customer hits them at midnight.

The surprising realization was "the amount of iteration required to make the system feel *boring*" — meaning reliable, invisible, and fast.

**Long-term vision:** "unify these two amazing databases as components of a single stack rather than separate databases," involving a more solid replication layer, less data-modeling overhead, and smoother application/query migration.

## Reference Links

- [Postgres CDC in ClickPipes docs](https://clickhouse.com/docs/integrations/clickpipes/postgres)
- [PeerDB open-source repository](https://github.com/PeerDB-io/peerdb)
- [Other ClickPipes data sources](https://clickhouse.com/docs/integrations/clickpipes#supported-data-sources)
