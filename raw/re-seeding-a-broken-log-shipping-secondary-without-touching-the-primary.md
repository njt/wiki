---
url: https://www.sqlservercentral.com/articles/re-seeding-a-broken-log-shipping-secondary-without-touching-the-primary
date_fetched: 2026-10-14
---

A SQL Server upgrade broke log shipping on the secondary. It could no longer accept transaction logs because of version incompatibility. The fix is simple: drop the secondary database and then restore it using the full + differential + transaction log chain, in order and WITH NORECOVERY, and then hand it off to the log shipping job. The catch is that the primary database has mirroring configured to another instance. In hindsight this is not really a catch, because we never touch the primary's mirror config, or the primary in general when doing this.

There are basically two ways of starting up log shipping again when a secondary is no longer receiving transaction logs or is broken. One is to use the wizard on the primary, remove the log shipping that is already configured, and create it from scratch again. The other is to leave the primary alone and do all the work on the secondary. There are however a couple of quirks that the documentation doesn't warn you about. This is a writeup of those gotchas, in the order you'll hit them.

The steps below were tested in a lab environment.

**The Scenario**

The Primary instance hosts the AlphaDB database and also runs database mirroring for it. The mirror must not be disturbed.

The Secondary instance holds the log shipping target. Its copy of AlphaDB was no longer able to receive transaction logs because of the upgrade on the primary, breaking the log shipping chain. Transaction logs land on the secondary in a local folder, copied there from a share on the primary. Since the secondary SQL Server Version is lower than the primary, it is no longer able to receive transaction logs. This is why the first step is to upgrade the secondary to the same version as the primary.

It realistically doesn't matter how the log shipping broke. If only the secondary was affected it's basically the same scenario. It could be during a SQL Server version upgrade, a restore on the secondary, and so on. The most important part here is to remember that a restore chain is continued by LSN, not by filename. Full -> differential -> logs link by LSN, and SQL Server enforces that continuity. It will reject a log that's already been applied ("too old") or one that would leave a gap ("too recent"). It physically cannot skip or misapply.

**The Procedure at a Glance**

Everything in this article runs on the secondary instance. It assumes the secondary has already been upgraded to the same version as the primary. The sequence is:

