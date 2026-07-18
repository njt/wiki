---
url: https://www.sqlite.org/howtocorrupt.html
title: How To Corrupt An SQLite Database File
author: The SQLite Team (D. Richard Hipp et al.)
date_fetched: 2026-07-18
date_published: 2026-04-13
---

# How To Corrupt An SQLite Database File

## Overview

An SQLite database is highly resistant to corruption. If an application crash, or an operating-system crash, or even a power failure occurs in the middle of a transaction, the partially written transaction should be automatically rolled back the next time the database file is accessed. The recovery process is fully automatic and does not require any action on the part of the user or the application.

Though SQLite is resistant to database corruption, it is not immune. This document describes the various ways that an SQLite database might go corrupt.

## 1. File overwrite by a rogue thread or process

SQLite database files are ordinary disk files. That means that any process can open the file and overwrite it with garbage. There is nothing that the SQLite library can do to defend against this.

### 1.1. Continuing to use a file descriptor after it has been closed

We have seen multiple cases where a file descriptor was open on a file, then that file descriptor was closed and reopened on an SQLite database. Later, some other thread continued to write into the old file descriptor, not realizing that the original file had been closed already. But because the file descriptor had been reopened by SQLite, the information that was intended to go into the original file ended up overwriting parts of the SQLite database, leading to corruption of the database.

One example of this occurred circa 2013-08-30 on the canonical repository for the Fossil DVCS. In that event, file descriptor 2 (standard error) was being erroneously closed (by stunnel, we suspect) prior to sqlite3_open_v2() so that the file descriptor used for the repository database file was 2. Later, an application bug caused an assert() statement to emit an error message by invoking write(2,...). But since file descriptor 2 was now connected to a database file, the error message overwrote part of the database. To guard against this kind of problem, SQLite version 3.8.1 (2013-10-17) and later refuse to use low-numbered file descriptors for database files.

Another example of corruption caused by using a closed file descriptor was reported by facebook engineers in a blog post on 2014-08-12.

### 1.2. Backup or restore while a transaction is active

Systems that run automatic backups in the background might try to make a backup copy of an SQLite database file while it is in the middle of a transaction. The backup copy then might contain some old and some new content, and thus be corrupt.

Safe approaches to making backup copies: sqlite3_rsync, VACUUM INTO, the backup API. It is also safe to make a copy of an SQLite database file as long as there are no transactions in progress while the copy is taking place. If the previous write transaction failed, then it is important that any rollback journal (the *-journal file) or write-ahead log (the *-wal file) be copied together with the database file itself.

### 1.3. Deleting a hot journal

SQLite normally stores all content in a single disk file. However, while performing a transaction, information necessary to recover the database following a crash or power failure is stored in auxiliary journal files. Such journal files are described as "hot". The journal files have the same name as the original database file with the addition of -journal or -wal suffix.

SQLite must see the journal files in order to recover from a crash or power failure. If the hot journal files are moved, deleted, or renamed after a crash or power failure, then automatic recovery will not work and the database may go corrupt.

### 1.4. Mispairing database files and hot journals

The state of an SQLite database is controlled by both the database file and the journal file. In a quiescent state, the journal file does not exist and only the database file matters. But if the journal file does exist, it must be kept together with the database to avoid corruption. Actions likely to lead to corruption: swapping journal files between databases, overwriting a journal file with a different one, moving a journal file, copying a database without its journal, overwriting a database file without deleting its hot journal.

## 2. File locking problems

SQLite uses file locks on the database file, and on the write-ahead log or WAL file, to coordinate access between concurrent processes. Without coordination, two threads or processes might try to make incompatible changes to a database file at the same time, resulting in database corruption.

### 2.1. Filesystems with broken or missing lock implementations

SQLite depends on the underlying filesystem to do locking as the documentation says it will. But some filesystems contain bugs in their locking logic such that the locks do not always behave as advertised. This is especially true of network filesystems and NFS in particular.

### 2.2. Posix advisory locks canceled by a separate thread doing close()

