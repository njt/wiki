---
url: https://claude.com/blog/bringing-mcp-2026-07-28-to-claude
title: Bringing MCP 2026-07-28 to Claude
author: Anthropic
date_fetched: 2026-08-01
date_published: 2026-07-28
---

# Bringing MCP 2026-07-28 to Claude

The fifth spec release of the Model Context Protocol, MCP 2026-07-28, is live today. Support is being rolled out across Claude products.

## What's new in MCP

MCP recently surpassed 400M monthly SDK downloads — a 4x increase this year — and has become the industry standard for connecting AI agents to applications. Three headline changes:

- **Stateless core.** MCP moves from a bidirectional stateful protocol to a request/response model. Servers can now deploy on serverless and edge infrastructure, simplifying server building and scaling.
- **Standardized extensions.** MCP Apps and Tasks now ship under a versioned extensions framework, giving developers a formal path to add capabilities like interactive UIs and long-running work without changing the core protocol.
- **Auth hardening.** Authorization now aligns with production OAuth 2.0 and OIDC deployments, so MCP servers connect to enterprise identity systems like Entra or Okta without workarounds.

## Ecosystem quotes

Companies built on the new spec alongside the MCP community since beta:

- **Figma — Josh Clemm, VP of Engineering:** More builders use the MCP server to bring generated outputs into Figma's canvas. As usage grows, their stateless architecture can scale with it; MCP Apps, Tasks, and Enterprise-Managed Auth help keep design and code in one connected flow.
- **Intuit — Chris Kasten, Chief Architect and SVP:** MCP is "the industry standard for connecting AI agents to tools and data." The stateless protocol core and extensions framework let Intuit build and connect agentic experiences at enterprise scale for its 100 million consumers and businesses.
- **Netlify — Sean Roberts, VP of Applied AI:** "The stateless core in the 2026-07-28 spec makes MCP a first-class HTTP workload with no session management to work around." Customers wanted MCPs on Netlify to be as simple as the rest of the platform.
- **PostHog — Paul D'Ambra, Product Engineer:** A stateless protocol makes it easier to scale their own service and add analytics for customers' MCP servers — showing how tools are used and which tools users want but are missing.
- **Xero — Andrew Goodman, VP of AI:** The stateless core reduces the complexity they manage, so they can ship more features faster and at scale.
- **Zoom — Ross Mayfield, Head of Product for AI Platform:** Zoom built MCP servers that securely bring meeting intelligence into AI platforms like Claude. The new spec makes it far easier to deploy and scale MCP servers on standard HTTP infrastructure.

A note points to the MCP 2026-07-28 release announcement at blog.modelcontextprotocol.io for full spec details.

## Advancing MCP in Claude

Claude now lists over 950 MCP servers in the connectors directory, used by millions of people daily. Features shipped this year:

- **MCP Apps:** Let servers render interactive UI directly in the conversation, so users can see what a connector is doing and work with it inline without switching tabs.
- **Enterprise-managed auth:** Admins provision MCP connectors organization-wide through their identity provider. "Admins authorize a connector once, users inherit access through their existing IdP groups" — zero-touch setup on first login.
- **Observability for developers:** Published connectors get a dashboard showing performance across Claude product surfaces, enabling tracking of adoption, error diagnosis, latency analysis, and usage breakdown by product.
- **MCP tunnels (research preview):** Connect Claude to MCP servers inside a private network without exposing them publicly — no inbound firewall rules, no public endpoints, no IP allowlisting on the origin.

The article states the stateless core, standardized extensions, and hardened auth will help developers bring more applications to Claude with a lower-friction, more consistent end-user experience, and that Anthropic will continue investing in MCP as an open standard.

## Getting started

Readers are directed to explore the spec and SDKs at modelcontextprotocol.io. Support is rolling out across Claude products soon. Server submission guidance for the connectors directory is linked.
