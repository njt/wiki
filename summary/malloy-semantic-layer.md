---
url: https://github.com/malloydata/malloy
title: Malloy — A Semantic Modeling and Query Language
author: Malloy Contributors
date_fetched: 2026-07-18
date_published: 2021
topics:
  - databases-and-data
---

# Malloy (Précis)

Malloy is an open-source semantic modeling and query language that compiles to SQL. Built in TypeScript as a ~93K-line monorepo, it supports BigQuery, Snowflake, DuckDB, PostgreSQL, MySQL, Trino, Presto, and Databricks as execution engines. The core innovation is a **two-phase compilation architecture**: Malloy source → Translator → serializable IR → Compiler → dialect-specific SQL. The IR is a JSON-serializable representation of the entire semantic model, enabling caching, transmission, and multi-dialect compilation from a single parse.

Key technical achievements: **symmetric aggregates** that correctly handle joined data without double-counting (path-prefixed expressions like `line_items.amount.sum()` specify the aggregation grain); a **pipeline query model** where every query transforms a source into a new source, enabling composable analytics; a **trait-based dialect system** with ~40 boolean flags per database adapter; a **Solid.js + Vega rendering layer** with a plugin system for custom visualizations; and a **tag/annotation system** that serves as both user-facing renderer hints and a compiler metadata side-channel.

The project was originally developed at Google by a team that included the creator of Looker (Lloyd Tabb) and represents a language-first alternative to SaaS semantic layers.