The default locking mechanism used by SQLite on unix platforms is POSIX advisory locking. Unfortunately, POSIX advisory locking has design quirks that make it prone to misuse and failure. In particular, any thread in the same process with a file descriptor that is holding a POSIX advisory lock can override that lock using a different file descriptor. One particularly pernicious problem is that the close() system call will cancel all POSIX advisory locks on the same file for all threads and all file descriptors in the process.

So, for example, suppose a multi-thread process has two or more threads with separate SQLite database connections to the same database file. Then a third thread comes along and wants to read something out of that same database file on its own, without using the SQLite library. The third thread does an open(), a read() and then a close(). One would think this would be harmless. But the close() system call caused the locks held on the database by all the other threads to be dropped. Those other threads have no way of knowing that their locks have just been trashed (POSIX does not provide any mechanism to determine this) and so they keep on running under the assumption that their locks are still valid. This can lead to two or more threads or processes trying to write to the database at the same time, resulting in database corruption.

Note that it is perfectly safe for two or more threads to access the same SQLite database file using the SQLite library. This problem only arises when a thread tries to bypass the SQLite library and read the database file directly.

Beginning with SQLite version 3.51.0 (2025-11-04), SQLite implements additional defenses to try to avoid problems caused by locks that are broken by close().

### 2.3. Multiple copies of SQLite linked into the same application

SQLite takes steps to work around the quirks of POSIX advisory locking. Part of that work-around involves keeping a global list (mutex protected) of open SQLite database files. But, if multiple copies of SQLite are linked into the same application, then there will be multiple instances of this global list. Database connections opened using one copy of the SQLite library will be unaware of database connections opened using the other copy, and will be unable to work around the POSIX advisory locking quirks.

### 2.4. Two processes using different locking protocols

It is important that all connections to the same database file use the same locking protocol. If one application is using POSIX advisory locks and another application is using dot-file locking, then the two applications will not see each other's locks and will not be able to coordinate database access, possibly leading to database corruption.

### 2.5. Unlinking or renaming a database file while in use

If two processes have open connections to the same database file and one process closes its connection, unlinks the file, then creates a new database file in its place with the same name and reopens the new file, then the two processes will be talking to different database files with the same name. Since rollback journals and WAL files are based on the name of the database file, the two different database files will share the same rollback journal or WAL file. A rollback or recovery for one of the databases might use content from the other database, resulting in corruption.

### 2.6. Multiple links to the same file

If a single database file has multiple links (either hard or soft links) then that is just another way of saying that the file has multiple names. If two or more processes open the database using different names, then they will use different rollback journals and WAL files. That means that if one process crashes, the other process will be unable to recover the transaction in progress because it will be looking in the wrong place for the appropriate journal.

### 2.7. Carrying an open database connection across a fork()

Do not open an SQLite database connection, then fork(), then try to use that database connection in the child process. All kinds of locking problems will result and you can easily end up with a corrupt database. SQLite is not designed to support that kind of behavior. Any database connection that is used in a child process must be opened in the child process, not inherited from the parent.

Do not even call sqlite3_close() on a database connection from a child process if the connection was opened in the parent.

## 3. Failure to sync

In order to guarantee that database files are always consistent, SQLite will occasionally ask the operating system to flush all pending writes to persistent storage then wait for that flush to complete. This is accomplished using the fsync() system call under unix and FlushFileBuffers() under Windows. We call this flush of pending writes a "sync".

Actually, if one is only concerned with atomic and consistent writes and is willing to forego durable writes, the sync operation does not need to wait until the content is completely stored on persistent media. Instead, the sync operation can be thought of as an I/O barrier. As long as all writes that occur before the sync are completed before any write that happens after the sync, no database corruption will occur.

### 3.1. Disk drives that do not honor sync requests

