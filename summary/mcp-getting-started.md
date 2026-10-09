---
url: https://motherduck.com/docs/getting-started/mcp-getting-started/
title: "Talk to Your Data with AI (MotherDuck MCP Server Getting Started)"
author: MotherDuck
date_fetched: 2026-10-09
date_published: unknown
topics:
  - mcp-and-tool-protocols
  - databases-and-data
---

MotherDuck's getting-started guide for its remote MCP server (`https://api.motherduck.com/mcp`), a hosted endpoint that lets MCP-compatible clients — Claude Desktop, ChatGPT, Cursor, Claude Code — query MotherDuck/DuckDB databases in natural language without writing SQL.

The guide walks a five-minute path: add the MotherDuck connector in Claude Desktop (browser-based OAuth-style authentication), review per-tool permissions (`query`, `list_databases`, `ask_docs_question`), list databases, attach a sample Hacker News dataset via an `md:_share/...` share URL, and then turn analysis into a "Dive" — a persistent, shareable, live-data visualization built conversationally ("add a filter for the last 30 days", "switch to a bar chart"), with each edit saved as a version.

Notable details: the server is fully managed (a local MCP server option exists for local DuckDB files and full control); tool permissions are configurable per tool from the client; and the doc's own next-steps list admits that for coding agents with a terminal, the MotherDuck CLI can beat MCP on token cost — a rare vendor acknowledgment that MCP is not always the right interface.
