# Microsoft Fabric Mirroring for SQL Server

Redgate's Simple-Talk guide to Microsoft Fabric mirroring — the near-real-time replication of SQL Server data into OneLake — makes its real contribution in a version split: SQL Server 2016–2022 mirror via CDC, while SQL Server 2025 switches to a Fabric change-feed mechanism that demands Azure Arc, works only on-premises, and invalidates every troubleshooting habit built on the CDC path.

---

## What it covers

The article is a fit-and-operations guide, not a tutorial's first five minutes. It answers: when mirroring is the right tool (near-current analytical copies without a batch pipeline), when it is not (heavy reshaping, unsupported source configuration, features it doesn't replicate), how the mechanism differs by SQL Server version, what 2025 setup actually requires, the full blocker matrix at database/table/column level, and a concrete troubleshooting ladder down to `log_reuse_wait_desc`.

The version split is the load-bearing fact. CDC-based mirroring for 2016–2022 works on-premises, on Azure VMs, and even on AWS/GCP; the 2025 change-feed version is closer to Azure SQL Database's mechanism, uses entirely different DMVs and stored procedures, and currently supports *only* on-premises instances — no Azure VMs, no Linux.

## Key quotes

> "Mirroring is a good fit when you or your organization wants current or near-current analytical copies without building a more traditional batch pipeline. However, mirroring is not the best choice for every SQL Server integration need."

The honest-fit framing is what makes this guide useful. Most vendor-adjacent replication content sells the feature; this one leads with the boundary — freshness short of synchronous replicas, and transformation deferred rather than eliminated.

> "Data transformation still needs to happen somewhere in the system. Mirroring fits when the immediate need is simply to replicate operational data into Fabric and make it available quickly."

A sentence that should be stapled to every "ELT is dead" pitch. Mirroring moves the transform problem; it doesn't remove it.

> "I do think this functionality is better (and this is a better direction for mirroring overall), but there is currently one key limitation: SQL Server 2025 mirroring is currently only supported for on-premises instances."

The author's own verdict, and a sharp one: Microsoft rebuilt the mechanism mid-family, so playbooks — setup, DMVs, stored procedures, troubleshooting assumptions — do not transfer from the CDC generation to the change-feed one.

> "Over the years, I've been disappointed by the lack of support for managed identities across the SQL Server product line, including in Azure Analysis Services (AAS)."

The identity story for 2025 mirroring rides entirely on Azure Arc, which makes the "simple" feature a hybrid-management commitment before it is a replication feature. That disappointment is doing real analytical work in the piece.

> "If you have a transaction log growing out of hand and not truncating, it could be that mirroring is preventing log truncation for the mirrored database. … If it says `REPLICATION`, it is waiting for mirroring."

The operational dependency laid bare: mirroring scans the transaction log, so the log becomes a shared resource between OLTP health and analytics freshness. A classic CDC-class failure mode returning in change-feed clothes.

## Themes

#concept — CDC versus change feed as two generations of change-capture; near-real-time replication as a distinct product from HA replication. #tool — Fabric mirroring, Azure Arc, the change-feed DMVs. #pattern — replication-as-ETL-avoidance, with the transform bill simply moving downstream.

## Analysis

The article's quiet thesis is that a replication mechanism change is an *operational* event, not a version footnote. When Microsoft moved mirroring from CDC to the change feed, it also moved the diagnostics (`sys.dm_change_feed_*` instead of CDC tables), the permission model (managed identity via Arc), and the topology constraints (on-premises only). Any team whose runbook assumes one mechanism will misdiagnose the other — the guide's version-split framing is the part worth stealing for any integration decision.

The Arc requirement deserves skepticism. A data-movement feature that requires enrolling the database server in Azure's management plane is doing two things at once: solving identity (the author's long-standing complaint about managed identities in the SQL Server line) and deepening platform lock-in. The least-privilege guidance is genuine — a dedicated user with exactly four grants, and no admin requirement — but the availability-group story complicates it: every secondary replica's system-assigned managed identity needs contributor rights on the Fabric workspace, so failover readiness quietly widens the write-blast-radius into the analytics estate.

The blocker list reads like a feature younger than its marketing. No `JSON` or `VECTOR` column types mirrored — in a pipeline whose whole point is feeding analytics, and from an engine (SQL Server 2025) that ships a native `VECTOR` type — is the standout gap. RLS/OLS/DDM not replicating means security posture does not travel with the data, which turns every mirrored copy into a place where access decisions must be re-imposed rather than inherited. And the 1,000-table ceiling plus cross-tenant prohibition shape architecture more than any pitch deck will admit.

Finally, the log-truncation troubleshooting is the piece with the longest half-life: any log-scanning replication consumes the transaction log as a shared resource, and `log_reuse_wait_desc = REPLICATION` is the same diagnostic shape whether the consumer is CDC, replication, or the change feed. That principle predates and will outlast Fabric.

## Related pages

This is Microsoft's entry in the same family as [[Postgres to Snowflake Data Mirroring]] — a vendor pushing CDC out of the OLTP engine into its analytics platform instead of pulling — and the 2025 change-feed redesign strengthens that page's pattern: the mechanism is a moving target even within one vendor's version line. Against [[Streambed]]'s single-binary Postgres-to-Iceberg CDC and [[Artie]]'s managed no-Kafka replication, Fabric mirroring is the same "skip the middle bus" bet taken further: the platform itself becomes the replication target, with ETL complexity traded for platform prerequisites (Arc, gateways, workspace permissions). And the log-truncation section is a modern instance of what [[ARIES — Write-Ahead Logging Recovery]] formalized: the write-ahead log is the system's spine, and every incremental consumer that reads it taxes the database that owns it.

---
*Sources: [[raw/how-to-mirror-data-from-sql-server-to-microsoft-fabric-complete-guide]], [[summary/how-to-mirror-data-from-sql-server-to-microsoft-fabric-complete-guide]]*
*Last updated: 2026-09-13*