Unfortunately, most consumer-grade mass storage devices lie about syncing. Disk drives will report that content is safely on persistent media as soon as it reaches the track buffer and before actually being written to oxide. This makes the disk drives seem to operate faster. And in fairness, the lie normally causes no harm, as long as there is no power loss or hard reset prior to the track buffer actually being written to oxide. But if a power loss or hard reset does occur, and if that results in content that was written after a sync reaching oxide while content written before the sync is still in a track buffer, then database corruption can occur.

USB flash memory sticks seem to be especially pernicious liars regarding sync requests. Pulling out the memory stick while the LED is still flashing will frequently result in database corruption.

Note that SQLite must believe whatever the operating system and hardware tell it about the status of sync requests. However, SQLite in WAL mode is far more forgiving of out-of-order writes than in the default rollback journal modes. In WAL mode, the only time that a failed sync operation can cause database corruption is during a checkpoint operation.

### 3.2. Disabling sync using PRAGMAs

The sync operations that SQLite performs to help ensure integrity can be disabled at run-time using the synchronous pragma. By setting PRAGMA synchronous=OFF, all sync operations are omitted. This makes SQLite seem to run faster, but it also allows the operating system to freely reorder writes, which could result in database corruption if a power failure or hard reset occurs.

## 4. Disk Drive and Flash Memory Failures

An SQLite database can become corrupt if the file content changes due to a disk drive or flash memory failure. It is very rare, but disks will occasionally flip a bit in the middle of a sector.

### 4.1. Non-powersafe flash memory controllers

We are told that in some flash memory controllers the wear-leveling logic can cause random filesystem damage if power is interrupted during a write. This can manifest, for example, as random changes in the middle of a file that was not even open at the time of the power loss.

### 4.2. Fake capacity USB sticks

There are many fraudulent USB sticks in circulation that report to have a high capacity (ex: 8GB) but are really only capable of storing a much smaller amount (ex: 1GB). Attempts to write on these devices will often result in unrelated files being overwritten.

## 5. Memory corruption

SQLite is a C-library that runs in the same address space as the application that it serves. That means that stray pointers, buffer overruns, heap corruption, or other malfunctions in the application can corrupt internal SQLite data structure and ultimately result in a corrupt database file. Normally these kinds of problems manifest themselves as segfaults prior to any database corruption occurring, but there have been instances where application code errors have caused SQLite to malfunction subtly so as to corrupt the database file rather than panicking.

The memory corruption problem becomes more acute when using memory-mapped I/O. When all or part of the database file is mapped into the application's address space, then a stray pointer that overwrites any part of that mapped space will immediately corrupt the database file, without requiring the application to do a subsequent write() system call.

## 6. Other operating system problems

### 6.1. Linux Threads

Some older versions of Linux used the LinuxThreads library for thread support. LinuxThreads is similar to Pthreads, but is subtly different with respect to handling of POSIX advisory locks. SQLite versions 2.2.3 through 3.6.23 recognized that LinuxThreads were being used at runtime and took appropriate action to work around the non-standard behavior of LinuxThreads. But most modern Linux implementations make use of the newer, and correct, NPTL implementation of Pthreads. Beginning with SQLite version 3.7.0 (2010-07-21), the use of NPTL is assumed.

### 6.2. Failures of mmap() on QNX

There exists some subtle problem with mmap() on QNX such that making a second mmap() call against a single file descriptor can cause the memory obtained from the first mmap() call to be zeroed. SQLite on unix uses mmap() to create a shared memory region for transaction coordination in WAL mode, and it will call mmap() multiple times for large transactions. The QNX mmap() has been demonstrated to corrupt database file under that scenario.

When running on QNX, it is recommended that memory-mapped I/O never be used. Furthermore, to use WAL mode, it is recommended that applications employ the exclusive locking mode.

### 6.3. Filesystem Corruption

Since SQLite databases are ordinary disk files, any malfunction in the filesystem can corrupt the database. Filesystems in modern operating systems are very reliable, but errors do still occur. For example, on 2013-10-01 the SQLite database that holds the Wiki for Tcl/Tk went corrupt a few days after the host computer was moved to a dodgy build of the (linux) kernel that had issues in the filesystem layer.

