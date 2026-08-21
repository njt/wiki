---
url: https://acadia.engineering/blog/rethinking-database-programming
title: Rethinking Database Programming
author: Evan Czaplicki
date: 2026-08-18
date_fetched: 2026-08-21
---

Evan Czaplicki (creator of Elm) announces the public alpha of Acadia, a language that "brings the benefits of 'languages like Elm' to SQL." Four goals: store precise custom types directly in the database (no hand-rolled binary layouts, JSON, or nullable-column encodings); compiler-verified migrations (no more "body on high alert" when touching a live database); friendly error messages; and end-to-end types shared across client, server, and database.

Tables are defined as typed values carrying an explicit primary key, a row-level security policy, indexes, and constraints. Queries are written as functional pipelines (`filter`/`map`/`select`) that compile to SQL at compile time — Acadia prints the SQL it generates so you can judge its quality. Multi-step writes use a `:=` let-binding inside a transaction that commits only if every step succeeds.

The backstory traces the idea to 2017 Elm server-side-rendering experiments and 2019 feedback from Elm companies that "our major engineering challenges are with our backend code." The design crystallized in 2020 when Czaplicki's partner Tereza pushed back on a SQL-like prototype: "I thought it would look like Elm, with map and filter." Acadia currently supports Elm and Haskell integration, runs on SQLite under the hood with a drop-down-to-SQL escape hatch, and defers window functions and custom aggregates past the MVP.
