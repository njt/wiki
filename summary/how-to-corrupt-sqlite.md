---
url: https://www.sqlite.org/howtocorrupt.html
title: "How To Corrupt An SQLite Database File"
author: The SQLite Team (D. Richard Hipp et al.)
date_fetched: 2026-07-18
date_published: 2026-04-13
topics:
  - databases-and-data
---

SQLite is highly resistant to corruption — it automatically rolls back
partially-written transactions after a crash or power failure. This document,
maintained by the SQLite team, catalogs the known ways a database can still go
bad. It's part reference, part cautionary guide for application developers.

The causes fall into eight categories. **File overwrites** from rogue processes
or threads — the database is an ordinary file and can be overwritten by
anything with filesystem access. Classic examples include reusing a closed file
descriptor that SQLite then reopens, or making backup copies mid-transaction.
Deleting or swapping hot journal files (-journal, -wal) during crash recovery
also leads to corruption.

**File locking problems** are the second major category. POSIX advisory locking
has a notorious design flaw: `close()` on *any* file descriptor cancels *all*
locks on that file across all threads in the process. A thread reading the
database file directly (bypassing SQLite) triggers this. Other locking
pitfalls: multiple copies of SQLite linked into one app, mixing locking
protocols, unlinking/renaming a database while open, multiple hard/soft links
to the same file, and carrying database connections across `fork()`.

**Sync failures** occur when hardware lies about flushing writes to persistent
storage (common in consumer disks and USB sticks), or when developers disable
syncs via `PRAGMA synchronous=OFF` for speed. WAL mode is more forgiving of
sync failures than rollback journal mode.

**Disk/flash failures** include bit flips, non-powersafe flash controllers
corrupting unrelated files on power loss, and fraudulent USB sticks that
silently overwrite data when their real capacity is exceeded.

**Memory corruption** in the application's address space — stray pointers or
buffer overruns — can corrupt SQLite internals. Memory-mapped I/O makes this
worse, since stray writes to the mapped region corrupt the database file
directly.

**OS-specific issues** include broken mmap() on QNX, defunct LinuxThreads
locking semantics on very old Linux, and general filesystem corruption from
kernel bugs.

**Configuration errors** arise when SQLite's built-in protections are
deliberately disabled: `PRAGMA synchronous=OFF`, `journal_mode=OFF` or
`MEMORY`, `writable_schema=ON` + DML, or changing `schema_version` while other
connections are open.

Finally, the document acknowledges **bugs in SQLite itself**. Despite ~2
billion deployments and extensive testing (including simulated power loss, I/O
errors, and OOM), historical bugs have caused corruption — a race condition in
WAL-mode writes (the "WAL-reset bug" present from 3.7.0 through 3.51.2), stale
expression indexes when moving across platforms, and a boundary-value error in
nested-transaction journals (3.35.0, fixed in 3.37.2), among others.

The document doubles as practical advice: use WAL mode, don't bypass the SQLite
library for file access, keep journal files with their databases, and be wary
of network filesystems (especially NFS).
