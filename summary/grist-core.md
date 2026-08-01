---
url: https://github.com/gristlabs/grist-core
title: "Grist"
author: Grist Labs Inc.
date_fetched: 2026-07-29
date_published: 2014
---

An architectural walkthrough of the Grist open-source codebase — a relational spreadsheet that combines the flexibility of a spreadsheet with the robustness of a database. The `grist-core` repository (~233K lines TypeScript, ~45K lines Python) is Apache 2.0 licensed, with contributions from the French government's ANCT and DINUM alongside Grist Labs (NYC).

The deployment topology splits into **Home Servers** (user-facing requests, Postgres home DB) and **Doc Workers** (per-document handlers that spawn sandboxed Python interpreters), though both run the same `FlexServer.ts` codebase. A Grist document is a single `.grist` file — a SQLite database with `_grist_*` metadata tables managed by Python and `_gristsys_*` system tables managed by Node. The request flow goes: browser user action → WebSocket → Doc Worker (Node) → Python data engine via stdin/stdout pipes → Doc Actions back to Node → SQLite writes → broadcast to all connected browsers.

The most distinctive architectural choice is **dynamic code generation**: the Python engine compiles the document schema into a Python module at runtime via `exec()`, turning each table into a class and each formula column into a method. Formula evaluation uses a cooperative exception-driven scheduler — when a formula dependency hasn't been computed yet, an `OrderError` is thrown and the requester is added to a wait list, handling dynamic dependencies that a static topological sort cannot. Dependency tracking is relation-based, specifying *which rows* need recalculation (identity, reference, composed relations), not just which columns.

The **two-level action system** (User Actions → Doc Actions) acts like a compiler pipeline: high-level user intent is translated by Python into atomic operations, then by Node into SQL. This separation enables straightforward undo (inverse Doc Actions), access-control filtering, and per-client action subsets. **Granular access control** checks actions twice — before the engine processes them and after it produces results — correctly handling cases where formula recalculation touches columns a user shouldn't see.

Design trade-offs: optimised for formula expressiveness (full Python, not a restricted subset), portability (self-contained SQLite files), and deployment simplicity (single Docker container); sacrificed memory efficiency (entire document loaded in Python and in each browser tab) and concurrent write performance (SQLite single-writer, one Doc Worker per document). The client codebase carries legacy from multiple UI eras (Knockout.js, GrainJS, newer components) and `GridView.js` is noted as one of the oldest and largest pieces of the codebase.

Compared to Airtable, Grist is self-hostable with more powerful Python formulas. Compared to Google Sheets/Excel, it enforces column types and has proper relational features (References, summary tables). Against other open-source alternatives (NocoDB, Baserow), it differentiates on Python formulas, the dependency-graph architecture, and the self-contained file format.
