---
url: https://mirekrusin.com/cave/
title: "CAVE — Compressed Atomic Verb Expressions"
author: Mirek Rusin
date_fetched: 2026-11-03
date_published: 2026 (undated; CAVE v0.39.1)
topics:
  - coding-agents-and-frameworks
  - agent-memory-and-context
---

CAVE (Compressed Atomic Verb Expressions) is a small plain-text language for writing down what you know — one claim per line, `subject VERB object` — plus a CLI that stores those claims in a single local file (SQLite under the hood), remembers how beliefs change, and answers questions across the whole graph. The landing page is also a full tutorial: a monorepo example (which package uses which, blast radius via `USES+`, custom verbs, CSV import, rules that derive exposure to a CVE) and a market-watchlist example where Claude Code reads news articles and records claims with line-level source spans and confidences.

The design ideas are the substance: claims are **append-only** — nothing is ever overwritten, retraction is a new line at `@ 0%`, and `--as-of` reconstructs what the store believed on an earlier date. Contradictions coexist (different sources are different series; `--resolve` ranks by reliability, confidence, recency). Rules are `premises => conclusion` lines whose derived claims carry `BECAUSE`/`VIA` lineage back to their premises. Confidence is first-class (`@ 60%`), and derived confidence multiplies premises. "Actions" are governed writes — the only way a decision gets recorded — declared as named operations that become `act_<name>` MCP tools, so an agent gets a vocabulary of allowed writes instead of free-form appends.

The agent surface is a deliberate centerpiece: `cave mcp` serves the store to any MCP client, `cave ingest "news/*.md" --agent "claude -p"` lets an LLM read documents and emit claims with `@src:file#L3-L7` spans pointing at the exact sentences, and the spec itself ships as four Claude Code skills. Everything is local-first (one file, browser playground, CC0), and reports cite the exact claims behind every query result. Nothing is permanent-friendly to secrets: there is no claim-level delete.
