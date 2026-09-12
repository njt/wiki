---
title: "Dolt"
url: https://docs.dolthub.com/
date_fetched: 2026-05-14
section: "Databases and Data"
topics:
  - databases-and-data
---

# Dolt: A Version-Controlled SQL Database

## Core Concept

Dolt fundamentally reimagines database management by combining Git's version control capabilities with MySQL's SQL functionality. "Git versions files, Dolt versions tables. It's like Git and MySQL had a baby."

## Key Use Cases

1. **Version Controlled Database** - Users interact with Dolt via standard MySQL connections while accessing version control through specialized system tables, functions, and procedures.

2. **Git for Data** - The CLI mirrors Git's command structure exactly: `dolt add` replaces `git add`, `dolt commit` replaces `git commit`, enabling familiar workflows for data management.

3. **Versioned MySQL Replica** - Dolt deploys as a MySQL-compatible replica, offering version control advantages without requiring migration from existing MySQL infrastructure.

## Ecosystem

- **Hosted Dolt**: Cloud-provisioned database service compatible with standard MySQL clients
- **DoltHub**: Web-based collaborative platform for sharing public databases with pull request workflows
- **DoltLab**: Self-hosted alternative providing DoltHub features on private networks

## Key Features

- MySQL-compatible interface supporting complex queries
- Foreign keys, secondary indexes, triggers, check constraints
- Supports up to twelve-table joins
- Cell-level lineage tracking showing when and how data changed
- System tables and stored procedures expose version control functionality
- `dolt_reset()` allows undoing uncommitted changes
- `dolt_revert()` enables reverting specific commits
- `dolt_undrop()` recovers accidentally dropped databases

## Installation

Single ~103MB executable. Available via Homebrew, pacman, Chocolatey, Docker, MSI, curl, or compiled from Go source.

## Related

- **Doltgres**: PostgreSQL-compatible variant (Beta)
- Powers AI agent memory systems, especially multi-agent workflows
