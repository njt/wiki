---
url: https://motherduck.com/docs/getting-started/data-warehouse/
title: "Data Warehousing Overview"
author: MotherDuck
date_fetched: 2026-10-09
date_published: undated (product docs)
topics:
  - databases-and-data
  - mcp-and-tool-protocols
---

MotherDuck's getting-started overview describes its serverless cloud data warehouse built on DuckDB. The architectural pitch is "hypertenancy": every user, service account, or agent gets an isolated compute instance (a "Duckling") that starts in under a second and bills per second — no cluster tuning, no noisy neighbours. Five Duckling sizes scale vertically; read-scaling replicas absorb spiky BI load; Dual Execution splits query work between local DuckDB and the cloud.

The doc walks the classic warehouse lifecycle: ingestion (dltHub, Estuary, Fivetran, Airbyte, plus native "Flights" — scheduled Python jobs running inside MotherDuck), loading best practices (big batched single-table loads, Parquet over CSV for 5–10x compression, queues in front because ACID ≠ OLTP), transformation (SQL or dbt via the Postgres wire-compatible endpoint), sharing (Explorer role with isolated compute), and serving (Power BI/Tableau/Looker over Postgres protocol, or Dives — AI-agent-built dashboards from natural language).

Two threads run through the piece that make it more than product marketing. First, agents are first-class data consumers: connect Claude or Cursor through the MotherDuck MCP Server, and use "Guides" — markdown files of metric definitions and query conventions — to keep agent answers consistent; Flights can be built and maintained by an agent through the same MCP server. Second, the scaling model is deliberately per-principal isolation rather than shared clusters, which is both a performance and a billing story.
