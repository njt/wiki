---
title: "Mole"
url: https://github.com/lajosdeme/mole
author: Lajos Deme
date_fetched: 2026-08-22
date_published: 2026-08-13
topics:
  - agent-architecture
---

# Mole: Enforced-Budget Deep Research Agent

Mole is a deep-research agent written in Go (~82,600 lines across 254 files) that decomposes a question, searches the web and academic databases, mines claims, verifies each against its source, builds a claim graph, and writes a cited answer — all under a hard cost ceiling enforced in the SQLite schema itself. It runs as two static binaries (`mole`, `mole-mcp`) on your own machine with your own API keys, and speaks MCP.

## What it does

Ask a question; mole decomposes it, searches, reads sources, extracts claims, checks each claim against the text it came from, looks for contradictions between them, and writes an answer with citations. Every model call is reserved against a budget before it happens and settled after, so the ceiling you set is the ceiling it hits.

## Three differentiators

- **The budget is enforced, not estimated.** Every call is reserved before it is made and settled after, against a ledger with non-negative `CHECK` constraints in the database schema. `--usd 0.50` means the run stops at fifty cents; measured overshoot across the test corpus is 0%.
- **Every claim carries a quote, checked against the source.** A claim whose quote does not appear verbatim in the page it was mined from is discarded at extraction, before it can reach an answer.
- **Local data stays local.** Point mole at a CSV or a folder and it analyses it without the contents leaving the machine: the model chooses a hypothesis template and column names, mole renders and runs the SQL, and only aggregates — counts, means, test results, buckets covering at least five records — are allowed back. `mole crossings` shows what left.

## Modes

- **Autonomous** — `mole research "..." --usd 0.50` runs the full pipeline with your keys. Also supports follow-ups (`mole ask`), dataset extraction (`--mode dataset`), and local-data analysis (`--actors local_compute`).
- **Toolkit mode** — `mole serve --toolkit` exposes fourteen `mole.*` tools so a coding agent's model does the reasoning while mole supplies the deterministic half (search/fetch SSRF guard and rate limits, quote verification, the aggregation gate). Built for people inside a Claude Code or Qwen Code subscription whose tokens are already paid for.

## Architecture in one line

Planner (decompose, replan) → executor (one lead per worker, reserved and settled) → actor (search → fetch → extract → chunk → mine → quote-check) → verifier (pair related claims, adjudicate contradictions, derive confidence from the graph) → output (synthesize from surviving claims, with citations assigned mechanically).
