---
url: https://github.com/bjonesdataliteracy/datapitfalls
title: "datapitfalls"
author: Ben Jones / Data Literacy, Inc.
date_fetched: 2026-10-20
date_published: 2026
topics:
  - guardrails-and-feedback-loops
  - mcp-and-tool-protocols
---

**datapitfalls** is an open-source tool (npm `datapitfalls`, MIT code / CC BY-NC 4.0 taxonomy, v0.11.0) that detects common blunders in data work — not just chart pixels, but the whole chain from question framing to communication. It is the software companion to Ben Jones's book *Avoiding Data Pitfalls* (Wiley, 2019), whose eight pitfall domains (epistemic errors, technical trespasses, mathematical miscues, statistical slip-ups, analytical aberrations, graphical gaffes, design dangers, biased baseline) form the rule catalog.

The engine (`src/analyze.ts`, ~800 lines) takes four input kinds — code, plain-English descriptions, chart images (Claude Vision), and PDF documents — plus slides and multi-stage "chain" inputs, retrieves the relevant pitfall rules from the taxonomy, and asks Claude (default `claude-sonnet-4-6`) to reason over the artifact *and* the catalog. Findings come back via a forced `report_findings` tool call, and every `rule_id` is validated against the catalog server-side, so the model can only cite real rules. A key conceptual move is the **active vs latent** finding distinction: "active" pitfalls are evident from the artifact alone; "latent" ones are risky patterns whose impact depends on unseen data, reported with an explicit "condition".

Distribution is three-pronged: an npm CLI with `--ci` exit codes for pipelines, a Next.js web app (web/) with per-IP rate limits and PDF/DOCX/PPTX/notebook upload, and — most interestingly — a **keyless, read-only MCP server** (`src/mcp/server.ts`) that ships only the taxonomy (tools: `get_checklist`, `get_pitfall`, `list_pitfalls`, `search_pitfalls`) so Claude itself does the checking with no API key or network. The MCP path deliberately imports only the taxonomy module, never the engine or its SDKs. There is also an eval harness (`evals/`, precision/recall/F1 against golden fixtures) and a dependency-free bridge that turns reports into Semiotic v3 chart annotations — unanchored, because the tool sees findings, not pixel coordinates.
