---
url: https://apron.chat/
title: "Apron Chat Protocol"
author: Apron contributors
date_fetched: 2026-10-24
date_published: 2026 (undated)
topics:
  - developer-tools
  - personal-agents
---

Apron is a minimal protocol for self-hosted chat: JSON messages over any transport (WebSocket being the example), designed so that a competent server is ~100 lines while still supporting rooms, threads, bots, and reactions. The pitch is a small core spec plus optional incremental capabilities with graceful fallbacks.

The page demonstrates the protocol with a short handshake transcript: the client asks the server for its version and auth schemes (`{"method":"server"}`), authenticates as a guest (`{"method":"auth"}`), then sends a message and receives a `message_id`. That's the whole core loop.

The ecosystem page is the argument made in code. Servers: a Go reference implementation with every capability, a Cloudflare Worker running the public demo on the free tier, and two one-file servers (Python under `uv`, Bun with zero dependencies). Clients: a SvelteKit web client and an `apron-pr-bot` that posts GitHub pull request activity to the demo server in "two frames" — one auth frame, one message frame. The bot is the agentic hook: any automation that can emit two JSON lines can talk to any Apron server, and any client works with any server.
