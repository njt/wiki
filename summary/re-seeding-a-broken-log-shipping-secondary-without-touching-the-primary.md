---
url: https://www.sqlservercentral.com/articles/re-seeding-a-broken-log-shipping-secondary-without-touching-the-primary
title: "Re-seeding a Broken Log Shipping Secondary Without Touching the Primary"
author: SQL Server Central (article author not named on page)
date_fetched: 2026-10-14
date_published: unknown
topics:
  - databases-and-data
  - distributed-systems
---

A field-tested runbook for resuscitating a SQL Server log shipping secondary whose restore chain broke — here because a version upgrade on the primary left the secondary unable to apply transaction logs. The thesis is that the whole repair happens on the secondary: drop the broken copy, rebuild it from full → differential → bridging transaction log with NORECOVERY, then hand it back to the log shipping jobs. The primary (and its database mirroring) is never touched.

The article's real value is three undocumented gotchas. First, SSMS's default "Delete backup and restore history" checkbox runs one giant transaction against msdb's poorly-indexed history tables, which can fill msdb's log (error 9002) and roll back the drop — script the drop instead, and trim history in date batches. Second, restore chains chain by LSN, not by filename or timestamp; the bridging log is the one whose LSN range contains the differential's LastLSN, and UTC-vs-local filename clocks can mimic a backup lag that isn't there. Third, log shipping stores its "last restored file" pointer in two msdb tables (config and monitor), and a manual re-seed moves the database's LSN but not the pointer — the restore job resumes from the dead chain until you align both.

The closing discipline: keep NORECOVERY on everything you restore by hand, let the log shipping job bring the database to its serving state, and verify an hour later that "time since last restore" is small and staying small — not just that the first restore succeeded.
