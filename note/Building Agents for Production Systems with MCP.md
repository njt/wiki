# Building Agents for Production Systems with MCP

Anthropic's guide to connecting AI agents to production systems, making the case for MCP as the standard integration layer. The key framing: "Agents are only as useful as the systems they can reach." Three approaches compared (direct API calls, CLIs, MCP), with MCP winning on scalability -- one remote server reaches every compatible client. MCP SDK downloads hit 300M/month, up from 100M at the start of the year.

---

## Key Quotes

> "Agents are only as useful as the systems they can reach."

> "Group tools around intent, not endpoints -- fewer, well-described tools consistently outperform exhaustive API mirrors."

> "As production agents move to the cloud, MCP becomes the critical layer, and it's the one that compounds."

## Key Themes

#mcp #agent-architecture #production #integration #tool-design #oauth #context-efficiency

The "group tools around intent" pattern is the most actionable design advice. A single `create_issue_from_thread` tool outperforms four separate tools for the individual steps. Cloudflare's approach -- 2,500 endpoints through just 2 tools that accept code -- is the extreme version. This is API design for AI consumers, not human consumers, and the principles are different.

The context efficiency section (Tool Search reduces tool-definition tokens by 85%+, Programmatic Tool Calling reduces tokens by ~37%) provides concrete optimization targets. These aren't marginal improvements -- they're the difference between agents that run out of context and agents that don't.

## Critical Analysis

This is Anthropic's official position paper on MCP, so it's both authoritative and self-interested. They built MCP, they're pitching MCP. The arguments are sound but the alternatives (direct APIs, CLIs) are underweighted. For many use cases, a CLI wrapper is simpler, faster to build, and good enough. The MCP advantage is strongest for cloud-hosted agents that can't access a local filesystem -- which is Anthropic's product direction (Claude Cowork, Managed Agents).

The Skills + MCP pairing section is forward-looking: MCP provides tool access, skills provide procedural knowledge for using those tools. The upcoming protocol extension to deliver skills from MCP servers would create a single distribution mechanism for both capabilities and playbooks -- a meaningful step toward agent composability.

Willison's [[Stateless MCP]] provides the independent-practitioner complement: where this guide argues from Anthropic's platform perspective, Willison validates the stateless spec by actually building against it — shipping three tools (mcp-explorer, datasette-mcp, llm-mcp-client) in a week, with datasette-mcp succeeding on its fourth attempt only because the stateless redesign removed the complexity that had killed three prior versions.

For practitioners building agent systems, this is essential reading alongside [[Awesome Agentic Patterns]] (the pattern catalogue) and [[Two Kinds of User Are Emerging]] (the adoption context). Sierra's [[The MCP Gateway Iceberg]] provides the most detailed production case study to date — validating the CLI-over-MCP pattern (Lesson 6), documenting the "80% of a workflow = 0% of the value" coverage problem, and describing a multi-pass cross-customer data guard that operationalizes the tool-design principles laid out here.

AMD's [[Execution-Free Agentic Program Repair]] provides the strongest empirical validation of this article's tool-design principles: their ablation study shows that exposing the full GitHub/Jira MCP toolsets (vs. a pruned set of 6 essential tools) drops localization accuracy from 70.5% to 53.6%, and eliminating response filtering (HTML cleanup, metadata stripping) causes a catastrophic collapse to 23.2% localization with 46.4% no-patch rate. Tool curation and response normalization aren't optimizations — they're the difference between a working system and a broken one.

---
*Sources: [[summary/building-agents-for-production-systems-with-mcp]]*
*Last updated: 2026-05-14*
