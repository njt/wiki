# Search for Agents (Cloudflare)

Cloudflare's Search for Agents turns "make your site searchable by AI agents" into a zone-level toggle: auto-created AI Search index, a `/.well-known/search` endpoint on your own domain, and per-page metadata (`agent:instructions`, `agent:endpoint`) so any agent landing on the site discovers the API without being told. It returns Markdown retrieval, not generated answers, and natively supports the new IETF HTTP QUERY method.

---

## What it does

One toggle creates an AI Search instance for a zone, starts crawling, and exposes keyword and natural-language search at `/.well-known/search?q=...`, subject to the site's own bot management, AI Crawl Control and WAF. Discoverability is the interesting design decision: every HTML response carries meta tags advertising the endpoint, and Markdown-for-Agents zones get the same advertisement as YAML front matter. An agent "picks up these instructions and knows what to do next" — in their OpenCode demo the agent finds the endpoint on the homepage and reuses it throughout the session.

## Key quotes

> "Note that this search endpoint returns retrieved content, in Markdown, not a generated answer. The agent already has an AI model and a conversation with the user; all it needs is the relevant information from your site, not another AI model's attempt to answer on its behalf."

A refreshingly correct division of labour. Most "AI search" products wrap retrieval in a generated answer and thereby duplicate — and degrade — the calling agent's own reasoning. Cloudflare keeps the model where it already is and ships only facts.

> "The idea follows naturally from Markdown for Agents... Search for Agents leverages the power of AI Search by automatically creating an index for your zone and exposing APIs that any AI agent can discover and use to search your website, under your domain, all done at the network level."

The pattern here is network-level affordances: the infrastructure layer silently reshapes what agents see and can do, site owners opt in with a toggle. That's a genuinely scalable answer to the discovery problem — no central registry, no MCP handshake, just metadata on every page.

> "The new HTTP QUERY method is an IETF Proposed Standard... A QUERY allows a client to send search criteria in a request body safely and idempotently without modifying server state."

Getting a real RFC (10008) into an agent-facing API on day one is quietly significant: it's the first widely deployed case of agent traffic pulling new HTTP semantics into the mainstream.

## Key themes

#tool #pattern #concept

## Analysis

This is a small product announcement carrying a big architectural bet: that **agent-facing web infrastructure** — search-as-utility, Markdown responses, self-describing endpoints — becomes a normal part of running a website, the way SEO and sitemaps did. The design choices are all defensible: retrieval-not-answers avoids double-modelling; per-domain APIs keep the site owner's access controls (WAF, crawl control) in the loop; metadata-in-every-page solves discovery without a registry.

The honest caveat: "search your site" assumes the site's content is the answer, and an index built by the same party that controls bot management means the site owner fully controls what agents can see — which is the point, but also means Search for Agents is an *optimisation within* a controlled perimeter, not an open web search. It also inherits the usual tension of agent-optimised surfaces: humans never see the `agent:*` meta tags, so this is a parallel interface maintained purely for machine readers.

Compared with MCP-first approaches, this is deliberately dumber and more universal — a plain GET with query parameters that any agent with HTTP access can use, no protocol negotiation required. That restraint is likely why it will actually get used.

## Relations

- [[Your App's Frontend UI Is Now Optional]] — Seroter's argument that agent visitors will outnumber humans gets concrete infrastructure here: Cloudflare shipping a first-class, network-level agent interface that no human ever sees.
- [[10 Principles for Agent-Native CLIs]] — the same design ethic at the API layer: advertise yourself, return machine-friendly formats, make the agent's next step obvious from the response.
- [[AI-Ready APIs — Postman AWS Competency]] — Matt Gray's claim that API quality, not model capability, is the agentic bottleneck is strengthened by Cloudflare building exactly this: an API designed so agents can discover and use it without hand-holding.
- [[The Agent Access Model]] — Cloudflare's own research on task-scoped agent access; Search for Agents is the site-owner side of the same story, with WAF and AI Crawl Control deciding what agents may reach.

---
*Sources: [[raw/search-for-agents]], [[summary/search-for-agents]]*
*Last updated: 2026-11-26*
