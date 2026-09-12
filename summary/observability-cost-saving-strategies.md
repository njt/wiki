---
url: https://michaelscodingspot.com/observability-cost-saving-strategies/
title: Observability Cost Saving Strategies
author: Michael Shpilt
site: Michael's Coding Spot
date_fetched: 2026-08-26
topics:
  - software-engineering-craft
---

Michael Shpilt frames observability spend as a conflict of interest: vendors bill per data volume or per node, both of which have exploded over the last decade, so asking your vendor to help you save money is "asking the fox to guard the henhouse." He then lays out seven cost-reduction strategies the vendor won't volunteer.

The strategies form a rough ladder from least to most effort. **Logs-to-metrics conversion** intercepts raw logs (HTTP requests, load-balancer logs) at the OpenTelemetry Collector or an edge proxy, increments a counter or records a latency histogram, and drops the source log — same dashboard for a fraction of the cost, and it saves the "indexing" portion that is 80–90% of DataDog-style bills. **Cardinality monitoring** attacks the "silent killer": unbound values like `user_id` or `pod_name` attached to custom metrics explode unique timeseries, and vendors don't proactively alert on the spikes. **BYOC** (Bring Your Own Cloud) moves telemetry storage and query into your own S3/GCS/Azure account, so you pay cloud rates plus a platform fee — but it changes the billing *unit* to node count, which favors monoliths and big services over many small microservices. **DPM reduction** cuts metric volume by 80% by lengthening flush intervals from 10s to 60s — but only if your vendor meters data points (Grafana Cloud does; Datadog bills per unique timeseries per hour, so it doesn't). **Removing redundant logs at the source** is the only fix that solves the problem "once and for all," and the article notes its real price includes compute, network, LLM tokens when agents read telemetry, and slower MTTR. **Head/tail sampling** keeps representative traces (tail-based keeps 100% of error/slow traces, 5% of healthy ones). **Tiered log levels** retains full-fidelity INFO+ logs only on a few critical clusters/regions and WARN+ elsewhere.

The article is a companion to Shpilt's earlier "Reduce Logging Costs," and it doubles as a pitch for his own tool, Obics, which finds telemetry redundancies and opens pull requests to fix them at the source.

---
*Source: [[raw/observability-cost-saving-strategies]]*