## 7. SQLite Configuration Errors

SQLite has many built-in protections against database corruption. But many of these protections can be disabled by configuration options. If protections are disabled, database corruption may occur.

Examples of disabling the built-in protection mechanisms:
- Setting PRAGMA synchronous=OFF can cause the database to go corrupt if there is an operating-system crash or power failure
- Changing the PRAGMA schema_version while other database connections are open
- Using PRAGMA journal_mode=OFF or PRAGMA journal_mode=MEMORY and taking an application crash in the middle of a write transaction
- Setting PRAGMA writable_schema=ON and then changing the database schema using DML statements can render the database completely unreadable

## 8. Bugs in SQLite

SQLite is very carefully tested to help ensure that it is as bug-free as possible. Among the many tests that are carried out for every SQLite version are tests that simulate power failures, I/O errors, and out-of-memory (OOM) errors to verify that no database corruption occurs during any of these events. SQLite is also field-proven with approximately two billion active deployments with no serious problems.

Nevertheless, no software is 100% perfect. There have been a few historical bugs in SQLite (now fixed) that could cause database corruption. And there may be yet a few more that remain undiscovered. Because of the extensive testing and widespread use of SQLite, bugs that result in database corruption tend to be very obscure.

### 8.1. Race condition when writing to a WAL-mode database (WAL-reset bug)

When there are two or more database connections in separate threads or processes, both open on the same WAL-mode database, and if both connections try to write or run checkpoint at the same time, then there is a race condition that might corrupt the database file. It existed in all versions of SQLite from 3.7.0 through 3.51.2.

### 8.2. Stale expression indexes

An "expression index" is an index that references an expression or a VIRTUAL generated column. Expression indexes are suppose to use only deterministic functions. However, when moving a database across different platforms, or when changing SQLite versions, sometimes one of these supposedly deterministic functions will change its output slightly. This can cause the index to appear to be corrupt.

### 8.3. False corruption reports due to database shrinkage

If a database is written by SQLite version 3.7.0 or later and then written again by SQLite version 3.6.23 or earlier in such a way as to make the size of the database file decrease, then the next time that SQLite version 3.7.0 accesses the database file, it might report that the database file is corrupt. The database file is not really corrupt, however. Version 3.7.0 was simply being overly zealous in its corruption detection.

### 8.4. Corruption following switches between rollback and WAL modes

Repeatedly switching an SQLite database in and out of WAL mode and running the VACUUM command in between switches, in one process or thread, can cause another process or thread that has the database file open to miss the fact that the database has changed. That second process or thread might then try to modify the database using a stale cache and cause database corruption.

### 8.5. I/O error while obtaining a lock leads to corruption

If the operating system returns an I/O error while attempting to obtain a certain lock on shared memory in WAL mode then SQLite might fail to reset its cache, which could lead to database corruption if subsequent writes are attempted.

### 8.6. Database pages leak from the free page list

A bug in SQLite that was present in version 3.6.16 through 3.7.2 might cause pages to go missing out of the free list when incremental_vacuum was used.

### 8.7. Corruption following alternating writes from 3.6 and 3.7

If a database were originally created using SQLite version 3.7.0, then written by SQLite version 3.6.23.1 such that the size of the database file increased, then written again by SQLite version 3.7.0, the database file could go corrupt.

### 8.8. Race condition in recovery on Windows systems

SQLite version 3.7.16.2 fixes a subtle race condition in the locking logic on Windows systems. This race condition had apparently existed in all prior versions of SQLite for Windows going back to 2004.

### 8.9. Boundary value error in the secondary journals used by nested transactions

In SQLite version 3.35.0 (2021-03-12), a new optimization was added so that when SQLite is holding secondary journals in memory, less memory will be used. Unfortunately, a boundary check in the new logic was coded incorrectly. What should have been a "<" operator was coded as "<=". This problem was discovered by an independent researcher who was attempting to find bugs in SQLite using a fuzzer. This problem was fixed in version 3.37.2 (2022-01-06).