- Verify the backups you are going to restore from, and disable the copy and restore jobs.
- Drop the broken secondary database, using a script rather than the SSMS dialog (Gotcha #1).
- Restore the full backup WITH NORECOVERY.
- Restore the differential WITH NORECOVERY.
- Restore the bridging transaction log WITH NORECOVERY, identified by LSN rather than by filename (Gotcha #2).
- Align log shipping's "last restored file" pointers in msdb (Gotcha #3).
- Re-enable the copy job, then the restore job, and verify.

Steps 2, 5 and 6 are the ones that need a little attention and I've provided 3 "Gotchas" to explain them.

**Gotcha #1: The "Delete backup history" checkbox can fill msdb's log and fail the drop**

Step one is dropping the restored database on the secondary. In SSMS, the Delete Object dialog has a checkbox, "Delete backup and restore history information for databases", and that's ticked by default. On most servers it's harmless. On a busy log shipping instance it can cause some issues, and here's why.

That checkbox runs sp_delete_database_backuphistory, which deletes from msdb's backup and restore history tables. On a log shipped secondary that's been running restores every couple of minutes for a long time, those tables hold a huge amount of data and are poorly indexed. The delete runs as one transaction, and if it is big enough it will fill msdb's own transaction log before it can commit:

The transaction log for database 'msdb' is full due to 'ACTIVE_TRANSACTION'. (Microsoft SQL Server, Error: 9002)

The drop rolls back. Your database is still there, and now msdb's log is full too.

The fix is to never let that checkbox run. Drop with a script instead. If your secondary is configured for NoRecovery rather than Standby, skip the SET SINGLE_USER line.

USE master; ALTER DATABASE AlphaDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE; -- closes connections DROP DATABASE AlphaDB;

If you already hit the 9002, clear msdb (it's normally in SIMPLE recovery, so a checkpoint frees the log once the failed transaction has rolled back):

USE msdb; CHECKPOINT; DBCC SQLPERF(LOGSPACE); -- confirm msdb's % used has dropped

Then trim the bloated history in small date batches rather than one massive delete, so no single transaction can fill the log again. This code slowly removes old history:

EXEC msdb.dbo.sp_delete_backuphistory @oldest_date = '2020-01-01'; EXEC msdb.dbo.sp_delete_backuphistory @oldest_date = '2022-01-01'; EXEC msdb.dbo.sp_delete_backuphistory @oldest_date = '2024-01-01'; -- CHECKPOINT between batches.

Note that sp_delete_backuphistory works instance-wide, not per database.

Then retry the drop with the checkbox unticked.

**Gotcha #2: Chain by LSN, not by filename or timestamp**

After we restore the full and differential WITH NORECOVERY, we need a transaction log backup to bridge from the differential. This transaction log should be the one that started and completed after the differential finished backing up. Its LSN range still starts before the diff's last LSN, because a log backup covers everything since the previous log backup, and that point is earlier than where the diff ended.

Filenames and timestamps are fine for narrowing down which file you probably want, but always confirm by LSN before restoring. A common example: .trn filenames generated in UTC while the file system's modified-time and the SQL Agent schedule are in local time. On a UTC+4 server, a file created at 17:26 local gets named ...132601 (13:26 UTC), so every filename looks four hours behind the clock and mimics a backup lag that isn't there.

When you need the file that bridges from your differential, find the one whose LSN range contains the diff's last LSN:

RESTORE HEADERONLY FROM DISK = N'<candidate>.trn'; -- The bridge file is the one where: FirstLSN <= diff.LastLSN < LastLSN

Restore that one WITH NORECOVERY and the chain is proven.

**Gotcha #3: Log shipping tracks "last restored file" in two msdb tables**

After we manually restore the full, diff and transaction log, if we re-enable the restore job it will fail immediately with an LSN error, trying to resume from a file belonging to the old chain. The reason is that log shipping stores its pointer in msdb, and a manual re-seed only moves the database's actual LSN position, not the pointer. There are two tables involved and they do different jobs:

- msdb.dbo.log_shipping_secondary_databases is the config table. This is the one sqllogship.exe reads to decide which file to restore next, so a stale value here is what actually fails your restore job.
- msdb.dbo.log_shipping_monitor_secondary is the monitor table. This drives the Transaction Log Shipping Status report and the alert job. A stale value here won't break the restore, but it will keep reporting the secondary as out of sync.

Check both of these with this code:

SELECT 'config' AS src, last_restored_file FROM msdb.dbo.log_shipping_secondary_databases WHERE secondary_database = 'AlphaDB' UNION ALL SELECT 'monitor', last_restored_file FROM msdb.dbo.log_shipping_monitor_secondary WHERE secondary_database = 'AlphaDB';

If they don't both name the last .trn you actually restored, align them:

UPDATE msdb.dbo.log_shipping_secondary_databases SET last_restored_file = N'<last restored .trn>' WHERE secondary_database = 'AlphaDB';

UPDATE msdb.dbo.log_shipping_monitor_secondary SET last_restored_file = N'<last restored .trn>', last_restored_date = '<local datetime>', last_restored_date_utc = '<utc datetime>' WHERE secondary_database = 'AlphaDB' AND secondary_id = '<secondary GUID>';

Directly updating these msdb tables is not a Microsoft supported procedure. It is however a low risk update, because we are just changing the pointer, and it is reversible.

**The Rest of the Sequence (the documented part)**

Now that the three traps are out of the way, here is the whole procedure in order. Everything below runs on the secondary instance, and none of it touches the primary or its mirror.

The first step here is to drop the secondary database so make sure that you have the backups from production ready. You can verify with RESTORE HEADERONLY against the full and the differential.

- Check that the full backup shows BackupType = 1 and IsCopyOnly = 0, and that RecoveryModel is FULL.
- Also check that the DifferentialBaseLSN matches the full's CheckpointLSN.
- Confirm that the .trn files are actually on disk.

If you have the backups scheduled then it is recommended to take the latest FULL backup and the differential. If you need to take the backup yourself then keep in mind to not use COPY_ONLY, because it doesn't reset the differential base, so any differential you take afterwards is still anchored to the previous regular full. Its DifferentialBaseLSN won't match your copy-only full's CheckpointLSN, and the restore will fail.

- Disable the copy and restore jobs on the secondary so they can't fire mid-operation.
- Drop the broken secondary database (scripted, per Gotcha #1).
- Restore the full backup WITH NORECOVERY. Note its CheckpointLSN. Use RESTORE FILELISTONLY to get exact logical names for your MOVE clauses. If you use the GUI, double-check the Destination Database field, because SSMS sometimes pre-fills it with whatever database you had selected.
- Restore the differential WITH NORECOVERY. Confirm its DifferentialBaseLSN equals the full's CheckpointLSN. Note the diff's LastLSN.
- Bridge with the first transaction log WITH NORECOVERY (per Gotcha #2, find it by LSN).
- Align the pointers if the restore job resumes from the wrong file (Gotcha #3).
- Re-enable the copy job, let it stage current files, then re-enable the restore job. On its first successful run it converts the database to its configured final state, Standby/Read-Only or NoRecovery, automatically.
- Verify: state_desc matches the configured restore_mode, the restore job history is green, and the Transaction Log Shipping Status report shows "time since last restore" small and staying small. Check it again an hour later to confirm it's genuinely keeping up, not just that the first restore worked.

Keep NORECOVERY on everything you restore by hand. The log shipping restore job is what finally brings the database to its serving state.

The key thing to take away from this is to let the LSNs decide which transaction log to apply and that we only need to work on the secondary instance to resolve this issue.
