---
url: https://github.com/cloudflare/cloudflare-os
title: "Cloudflare OS: An AI productivity environment"
author: Cloudflare Workers team
date_fetched: 2026-08-06
date_published: 2026-08
---

Cloudflare OS is an open-source "operating system" for AI productivity built by
the Cloudflare Workers team. It provides an agent chat UI, sandboxed application
development (where agents build "gadgets" — small personal apps), and a
capability-based security framework called Gatekeepers. Originally developed for
internal use, it's now open-sourced so other companies can run their own
instance.

The architecture is built entirely on Cloudflare Workers and Durable Objects.
Every workspace is its own Durable Object, every Gadget runs in a Dynamic Worker
Facet, and Gatekeepers install facets into workspaces to manage access to remote
services. The frontend and backend communicate via Cap'n Web RPC over
WebSockets, with Yjs (V2 encoding) syncing code changes between clients and
agents in real time.

Gatekeepers are the security innovation: they wrap external services behind a
capability-based API, logging every action and providing simulated outcomes for
async human-in-the-loop approval. Instead of making the agent stop and wait for
approval on every side-effecting action, Gatekeepers simulate results locally so
the agent can proceed, queuing actions for bulk human approval later. This
solves the "agent stuck on first approval" problem that drives users to
dangerous auto-approve modes.

The agent uses a Code Mode approach — it performs tasks by writing and
immediately executing JavaScript snippets via Dynamic Workers, rather than
calling a fixed set of predefined tools. This makes every Gadget's Cap'n Web API
automatically available to the agent without additional MCP integration.

Gadgets are per-user, sandboxed application instances — like Google Docs but
where each "doc" can be a completely custom app built by AI. Blueprints are
shareable gadget templates (`.gadget` binary format). The permission system uses
capability-based introductions: agents and gadgets start with access to nothing
and must be explicitly introduced to each resource they need.
