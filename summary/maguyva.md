---
url: https://maguyva.ai/
title: "Maguyva — Remote MCP Server for AI Coding Agents"
author: UT International PTE. LTD.
date_fetched: 2026-07-21
topics:
  - agent-architecture
---

Maguyva is a remote MCP server that gives AI coding agents pre-indexed codebase
context — a "map of the codebase" so agents can navigate unfamiliar repos
without hallucinating functions and imports. It bills itself as deterministic
software ("same question, same answer — not another AI"), built by two humans
and a fleet of 33 AI agents, without venture capital.

It offers 11 MCP tools supporting 279 languages via Tree-sitter–based AST
queries. Five search modalities (semantic, structural, graph, text, fuzzy) fuse
into ranked results. Four graph views — dependency, type, data-flow, and
control-flow — let agents assess blast radius before editing. It works as a
remote MCP server with Claude Code, Claude Desktop, Cursor, VS Code, Windsurf,
Gemini CLI, GitHub Copilot CLI, and others.

Setup connects GitHub repos; the cloud pipeline indexes symbols, dependencies,
and import chains (2–15 minutes initially), incrementally re-parsing on push.
Pricing is per workspace rather than per user or agent, with a free daily-sync
tier and paid hourly rebuilds.

The landing page features simulated and human reviews (mostly 5/5), tech-stack
notes (PostgreSQL, Cloudflare, Tree-sitter, Supabase, Voyage AI), and stats: 30K+
commits, 4.6M+ lines indexed.
