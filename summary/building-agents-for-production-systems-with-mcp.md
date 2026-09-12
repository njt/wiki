---
title: "Building Agents for Production Systems with MCP"
url: https://claude.com/blog/building-agents-that-reach-production-systems-with-mcp
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - agent-architecture
---

# Building Agents That Reach Production Systems with MCP - Anthropic Blog (April 2026)

## Three Connection Approaches

1. **Direct API Calls** -- Works for simple integrations but creates M*N problem at scale.
2. **CLIs** -- Lightweight, fast for local. Limited to systems with shells. Doesn't scale to cloud.
3. **MCP** -- Standardized protocol with built-in auth, discovery, rich semantics. One server reaches multiple clients.

## MCP Adoption
SDKs exceeded 300M monthly downloads (up from 100M at start of year). Underpins Claude Cowork and Claude Managed Agents.

## Design Patterns

**1. Build Remote Servers** -- Only config that scales across web, mobile, cloud-hosted agents.

**2. Group Tools Around Intent** -- "Fewer, well-described tools consistently outperform exhaustive API mirrors." Single `create_issue_from_thread` beats four separate tools.

**3. Design for Code Orchestration** -- For services with hundreds of operations, expose thin tool surface accepting code. Cloudflare: ~2,500 endpoints with just 2 tools in ~1,000 tokens.

**4. Ship Rich Semantics** -- MCP Apps (first official extension) return interactive interfaces (charts, forms, dashboards) inline.

**5. Enable User Interaction** -- Elicitation (pause for user input), Form Mode (native forms), URL Mode (browser redirect for OAuth).

**6. Standardized Auth** -- CIMD for OAuth flows. Claude Managed Agents' Vaults handle credential storage.

## Context Efficiency

**Tool Search:** Defer loading tool definitions until needed. Reduces tool-definition tokens by 85%+.

**Programmatic Tool Calling:** Process results in code-execution sandbox, not context window. Reduces tokens ~37% on complex workflows.

## Skills + MCP
MCP provides tool access; skills teach procedural knowledge. Plugin bundling (e.g., data plugin: 10 skills + 8 MCP servers). Protocol extension in development to deliver skills from servers.

## Key Quote
"Agents are only as useful as the systems they can reach."

"As production agents move to the cloud, MCP becomes the critical layer, and it's the one that compounds."
