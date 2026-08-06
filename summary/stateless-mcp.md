---
url: https://simonwillison.net/2026/Jul/31/stateless-mcp/
title: Stateless MCP has recaptured my interest (and inspired mcp-explorer and datasette-mcp)
author: Simon Willison
published: 2026-07-31
---

Simon Willison returns to MCP after the release of the 2026-07-28 stateless specification, arguing that MCP has become a meaningfully safer and simpler way to give agents tool access compared to the prevailing pattern of granting shell and `curl` access. The stateless redesign collapses the old two-request initialize-then-call protocol into a single HTTP request, eliminating server-side session state and making MCP servers deployable as standard web workloads.

Willison built three tools against the new spec in a single week. **mcp-explorer** is a `uvx`-runnable Python CLI for interactively probing MCP servers — listing tools, inspecting their JSON schemas, and calling them with arguments. **datasette-mcp** is his fourth attempt at a Datasette MCP plugin, finally shipping because stateless MCP removed the complexity that killed earlier versions; it exposes `list_databases()`, `get_database_schema()`, and `execute_sql()` as MCP tools. **llm-mcp-client** is an alpha plugin for his LLM CLI tool that allows invoking MCP servers inline: `llm -T 'MCP("https://...")' 'count the notes'`.

The through-line is security. Willison revisits his earlier critique of MCP's prompt injection risks and argues that while MCP has sharp edges, granting agents arbitrary shell and network access — the default for most coding agent tools today — is far harder to secure. MCP tools, being explicitly declared and auditable, make it easier to reason about what an agent can do and what might go wrong. He plans to lean into MCP for sensitive LLM applications going forward.
