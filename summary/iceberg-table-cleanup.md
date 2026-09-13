---
url: https://lakeops.dev/blog/iceberg-table-cleanup
title: "Iceberg Table Cleanup"
author: LakeOps
date_fetched: 2026-09-13
date_published: unknown
topics:
  - databases-and-data
  - distributed-systems
---

A production-depth guide to Apache Iceberg table maintenance from LakeOps, the vendor behind an autonomous Iceberg "control plane." The thesis: Iceberg's append-only architecture (every write creates new files and a new snapshot, nothing is overwritten or garbage-collected) makes cleanup mandatory, and while Iceberg ships the four procedures — `expire_snapshots`, `remove_orphan_files`, `rewrite_data_files`, `rewrite_manifests` — it ships no intelligence about when to run them, in what order, with what parameters. Unattended, four things grow without bound: snapshots, orphan files, manifests, and merge-on-read delete files.

**Sequencing is load-bearing.** The correct order is expire → orphans → compact → manifests. Expire before orphan cleanup or the largest pool of reclaimable storage is invisible; expire before compaction or you rewrite files expiration would have deleted; compact before manifest rewriting or the rewritten manifests are instantly stale. Orphan cleanup is the most dangerous operation: a URI scheme mismatch (`s3://` in metadata vs `s3a://` from the storage listing) makes every file look orphaned and can delete an entire table in one run — hence dry-run first, 7+ day retention, scheme verification. Compaction resolves delete files (position/equality deletes, V3 deletion vectors) and needs partial-progress commits on tables over 100 GB or a late failure wastes hours of compute and creates new orphans.

**Streaming tables are the hard case.** Cleanup operations compete with writers under optimistic concurrency control, so OCC conflicts are expected events, not errors: raise retries to 10–20, exclude active partitions from compaction scope, align snapshot expiration to 3× the Flink checkpoint interval, and use the `Merge` rewrite strategy instead of `Sort`. Snapshot isolation beats serializable for streaming-plus-compaction.

**Cost and compliance.** Per-operation cost breakdown: orphan cleanup is pure waste removal (15–30% of storage spend on a mature lake), compaction is the big compute/query win (100× reduction in S3 GET costs in their example), manifest rewriting is the cheapest with the biggest planning-time impact. The GDPR section is the most under-appreciated: a logical `DELETE` is not physical erasure — data is recoverable until expiration, compaction, and orphan removal all complete, and time travel can still serve deleted rows from historical snapshots. Erasure latency = retention policy + compaction cadence, and it should be documented in data processing agreements.

**The vendor turn.** The final sections argue that manual Airflow-DAG-per-table cleanup works at 10 tables and collapses at 100+, because "cleanup is not a scheduling problem — it is a continuous optimization problem" — the pitch for the LakeOps control plane, which monitors table health across catalogs and runs the four-step pipeline autonomously on a Rust/DataFusion engine, with Autopilot, Manual Approval, and policy-driven modes.
