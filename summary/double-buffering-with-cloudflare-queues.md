---
url: https://bellone.com/technical/double-buffering-with-cloudflare-queues
title: "Double Buffering with Cloudflare Queues"
author: Bellone
date_fetched: 2026-11-02
date_published: unknown
topics:
  - distributed-systems
  - databases-and-data
---

A short engineering note on building a double-buffered telemetry pipeline on Cloudflare's serverless stack: two queues alternate as "active" and "inactive" each UTC hour, producers route by modulo math on the event timestamp, and a consumer triggers on the half-hour to drain the queue that just went inactive.

The pipeline chains Workers → Queues → Workflows → R2. The producer reads the payload's timestamp and routes the message to whichever queue is currently active. The consumer fires hourly on the half-hour, looks back one hour to determine which queue is now inactive (and therefore safe to drain), compresses the batch, and writes it to R2 with a simple partitioning scheme keyed on UTC hour.

The motivation is as much economic as architectural: by aligning batches to UTC hours — and using UTC specifically to sidestep DST handling — the pattern turns a chaotic event stream into a compressed, hourly-partitioned archive that the author calls a "serverless data lake optimized for downstream analytics," while minimizing Cloudflare billing. Each queue gets a full hour to drain before its data is needed, so the double-buffer smooths out the race between ingestion and processing.
