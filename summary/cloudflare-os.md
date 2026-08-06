---
url: https://blog.cloudflare.com/cloudflare-os/
title: "Cloudflare OS: an open platform for agents, apps, and work"
author: Cloudflare
date_fetched: 2026-08-06
---

Cloudflare announces the open-source release of Cloudflare OS, a platform that gives every person in an organization an agent workspace grounded in company context, skills, and internal systems. It was already deployed internally at Cloudflare — thousands of employees across every function use it daily — and is now available for any organization to deploy and customize.

The platform has three parts: (1) an agent workspace with browser-based interaction, curated organizational context, and an isolated code runtime; (2) a security and governance framework built around Gatekeepers — service-specific Workers that mediate access to external systems, enforce fine-grained policies, and track which resources an agent has observed so that downstream sharing can't leak data to unauthorized viewers; (3) a platform for personal, modifiable apps where every "file" can be a full-stack application (client code + server code as a Dynamic Worker + SQLite via Durable Object Facets + Cap'n Web RPC), shared either as a live collaborative app or as a blueprint others can fork and modify with AI.

Agents start with access to nothing. They request specific resources, which are granted as typed capability bindings. Gatekeepers hold credentials, enforce OAuth, apply rate limits, and record observations. Policy follows what the agent has seen — if an agent reads sensitive data and produces a dashboard, viewing the dashboard checks the viewer's access to the original resources. The platform supports any model through Cloudflare AI Gateway, with per-user/per-team attribution, budgets, and rate limits.

Two repositories are released: the Cloudflare OS core and an example deployment based on Cloudflare's internal configuration. Strategic partners Presidio and Happy Cog offer customization services. The post also mentions MCP Server Portals for connecting existing MCP servers, and future plans for a managed Cloudflare dashboard product, container support, and Slack/chat integration.
