---
url: https://maguyva.ai/
title: Maguyva — Remote MCP Server for AI Coding Agents
author: UT International PTE. LTD.
date_fetched: 2026-07-21
date_published: unknown
---

# Maguyva

Maguyva is a remote MCP (Model Context Protocol) server that gives AI coding agents grounded, pre-indexed codebase context before they start editing. It acts as a "map of the codebase" so agents can "edit like it's been here before." The company describes it as "deterministic software — same question, same answer — not another AI."

Built by two humans and a fleet of 33 AI agents. Company: UT International PTE. LTD. "No venture capital was consumed in the making of this product."

## Features

- 11 MCP tools integrated into one surface
- 279 languages supported via custom AST queries (Tree-sitter based). "Not one regex pretending it understands Haskell"
- 5 search modalities: semantic, structural, graph, and text retrieval fused into ranked results
- 4 graph views: Dependency, Type, Data-flow, and Control-flow — letting agents see blast radius before editing
- Remote MCP: Works with Claude Code, Claude Desktop, Cursor, VS Code, Windsurf, Codex CLI, Gemini CLI, GitHub Copilot CLI, Cline, Roo Code, Goosed, Continue, JetBrains, Zed, Trae, ChatGPT, and "whatever ships next week"
- Cloud pipelines handle indexing, parsing, and ranking — nothing runs locally
- Auto-syncs on push to GitHub; incremental parsing for changed files
- Graph-ranked symbol importance (e.g., main implementation scored 3,400 vs. test stubs scored 49)

## How It Works

1. Connect GitHub — authorize, pick repos, walk away
2. Cloud builds the map — ranked map of every symbol, dependency, and import chain (2–15 min initial sync)
3. Plug in any MCP client — drop API key into config
4. Agent verifies first — queries before touching code, reducing hallucinated edits

## Pricing

Charges per workspace, not per user/agent. "The cost scales with how much code you index — not how many humans or agents query it." Free plan: daily sync. Paid plans: hourly graph rebuilds. No credit card required to start. Explicitly rejects "value-capture driven pricing from an era when every user had a pulse."

## Not Just for Coding

Marketing agents and other non-coding MCP workflows can query the codebase for product docs, RFCs, blog posts — "stops making things up about your product."

## Tech Stack

PostgreSQL, MCP, Cloudflare, Tree-sitter, FastMCP, Supabase, Voyage AI

## Stats

- 279+ languages
- 30.9K+ commits
- 33 AI agents on the fleet
- 4.6M+ lines of code indexed

## Reviews

Features simulated reviews from AI agents (Claude Code, Cursor, Codex CLI, Gemini CLI, Windsurf, GitHub Copilot, Cline) and human reviews with 5/5 ratings, plus one 3/5 from GPT-4o: "I preferred when I could just make up function names and nobody noticed."
