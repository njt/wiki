---
url: https://chrlschn.dev/blog/2026/03/mcp-is-dead-long-live-mcp/
title: "MCP is Dead; Long Live MCP!"
author: Charles Chen (@chrlschn)
date_fetched: 2026-05-15
date_published: 2026-03-14
---

# MCP is Dead; Long Live MCP!

The article argues that while CLI tools have become the latest hype cycle in AI agent discourse at MCP's expense, the debate lacks crucial nuance. Key distinctions — between local MCP over stdio vs. server MCP over HTTP, between individual coders and organizations, and between MCP tools vs. MCP prompts/resources — are being lost. For enterprise and team use, the author contends MCP remains essential for moving from "vibe-coding" toward disciplined "agentic engineering."

## Key Sections

### The Influencer-Driven Hype Cycle

Chen describes watching the MCP frenzy from the sidelines at Motion, initially skeptical: vendors pitched MCP wrappers at a premium, and his response was that "it's just an API; why do I need a wrapper around that when I can just call the API directly." The team bypassed MCP entirely by writing small REST API wrappers.

He then observes that social media influencers — "constantly needing content to stay relevant" — have now flipped from praising MCP to promoting CLI tools as the new hot trend. He draws a sharp comparison, viewing figures like Garry Tan and Andrew Ng as "no different than influencers peddling Ivermectin as a cure-all or anti-vax conspiracies; it's simple ignorance."

### Understanding the Misunderstanding

Chen acknowledges MCP is "indeed the wrong choice for a class of use cases" but argues the CLI-vs-MCP framing misses deeper tradeoffs.

On token savings, he identifies three ways CLIs can save tokens:
1. CLI tools in the training dataset — familiar tools like jq, curl, git benefit from existing model knowledge
2. Chained extraction and transform — but "not unique to CLIs"
3. Progressive context consumption via --help — countered with five points, including Vercel's research finding agent performance improved when the full doc index was in AGENTS.md

### The Duality of MCP

The article's central thesis draws a sharp line between local MCP (stdio mode) and remote MCP (streamable HTTP).

For local stdio mode, Chen agrees it often adds unnecessary complexity over a simple CLI.

But MCP over streamable HTTP is called "an absolute game changer" and "a key linchpin in organizational and enterprise adoption."

Six arguments for centralized, HTTP-based MCP:
1. Centralization — server-side tooling logic with organizational unlocks
2. Richer underlying capabilities — Postgres with graph extensions, heavy backends
3. Ephemeral agent runtimes — offloads stateful workload management to centralized servers
4. Auth and security — OAuth, "sensitive API keys and secrets can be controlled behind the server"
5. Telemetry and observability — OpenTelemetry traces and metrics
6. Standardized, instant delivery of up-to-date content — subscriptions and notifications

### Prompts and Resources Deep Dive

Chen elaborates on three benefits of server-delivered prompts/resources:
- Dynamic content — servers can generate skills or docs on the fly with injected context
- Automatic and consistent updates — no manual syncs
- Org-wide knowledge — standard practices delivered across "all repositories, all workflows, all teams, and all agent front-ends"

### Closing Thoughts

Chen returns to the hype cycle critique, noting influencers "need to keep chasing something new to keep their audience engaged." He cites Amazon's AWS challenges — where the company required senior engineers to sign off on AI-assisted changes after outages — as evidence that "teams eventually have to operationalize and maintain these software systems produced by AI agents."

His parting advice: take individual success stories from figures like Tan and Ng with skepticism because what works "individually in a homogeneous environment likely will not hold true for teams and orgs."
