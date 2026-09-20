---
url: https://shiftmag.dev/mcps-arent-apis-stop-treating-them-like-one-11420/
title: "MCPs Aren't APIs – Stop Treating Them Like One"
date_fetched: 2026-09-20
topics:
  - mcp-and-tool-protocols
  - agent-memory-and-context
---

A practitioner's cost-focused guide to designing MCP servers, arguing that the dominant failure mode is treating MCP as a 1:1 wrapper over a REST API. Every enabled MCP server injects its tool definitions — names, parameters, descriptions, enums — into the context window on every request, before the prompt is even processed. The author reports a real case where adding just two MCP servers cost 13,000 tokens per request, roughly 16–17 A4 pages of overhead.

The prescriptions follow from that economics: keep servers to 10–15 tools (fewer if possible), split monolithic servers by domain so users only load what they need, and design tools as capabilities rather than endpoints — one `manage_user` tool that aggregates several API calls instead of seven CRUD wrappers. Output matters as much as input: raw API responses burn tokens on JSON scaffolding, so servers should strip metadata, summarise large fields, use MCP resources for big static datasets, and force filters or pagination on queries that could return thousands of results.

The article closes with configuration hygiene: register only general-purpose servers at the user level, push everything else to project config, disable anything not actively in use, and audit regularly — every enabled server taxes every request.
