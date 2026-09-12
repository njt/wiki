---
url: https://xcancel.com/i/article/2080002574472929392
title: "The MCP Gateway Iceberg"
author: Mihai Parparita
date_fetched: 2026-07-25
date_published: 2026-07-22
topics:
  - agent-architecture
---

Sierra built a single MCP-powered gateway to connect its internal AI agents (led by "Pinecone") to 45+ SaaS tools — Slack, GitHub, Salesforce, data warehouses, and more. What looked straightforward on the surface turned out to be an engineering iceberg. The article distills seven lessons from the build.

**Reduce coordination and collapse roles.** One team "grabbed the lock" on the gateway, owning it end-to-end. This prevented every team from building near-identical integrations, permission models, and audit systems. Once the gateway was usable, the builders switched to product-manager mode: recruiting early adopters, running a shared feedback channel, and routing one-off ideas into features that benefited everyone.

**Coding agents still need humans.** Agents cheat — they bypass broken auth by reading local tokens, or fall back to raw HTTP when MCP servers aren't spec-compliant. Sierra validated gateway tools with consumer-grade agents (ChatGPT, Claude) whose limited capabilities forced use of the official interface. A living `mcp-gateway.md` design doc, read before every major task and updated after, dramatically improved one-shot success rates. An integrated MCP Inspector let humans establish ground truth when agent conclusions didn't add up.

**An agent can't misuse what it never sees.** Preventing cross-customer data leaks required a multi-pass audit system: a deterministic phase identifies candidate customers, a fast model narrows them, and a slower model makes the final call on customer identity and sensitivity. Cross-customer access is allowed only with explicit out-of-band approval, fully logged. This gave legal and compliance the confidence to greenlight company-wide rollout.

**80% of a workflow rounds down to 0%.** An automation tool covering 80% of user needs delivers none of the value — it must span the full workflow or be end-user extensible. Sierra added a REST extension mechanism to fill gaps in official MCP servers, plus sidecar services and multi-region deployments for tools like Grafana and OpenSearch that didn't fit the clean proxy model. The complexity stays below the surface; users just see tools with an injected region parameter.

**Avoid the strategy tax.** The gateway integrates tightly with Pinecone but doesn't require it — any agent (local coding agents, Claude Design, etc.) can use it. This let new tools access real data on day one, and keeps production investigations possible even when Pinecone is down. The cost is client-compatibility work reminiscent of early web development, but debugging OAuth and protocol quirks is exactly the kind of work agents excel at.

**Don't fight the weights.** For GitHub and AWS, exposing the native CLI (`gh`, `aws`) proved better than proxying through MCP servers. Agents already know these CLIs from training data, and they provide more efficient access (filterable, pipeable output). Write operations are gated behind read-only tokens minted per session.

**Human identity and agent identity — you need both.** Interactive work runs as the user; scheduled or shared workflows run as scoped service accounts. Pre-authorized workflows for customer-data access declare allowed customers and tools before they ever run. Service ownership was pushed to the teams who understand each system, so the platform team didn't become a bottleneck.

Today 89% of Sierra employees use the gateway across 45 services, and nearly two-thirds of commits come from outside the original two-person team. The gateway has become plumbing — and increasingly, it can improve itself.
