---
title: "HeidiSQL"
url: https://www.heidisql.com/
author: Ansgar Becker
date_fetched: 2026-05-31
date_published: 2002 (initial release)
section: "Developer Tools"
topics:
  - developer-tools
---

# HeidiSQL: Free Open-Source Database Management Tool

## Overview

HeidiSQL is a free, open-source database management tool created by **Ansgar Becker** in 2002. It provides an intuitive graphical interface for working with multiple database systems. The tool remains actively maintained and widely used, especially for MariaDB and MySQL.

The name "Heidi" refers to the application itself; it allows users to connect to databases and "edit data and structures in these databases."

## Supported Database Systems

HeidiSQL works with seven database platforms:
- MariaDB
- MySQL
- Microsoft SQL (MSSQL)
- PostgreSQL
- SQLite
- Interbase
- Firebird

## Key Features

- **Free and open source** for everyone
- **Multi-server connections** in a single window
- **SSH tunnel and SSL support** for secure connections
- **Visual database object editors** for tables, views, stored routines, triggers, and scheduled events
- **SQL export** with compression and clipboard support, plus direct server-to-server exports
- **User privilege management**
- **Text file import**
- **Data export** as CSV, HTML, XML, SQL, LaTeX, Wiki Markup, and PHP Array
- **Grid-based data browsing and editing**
- **Bulk table operations** (move to another db, change engine, collation, etc.)
- **Batch insertion** of ASCII or binary files into tables
- **SQL query editor** with customizable syntax highlighting and code completion
- **SQL reformatting** for disordered queries
- **Client process monitoring and killing**
- **Cross-database text search** across all tables on a server
- **Batch optimization and repair** of tables
- Ability to launch a parallel `mysql.exe` command line window using current connection settings

## Architecture & Technical Details

The software is built using different compilers depending on the target platform:

- **Windows version** — built with **Delphi 12.3** (Embarcadero)
- **Linux and macOS releases** — compiled using **Lazarus v4.4** and FreePascal

A **preview of v13 for Windows** is available, sharing the same codebase as the Linux version. A macOS app bundle for **arm64** has also been released as a very early preview, including code signing certificates, though it lacks MSSQL support at this stage.

## Recent Version History

| Version | Date | Highlights |
|---|---|---|
| **12.17** | 2026-04-12 | User role management, ENUMs on PostgreSQL, invisible indexes |
| **12.16** | 2026-03-10 | Reverse foreign keys, display main menu, row count on MSSQL and SQLite |
| **12.15** | 2026-01-30 | Windows, Linux, and macOS release; bugfixes and enhancements |
| **12.14** | 2025-12-11 | Windows and Linux release; first macOS app bundle preview followed on Dec 16 |

## License

The site describes HeidiSQL as "Free for everyone, OpenSource." It operates under an open-source license.

## Community & Support

Additional resources include a **forum** with active discussions, a **bug tracker** on GitHub (organized by milestones, bug reports, feature requests, and enhancement requests), **screenshots**, and a **themes** showcase. The site also lists a **donation** page with a donor list.

## Author & Imprint

**Ansgar Becker** is the creator. The imprint lists:
> Ansgar Becker, Falkenstr. 10, 48485 Neuenkirchen, Germany

The website is hosted by Manitu, and the project has official presences on **Instagram** and **Bluesky**.
