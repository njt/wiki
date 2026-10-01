---
url: https://blog.cloudflare.com/cloudflare-k2-streams/
title: "Announcing Cloudflare K2: serverless event streams"
author: Cloudflare
date_fetched: 2026-10-02
date_published: 2026-09-30
topics:
  - distributed-systems
  - databases-and-data
---

Cloudflare launches K2, a serverless durable event-streaming primitive in public beta: a partitioned, durable log built on top of R2 object storage rather than Kafka-style local disks. K2 decouples producers from consumers with an ordered log that supports batch consumption, leases with ack/nack/extend, and both work-splitting subscriptions and pub/sub fan-out.

The design story is the interesting part. Cloudflare's edge — 335+ cities of small, ephemeral machine slices over public internet networking — cannot run traditional stateful systems like Kafka. So K2 offloads replication and consensus to R2's strongly consistent, 11-9s-durable object store, keeping the application layer "radically simpler, cheaper, and higher performance," with compute and storage scaling independently. The cost: object stores don't support appends, so events accumulate in-memory and flush as segment files, costing about 1 second of p99 produce latency.

The post positions K2 against Cloudflare's existing primitives: Queues (per-item work with retries and dead-letter queues), Basin Pipelines (ingestion into R2/Iceberg), and K2 (high-scale data movement, long retention, fan-out). Beta limits: 10GB storage, 30 MB/s per stream; planned pricing $0.04/GB produced and consumed, $0.02/GB/month retained. Roadmap includes multi-GB/s streams, key-based ordering, push-based worker consumers, and Kafka client compatibility.
