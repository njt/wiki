# How to Corrupt an SQLite Database

The SQLite team's extraordinary catalog of every way their database can be corrupted — a document that builds trust through radical honesty. Eight categories of failure spanning filesystem lies, OS design quirks, application bugs, hardware deception, and SQLite's own historical defects, each documented with the specific version, root cause, and fix. The meta-lesson: a database is only as reliable as the stack beneath it, and "reliable" means understanding exactly where the guarantees end.

---

## Key Quotes

> "An SQLite database is highly resistant to corruption. If an application crash, or an operating-system crash, or even a power failure occurs in the middle of a transaction, the partially written transaction should be automatically rolled back the next time the database file is accessed."

The opening frame is the most important sentence in the document. Everything that follows is the asterisk. SQLite makes a strong claim — automatic recovery from crash — and then spends thousands of words documenting the boundary conditions where that claim fails. This is how you earn credibility: state your guarantee, then enumerate every way it can break.

> "Unfortunately, most consumer-grade mass storage devices lie about syncing. Disk drives will report that content is safely on persistent media as soon as it reaches the track buffer and before actually being written to oxide."

The most quotable sentence in the document, and the most damning. SQLite does everything right — fsync, write-ahead logging, careful ordering — and the hardware lies to it. The authors note this is understandable (manufacturers optimize for benchmark numbers) but the consequence is brutal: no software-level correctness can compensate for a storage layer that reports completion before completion.

> "USB flash memory sticks seem to be especially pernicious liars regarding sync requests. One can easily see this by committing a large transaction to an SQLite database on a USB memory stick. The COMMIT command will return relatively quickly... and yet the LED on the end of the memory stick will continue flashing for several more seconds."

A concrete, reproducible demonstration. The LED is the truth-teller; the software API is the liar. This is the pattern throughout the document: there is always a way to observe the real state if you know where to look. The skill is knowing where.

> "Any thread in the same process with a file descriptor that is holding a POSIX advisory lock can override that lock using a different file descriptor. One particularly pernicious problem is that the close() system call will cancel all POSIX advisory locks on the same file for all threads and all file descriptors in the process."

POSIX advisory locking's design flaw, stated plainly. This isn't an SQLite bug — it's a Unix design decision that made sense in a single-threaded world and became a footgun in a multi-threaded one. The document names it without blaming the OS designers, which is characteristic of the tone throughout: these are facts about the environment, not indictments.

> "No software is 100% perfect. There have been a few historical bugs in SQLite (now fixed) that could cause database corruption. And there may be yet a few more that remain undiscovered."

The pivot from "here's how the environment fails you" to "here's how we failed you." The eight historical bugs documented in section 8 — each with version ranges, root causes, and fix dates — are a confession that builds more trust than a claim of perfection ever could.

---

## Key Themes

#database #reliability #corruption #filesystems #testing #sqlite

---

## Critical Analysis

**This is one of the best pieces of technical documentation on the internet, and it works by inversion.** Most database documentation tells you what the system guarantees. This one tells you what it *doesn't* guarantee — and in doing so, defines the guarantee perimeter more precisely than any affirmative statement could. "SQLite is resistant to corruption" is vague. "SQLite can be corrupted by any of the following eight categories of failure, and here are the specific mechanisms, historical examples, and version ranges for each" is precise.

**The document is secretly a taxonomy of the trust boundary.** Every corruption category maps to a layer in the stack: application code (memory corruption, fd reuse, fork), OS kernel (locking, LinuxThreads, mmap bugs), filesystem (broken lock implementations, filesystem corruption), hardware (lying disk controllers, fake USB sticks, non-powersafe flash), and SQLite itself (historical bugs). The boundary between "SQLite's responsibility" and "the environment's responsibility" is the document's real subject. If you read it as a database engineer, you learn about corruption. If you read it as a systems thinker, you learn about where guarantees die.

**The POSIX advisory locking section (2.2) is a masterclass in documenting platform pathologies.** The close()-cancels-all-locks behavior is an accident of history that every Unix database has to work around, and the SQLite team explains it clearly enough that a developer who's never touched a file lock can understand why their backup script is corrupting their database. The concrete scenario — "a third thread does open(), read(), close() on the database file" — turns an abstract OS design flaw into a recognizable production bug.

**Section 8 (Bugs in SQLite) is the trust move.** Listing your own historical bugs, with version numbers, root causes, and fix dates, is the opposite of what most software projects do. It says: we have made mistakes, here is exactly what they were, here is when we fixed them, and here is how obscure they were. The four-year window (2009-2013) with exactly eight corruption bugs — all obscure, most discovered through internal testing, none observed in the wild — makes a quantitative case for reliability that a qualitative claim never could.

**The hardware section (3.1, 4.1, 4.2) is an accidental argument for WAL mode.** The document repeatedly notes that WAL mode is more forgiving of the failures it describes: out-of-order writes during COMMIT cause durability loss but not corruption; sync failures only matter during checkpoint, not during normal operation. If you read between the lines, the practical recommendation is clear: use WAL mode unless you have a specific reason not to, because it shrinks the corruption surface area. The document doesn't state this as a thesis — it lets the evidence accumulate until the conclusion is obvious.

**What's conspicuously absent: recovery guidance.** The document is pure diagnosis — here are the ways things break — with no corresponding treatment section. There's no "what to do if your database is corrupt" counterpart. This is consistent with the SQLite philosophy (corruption should never happen; if it does, restore from backup), but it leaves a gap. For a document this comprehensive about failure modes, the absence of a "now what" section is noticeable.

**Comparison to the wiki's database landscape:** This page is the failure-mode companion to [[ARIES — Write-Ahead Logging Recovery]], which defines the recovery algorithm SQLite uses. Where ARIES explains how recovery *works*, this page explains when it *doesn't*. It's also the shadow side of [[SQLite is All You Need for Durable Workflows]] — that page argues SQLite is sufficient for agent workflow state; this page catalogs every reason that claim has an asterisk. For the agent infrastructure conversation, this document is the due diligence checklist: if you're building on SQLite, you need to understand which of these failure modes apply to your deployment and whether WAL mode + Litestream covers them. (Spoiler: most of the filesystem and hardware failures apply regardless.)

---

*Sources: [[raw/how-to-corrupt-sqlite]]*
*Last updated: 2026-07-18*
