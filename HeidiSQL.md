# HeidiSQL

A free, open-source database management GUI that has been actively maintained since 2002. Supports seven database engines (MariaDB, MySQL, MSSQL, PostgreSQL, SQLite, Interbase, Firebird) across Windows, Linux, and now experimentally on macOS. Built in Delphi (Windows) and Lazarus/FreePascal (cross-platform). Created and maintained primarily by one person — Ansgar Becker — for 24 years and counting.

---

## Key Quotes

> "HeidiSQL is a free and powerful client for MariaDB, MySQL, Microsoft SQL Server, PostgreSQL, SQLite, Interbase and Firebird."

The pitch in one sentence. Seven database engines is ambitious coverage for any tool, let alone a one-person open-source project. Most database GUIs pick a lane — Postgres tools, MySQL tools — and stay there. HeidiSQL's multi-engine ambition is what makes it notable: a single interface across heterogeneous database environments.

> "Free for everyone, OpenSource."

This sits front and center on the site. Not freemium. Not "community edition" with enterprise upsell. Free, period. In an era where every developer tool ships a pricing page, a 24-year-old project that never added one is a statement of values, not a missing feature.

> Windows version built with Delphi 12.3. Linux and macOS releases compiled using Lazarus v4.4 and FreePascal.

The cross-compilation story is worth noting. Delphi for Windows, Lazarus for cross-platform — same Pascal codebase, two different compiler toolchains targeting different OSes. This is the pragmatic path for a Delphi-native project going cross-platform without a rewrite. The v13 preview shares the same codebase across all platforms, confirming this isn't a fork but a unified tree.

## Key Themes

#tool #database #open-source #delphi

### The One-Person Power Tool

HeidiSQL is a case study in sustained solo open-source development. Ansgar Becker has maintained this project for 24 years, shipping regular releases (four minor versions in the last six months alone). The feature surface is genuinely deep: SSH tunneling, SSL, visual table designers, stored routine editors, cross-database text search, batch table operations, server-to-server exports. This isn't a minimalist tool that avoids complexity — it's a comprehensive one, maintained by someone who clearly uses it themselves.

### Multi-Engine Database Abstraction

Most database GUIs are single-engine (pgAdmin, MySQL Workbench, SSMS). HeidiSQL's support for seven engines puts it in a rare category alongside DBeaver and DataGrip — but unlike those, it's free and open source without a commercial edition. The tradeoff: engine-specific features lag. The macOS preview lacks MSSQL support, and PostgreSQL features like ENUMs only arrived in v12.17 (April 2026). Multi-engine breadth means no single engine gets first-class treatment.

### The Delphi Long Tail

HeidiSQL is one of the highest-profile active Delphi applications still in development. Alongside tools like [[QR Generator (delphi.tools)]], it demonstrates that Delphi — often written off as legacy — remains a viable platform for shipping real desktop software. The Lazarus/FreePascal cross-compilation story makes it particularly interesting: it's a Pascal codebase targeting Windows, Linux, and macOS from two different compilers, which is more than most Electron apps manage from one.

### Desktop Software as Counter-Current

In a world where everything is moving to the browser and to SaaS, HeidiSQL is stubbornly a desktop application. You download it, you install it, it connects directly to your databases — no cloud proxy, no web interface, no subscription. This is the same philosophy behind [[Simplicity in the Age of AI-Assisted]]: the tool does its job without an account, without a backend, without a pricing page. The SSH tunnel support means you can connect to remote databases securely without exposing them, which is the desktop app's answer to "but my database is behind a firewall."

## Critical Analysis

**What works:** The feature density is remarkable for a solo-maintained project. SSH tunneling, SSL, visual editors for every database object type, cross-database search, server-to-server export, batch operations — this covers the workflow of a working DBA or developer who lives in databases. The recent release cadence (monthly-ish since late 2025) suggests active maintenance, not abandonware. The Linux support via Lazarus is a genuine achievement given the Delphi heritage.

**What's sharp:** Seven database engines is too many for a one-person project to support equally. The MSSQL and PostgreSQL support will always be shallower than MySQL/MariaDB — that's the origin story baked into the architecture. The macOS version is described as "very early preview" and lacks MSSQL — that's honest labeling, but it means macOS users are years from parity. The UI is functional but distinctly Windows-native in aesthetics; it won't win design awards and on Linux/macOS it will feel like a port (because it is).

**Why it matters:** HeidiSQL is evidence that the "just use DBeaver / DataGrip / TablePlus" advice isn't the whole story. There's a class of developer who wants a native Windows tool they've used for a decade, that connects directly without a middleman, and that has exactly the features they need and nothing they don't. HeidiSQL serves that user perfectly, and the fact that it's still getting monthly releases in 2026 means that user base is real and paying (via donations, presumably).

**The tension:** The cross-platform push to Linux and macOS via Lazarus is the right strategic move — the Windows-only desktop tool market is shrinking — but it risks diluting the project's core strength, which is being a deeply polished Windows-native MySQL/MariaDB client. Cross-platform always costs something, and for a solo maintainer, that cost is attention and bug-fixing bandwidth.

## Cross-Links

- [[Databases and Data]] — synthesis page for database tooling and data engineering
- [[sql-crack]] — VS Code extension for SQL query visualization; complementary tool for the editor, while HeidiSQL owns the database administration side
- [[Shaper]] — SQL-first dashboards via DuckDB; HeidiSQL is the operational counterpart (admin, query, edit) to Shaper's analytical role
- [[QR Generator (delphi.tools)]] — another active Delphi-based web tool; evidence that the Delphi ecosystem still produces polished, single-purpose tools
- [[Simplicity in the Age of AI-Assisted]] — HeidiSQL is simplicity by never accumulating complexity in the first place, not by demolition
- [[Dolt]] — Git-for-databases; both are database tools that take a "files first, no cloud" approach
- [[AliSQL]] — Alibaba's MySQL fork; HeidiSQL could connect to it just like plain MySQL — the tooling layer is engine-compatible
- [[Dapper Performance Trap]] — the kind of database-level performance issue a tool like HeidiSQL helps you diagnose by making query behavior visible
- [[Materialized Views Are Obviously Useful]] — databases should handle derived data; HeidiSQL is the interface for managing those database objects
- [[Software Engineering Craft]] — fundamentals of tool design; HeidiSQL as a case study in single-purpose, sustained-quality desktop software

---
*Sources: [[raw/heidisql]]*
*Last updated: 2026-05-31*
