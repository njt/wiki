---
url: https://claude.com/blog/bringing-mcp-2026-07-28-to-claude
title: "Bringing MCP 2026-07-28 to Claude"
author: Anthropic
date_fetched: 2026-08-01
date_published: 2026-07-28
topics:
  - mcp-and-tool-protocols
  - claude-code
---

Anthropic announces the fifth spec release of the Model Context Protocol
(MCP 2026-07-28) and its rollout across Claude products. MCP has surpassed
400M monthly SDK downloads — a 4x increase this year — and is described as
the industry standard for connecting AI agents to applications.

Three headline changes in the spec: a **stateless core** that replaces the
bidirectional stateful protocol with request/response, enabling serverless and
edge deployment; **standardized extensions** (MCP Apps and Tasks) under a
versioned framework so developers can add interactive UIs and long-running
work without altering the core protocol; and **auth hardening** aligned with
production OAuth 2.0 and OIDC, so servers connect to enterprise identity
systems (Entra, Okta) without workarounds.

Quoted ecosystem support comes from Figma, Intuit, Netlify, PostHog, Xero, and
Zoom — all of whom cite the stateless core as a key enabler for scaling and
simplifying their MCP deployments.

On the Claude side, over 950 MCP servers are listed in the connectors
directory, used by millions daily. Recent Claude-specific features include MCP
Apps (interactive UI in-conversation), enterprise-managed auth (admin
provisions once via IdP, users inherit access), an observability dashboard for
published connector developers, and MCP tunnels (research preview) for
connecting to private-network servers without public exposure.
