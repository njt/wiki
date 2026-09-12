---
url: https://archive.ph/oIfml
title: "The Log: What every software engineer should know about real-time data's unifying abstraction"
author: Jay Kreps
site: LinkedIn Engineering Blog
date_published: 2013-12-16
date_fetched: 2026-08-06
topics:
  - databases-and-data
---

Jay Kreps, then Principal Staff Engineer at LinkedIn and later co-founder/CEO of Confluent, argues that the append-only log is the single most underappreciated abstraction in software engineering. Drawing on his experience building LinkedIn's distributed data infrastructure — including Kafka — he traces the log from its database origins through distributed systems to data integration and stream processing, making the case that the log is the natural data structure for handling data flow between systems.

The article is structured in three parts. **Part One** defines the log as an append-only, totally-ordered sequence of records, then traces its lineage: write-ahead logs in databases (where it enables atomicity, durability, and crash recovery), log shipping for replication (Oracle, MySQL, PostgreSQL), and the State Machine Replication Principle in distributed systems ("two identical, deterministic processes given the same inputs in the same order produce the same output"). Kreps introduces the duality of tables and events: a table is the current state; the log is the complete history of changes from which every past state can be reconstructed.

**Part Two** tackles data integration — making all of an organization's data available in all its systems. Kreps diagnoses two trends making this harder (the event data firehose and the explosion of specialized data systems) and proposes the log as the central pipeline: each data source publishes to its own log, and every subscribing system reads at its own pace. This decouples producers from consumers, turns each subscriber into an independent time-traveler that can catch up after downtime, and collapses the O(N²) point-to-point integration problem into O(N) connections to a central log. He contrasts this with traditional ETL, arguing that conflating extraction/cleanup with warehousing-specific restructuring is a mistake — the clean, integrated data repository should be available in real-time, not just batch.

**Part Three** reframes stream processing as continuous data processing rather than a niche SQL-engine concern. Kreps makes the historical argument that batch processing is a relic of manual, non-digital data collection — as data becomes continuously collected, processing naturally becomes continuous. The log enables this by making every dataset multi-subscriber and ordered, allowing derived feeds computed from other feeds to form a graph of processing stages.

The article is grounded in Kreps' experience building Kafka at LinkedIn. He describes the scaling techniques that made it practical: partitioning, batching for throughput, and zero-copy data transfer — yielding a system that (as of 2013) handled 60 billion message writes per day. The post is widely considered a foundational text in distributed systems engineering, having shaped how an entire generation thinks about data infrastructure.
