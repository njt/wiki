---
title: "Anomaly Detection"
url: https://uriv.me/blog/anomaly-detection-with-welford-and-kv
date_fetched: 2026-05-14
section: "Databases and Data"
topics:
  - software-engineering-craft
  - databases-and-data
---

# Anomaly Detection with Welford's Algorithm and KV Storage

## Overview
Uri built **anomalisa**, an open-source anomaly detection service that requires no configuration. The system learns normal event patterns and emails alerts when deviations occur, using only mathematical algorithms and a key-value store.

## Welford's Algorithm
Rather than collecting all values to compute variance, Welford's approach maintains three numbers in constant memory: count, running mean, and sum of squared deviations. The algorithm updates incrementally with high numerical stability because "delta is computed before updating the mean, and delta2 is computed after."

## Hourly Bucket Architecture
Events are grouped into hourly buckets using ISO timestamps truncated to hour boundaries (e.g., `2026-04-05T14`). This creates natural measurement units and the system backfills zeros during quiet periods to prevent models from ignoring gaps.

## Detection Modes

Three independent statistical checks:

1. **Total count z-score** -- detects volume spikes and drops
2. **Percentage spike** -- identifies when one event type becomes disproportionately large
3. **Per-user anomalies** -- catches individual user behavior changes

## Storage Design

The entire KV model includes:
- Hourly counts (7-day TTL)
- Welford state for total and per-user metrics
- Per-user counts (7-day TTL)
- Detected anomalies (30-day TTL)

Atomic check-and-set operations prevent duplicate anomaly alerts from concurrent requests.

## Key Tradeoffs

- Z-score threshold of 2 is hardcoded (roughly 5% false positive rate)
- Requires 3 data points before alerting to avoid noise
- Won't detect failures unrelated to event counts
- Entire detection engine fits in a single file
