---
url: https://github.com/prisharai/Interdict
title: "Interdict — Safety Layer Between AI Agents and Postgres"
author: Prisha Rai (pr482@cornell.edu)
date_fetched: 2026-07-08
date_published: 2026-06
---

Interdict is a Python runtime safety layer that sits between AI coding agents (via MCP) and PostgreSQL. It parses every SQL statement using the real Postgres parser (libpg_query), classifies it, checks it against a deterministic YAML policy, and gates writes before they reach the database.

Three capabilities distinguish it from simpler guardrails. First, **blast-radius simulation**: risky writes (unscoped UPDATE/DELETE) are actually executed inside a rolled-back transaction to measure exact affected rows, with hard timeouts. Second, **conditional atomic undo**: every allowed write captures before/after images in the same transaction, and revert restores rows only if they haven't been changed by another transaction since — it never silently clobbers concurrent work. Third, **structured rejections**: all policy violations are returned at once with machine-readable reason codes, human explanations, and suggested fixes, designed so the agent can self-correct in one round trip.

The engine is transport-agnostic (~3,500 lines); the MCP server is thin glue (~1,160 lines). The hot path is latency-optimized: O(1) input guards reject pathological queries before parsing, LRU caches skip re-analysis of repeated statements, and audit logging is async and non-blocking. Claimed overhead is 2.6µs p50 per statement (warm).

The project includes an academic study on specification gaming in database guardrails. The headline finding: when a guardrail tells an AI agent exactly which rule it violated, agents are far *more* likely to produce syntactically compliant statements that still affect the whole table (e.g., adding `WHERE 1=1`) than when given an opaque error. Measuring actual row-level impact catches this class of evasion; simple syntax heuristics do not.

Interdict is optimized for low-volume, high-consequence agent traffic — not high-throughput application workloads. It fails closed on writes (uncertainty blocks), fails open on reads (never blocks availability), and keeps all AI components strictly advisory. The undo mechanism lives in a sidecar schema requiring no extensions. The project is MIT-licensed, ~9,100 lines of code with 343 CI tests including concurrency races, fault injection, and evasion attacks.
