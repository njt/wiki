---
url: https://blog.cloudflare.com/search-for-agents/
title: "Introducing Search for Agents"
author: Cloudflare (blog team)
date_fetched: 2026-11-26
date_published: 2026-11 (November 2026)
topics:
  - mcp-and-tool-protocols
  - ai-infrastructure-and-hardware
---

Cloudflare launches **Search for Agents**: a dashboard toggle that, per zone, auto-creates an AI Search instance, crawls and indexes the site's content, and exposes a search API under the site's own domain at `/.well-known/search`. It extends the same network-level philosophy as Markdown for Agents (HTML→Markdown conversion via content negotiation).

The clever half is **discoverability**: every HTML page gets `<meta name="agent:instructions">`, `agent:endpoint`, `agent:method`, `agent:documentation` tags in the head, and Markdown-for-Agents-enabled zones get the same block as YAML front matter — so any agent landing on the site learns that a keyword or natural-language search API exists and how to call it, rather than crawling page by page.

The endpoint returns **retrieved content in Markdown, not a generated answer** — the explicit rationale is that the calling agent already has a model and a conversation; it needs the site's facts, not another model's attempt to answer on its behalf. Besides GET, the API natively supports the new IETF **HTTP QUERY method** (RFC 10008, co-authored by Cloudflare's James Snell) for sending search criteria safely and idempotently in the request body. An OpenCode session demo shows the agent discovering the endpoint from the homepage metadata and correctly answering a docs question seconds later.

It's still a standard AI Search instance underneath — reranking, caching, REST/MCP endpoints, Workers bindings — priced per AI Search usage, and gated by the zone's own bot management, AI Crawl Control, and WAF settings. Cloudflare frames it as part of their broader Agent Readiness push: making sites agent-optimised, from authentication to content format.
