---
url: https://www.usecontextlayer.com/log/a-platform-for-bespoke-software
title: "A Platform for Bespoke Software"
author: ContextLayer ( unnamed founder post )
date_fetched: 2026-11-09
date_published: unknown (2026)
topics:
  - personal-agents
  - coding-agents-and-frameworks
---

ContextLayer is building "slates": a platform for vibe-coded personal tools where agents, not humans, write all the code. Because agents are the only authors, the platform removes every choice a human developer would have made — one opinionated stack (TypeScript, React, React Router, Tailwind, Kysely, "gigatons of Cloudflare"), git and auth baked in, deployment as `git push` — and instead invests in packaging, composability, and containment.

Security is the load-bearing idea: vibe-coded code is treated as untrusted, a black box nobody reviews. Slates run on Cloudflare Workers for Platforms with all egress blocked except via authorized MCP servers — one observable way for data or effects to cross the boundary, so a rogue agent deleting files has exactly one path to monitor and constrain. Even browser code is locked down with CSP headers.

The most distinctive design decision is identity: slates are never told who the logged-in user is. Instead, resources (database, folders, third-party connections) are injected at runtime and declared as belonging to either the viewer or the owner — so the same email-reading slate can serve both author and friend, touching whichever inbox is viewing it. Future directions: React Router loaders/actions auto-exposed as MCP tools, forking as the default way to share slates, and agents connecting over MCP to the live React state of every open browser session. The essay closes by asking whether this line of thinking ends at "rethinking operating systems entirely."
