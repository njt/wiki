---
url: https://www.sqlservercentral.com/articles/how-we-reduced-sql-server-infrastructure-costs-by-33-on-aws-lessons-from-a-database-operations-manager
title: "How We Reduced SQL Server Infrastructure Costs by 33% on AWS — Lessons from a Database Operations Manager"
author: SQLServerCentral (Database Operations Manager)
date_fetched: 2026-09-20
topics:
  - databases-and-data
  - ai-infrastructure-and-hardware
---

A database operations manager recounts cutting his organisation's SQL Server bill on AWS from $60,000 to $40,000 a month — a 33% reduction, $240,000 a year — in two weeks, with zero production changes. The environment was 27 instances (15 production, 12 non-production) all running Enterprise Edition, with stale snapshots and 24/7 non-production uptime.

Three changes did the work. First, migrating all 12 non-production instances from Enterprise Edition to the free Developer Edition, which has full feature parity but cannot be used in production — $12,000/month, 60% of the savings, after auditing for Enterprise-only feature dependencies. Second, deleting stale snapshots from closed projects that nobody had cleaned up — $6,000/month, achieved with no architecture changes at all, now maintained by a quarterly audit. Third, scheduling non-production instances to run only business hours, after auditing overnight SQL Server Agent jobs so the stop schedules wouldn't kill them mid-run — $2,000/month.

The methodological argument is that the inventory audit came first and drove everything: the snapshot problem was only found because the full picture was built before touching anything. The author also credits engaging the AWS TAM early, before things break, and revisiting decisions that made sense when the environment was built but no longer do.
