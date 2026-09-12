---
url: https://greptime.com/blogs/2026-08-11-observability-three-pillars-history
title: Observability's Three Pillars: A History
author: Greptime
site: Greptime
date_fetched: 2026-08-21
date_published: 2026-08-11
topics:
  - databases-and-data
---

A history of the "three pillars" of observability — metrics, logs, traces — arguing they were never a designed framework but three signals that evolved independently and were grouped together only later. Greptime's central claim: unified storage (all three signals in one columnar store, queryable together, at production scale) is now a solved, even commoditized problem, so the interesting question has moved down a layer — to whether agents, as first-class consumers of observability data, force the database itself to change.

The three signals have separate lineages. Metrics run RRDtool (1999) → Graphite (2008) → Prometheus (2012); logs run Splunk (2003) → Elasticsearch (2010); traces arrived with Google's Dapper paper (2010) and Zipkin (2012). Their engineering constraints genuinely conflict — aggregation vs. full-text search vs. point lookup — and splitting them was the cheaper, likelier-to-succeed path, reinforced by business moats and three separate budget pockets.

The unification argument goes back eight years: Peter Bourgon's 2017 Venn diagram and 2018 "Observability signals" (an "über-system" of shared ingestion with purpose-built backends underneath, a detail the article says is frequently misread), Ben Sigelman's 2018 "Three Pillars, Zero Answers," the OpenTelemetry merger in 2019, and Charity Majors' Observability 2.0 in 2023.

By 2026 every major vendor had connected agents — SigNoz's agent-native observability and MCP server, ClickHouse's ClickStack MCP server and AI Notebooks, Grafana's six GA AI capabilities, Honeycomb's repositioning to serve developers and agents alike. The closing question: unified storage solved data fragmentation inherited from the human era; agents now raise the harder problem of whether data unified in storage is also unified in meaning — and what an observability database must become to serve a machine that reads rather than a human who browses.
