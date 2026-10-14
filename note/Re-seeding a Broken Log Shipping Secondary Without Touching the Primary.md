# Re-seeding a Broken Log Shipping Secondary Without Touching the Primary

A SQL Server Central runbook for repairing a log shipping secondary whose chain broke (a primary upgrade made the secondary unable to apply logs) — entirely on the secondary, without disturbing the primary or its mirror. The procedure is drop-and-rebuild from the backup chain, but the article's substance is three undocumented gotchas: msdb's backup-history delete can fill the transaction log, restore chains are ordered by LSN not filenames, and log shipping's resume pointer lives in two msdb tables that a manual re-seed does not update.

---

The scenario: the primary hosts AlphaDB and runs database mirroring that must not be disturbed; the secondary's copy can no longer receive transaction logs after a version upgrade broke the chain. The author's stance is refreshingly scope-narrow: it doesn't matter how the shipping broke, and the fix is contained to one instance.

The procedure at a glance: verify backups, disable the copy and restore jobs, drop the broken database (scripted, not via the SSMS dialog), restore full → differential → bridging log all WITH NORECOVERY, align the msdb pointers, re-enable copy then restore jobs, and verify — twice.

---

## Key quotes

> "A restore chain is continued by LSN, not by filename. Full -> differential -> logs link by LSN, and SQL Server enforces that continuity. It will reject a log that's already been applied ('too old') or one that would leave a gap ('too recent'). It physically cannot skip or misapply."

This is the article's one theoretical idea, and it's stated with unusual confidence: the restore engine is a deterministic, self-guarding consumer of a totally ordered log. Everything else in the runbook exists because the *tooling around* that engine — SSMS dialogs, msdb bookkeeping, filename conventions — is not so disciplined.

> "The checkbox runs sp_delete_database_backuphistory, which deletes from msdb's backup and restore history tables... The delete runs as one transaction, and if it is big enough it will fill msdb's own transaction log before it can commit."

Gotcha #1 is the best of the three: a default-ticked UI checkbox that destroys the repair before it starts. The remedy is characteristically pragmatic — script the drop, trim history in date batches ("2020-01-01, 2022-01-01, 2024-01-01"), checkpoint between batches — plus the honest caveat that `sp_delete_backuphistory` works instance-wide, not per database.

> "Filenames and timestamps are fine for narrowing down which file you probably want, but always confirm by LSN before restoring."

The UTC-vs-local example (a UTC+4 server where every .trn filename looks four hours behind the clock) is the perfect illustration: human-meaningful labels lie, machine-checked continuity doesn't.

> "Directly updating these msdb tables is not a Microsoft supported procedure. It is however a low risk update, because we are just changing the pointer, and it is reversible."

Gotcha #3's confession is the runbook's most interesting judgment call: documenting an unsupported direct UPDATE because it's small, reversible, and the alternative (reconfiguring log shipping from scratch on the primary) is exactly what the article set out to avoid.

## Key themes

#concept — LSN-ordered restore chains as a self-guarding log consumer
#pattern — repair on the replica, never the primary; blast-radius containment
#tool — msdb internals: config table vs. monitor table, and SSMS defaults that betray you

## Analysis

What makes this a good runbook rather than a checklist is that each gotcha is a small case study in where a well-designed system gets undermined by its periphery. SQL Server's restore engine "physically cannot skip or misapply" — yet the repair can still fail three ways, all in the scaffolding: a GUI default that issues an unbounded transaction against a bloated msdb, human-readable filenames that encode timezones inconsistently, and a resume pointer kept in bookkeeping tables the engine doesn't consult for chain validity but the job does consult for progress.

The article is also a nice argument for the replica-side repair strategy over the wizard's "tear it down and reconfigure" path. The trade is explicit: aligning pointers via direct UPDATE is unsupported, but it's a two-row, reversible change on the secondary, versus reconfiguring log shipping on an instance whose mirroring must not be disturbed. The author chooses blast radius over blessing, and says so — the same judgment any operator makes when choosing the reversible hack over the supported outage.

The verification advice is the quiet moral of the piece: "check it again an hour later to confirm it's genuinely keeping up, not just that the first restore worked." One successful restore proves nothing about a system whose whole job is doing that every couple of minutes — steady state, not single success, is the test.

## Related pages

This source is the concrete SQL Server instance of the abstraction in [[The Log — Unifying Abstraction for Real-Time Data]]: a totally ordered log with LSNs as offsets and a consumer that rejects out-of-order application — Kreps' theory showing up as an error message about files that are "too old" or "too recent".

It strengthens [[ARIES — Write-Ahead Logging Recovery]] with the operational flip side: ARIES is how the engine recovers its own log from a crash, while this runbook is what you do when the *replication* of logs to another instance breaks — both rest on the same LSN-continuity discipline.

It nuances [[Postgres CDC in ClickHouse, A Year in Review]] — that piece is about the happy path of log-based replication running continuously; this is the field manual for when the consumer's chain breaks and the metadata (here msdb's resume pointer, there the replication state) is the thing that's actually stale.

---
*Sources: [[raw/re-seeding-a-broken-log-shipping-secondary-without-touching-the-primary]], [[summary/re-seeding-a-broken-log-shipping-secondary-without-touching-the-primary]]*
*Last updated: 2026-10-14*
