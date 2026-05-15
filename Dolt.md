# Dolt

A SQL database that you can fork, clone, branch, merge, push, and pull just like a Git repository. MySQL-compatible, with the entire Git workflow applied to tables instead of files. "It's like Git and MySQL had a baby."

---

## Key Quotes

> "Git versions files. Dolt versions tables."

## Key Themes

#database #git-for-data #devtools

Dolt occupies a unique niche: it is simultaneously a fully functional MySQL-compatible database and a version control system. You connect to it with any MySQL client and run SQL, but you can also branch, diff, merge, and revert at the data level. Cell-level lineage tracking means you can ask "when and how did this specific value change?" -- something no traditional database offers.

Three deployment modes cover the spectrum: as a standalone version-controlled database, as a Git-like CLI for data workflows, or as a MySQL replica that adds versioning to existing infrastructure without migration. The ecosystem around it (DoltHub for public data, DoltLab for self-hosted, Hosted Dolt for cloud) mirrors GitHub's model.

The connection to AI is interesting: Dolt is being used for AI agent memory systems, particularly in multi-agent workflows where different agents may need to branch, experiment with, and merge shared state. See [[NornicDB]] for a different approach to the same problem (graph + vector + temporal).

Also related: [[Graft]] for SQLite replication (a different philosophy of "database versioning"), and [[Data Engineering for Large Models]] for the pipeline that feeds LLMs.

## Critical Analysis

Dolt solves a real problem -- data versioning is genuinely painful in traditional databases, and the "oops, wrong UPDATE" scenario is terrifyingly common. The MySQL compatibility story is strong. The weakness is adoption: Dolt requires teams to learn a new database, and the "just add version control to your data" pitch, while compelling, hasn't broken through to mainstream enterprise use yet. The Doltgres (PostgreSQL-compatible) variant in beta suggests the team knows MySQL alone won't be enough. The ~103MB single binary and simple installation are smart -- they remove the "it's too hard to try" objection.

---
*Sources: [[raw/dolt]]*
*Last updated: 2026-05-14*
