---
url: https://chaosmanage.github.io/i-killed-unlimited-chatgpt/
title: "I Am the Reason OpenAI Is Killing Unlimited Chats"
author: chaosmanage
date_fetched: 2026-11-02
date_published: 2026 (undated)
topics:
  - mcp-and-tool-protocols
  - agent-orchestration
---

OpenAI's move to kill unlimited ChatGPT chats and fold them into Codex usage prompts this practitioner's writeup of what he was doing with those chats: bolting MCP servers onto ordinary ChatGPT conversations until they became a pool of coding agents.

He built a unified MCP gateway on his Windows laptop (and a mirror on his VPS), wired in Desktop Commander (files, terminal commands, test runs) and CUA (computer use — screenshots, mouse, clicking buttons), and exposed the whole thing to regular ChatGPT via OpenAI's Secure MCP Tunnel, authenticated with an API key. No custom cloud infrastructure: just his subscription, his machines, and OpenAI's tunnel. Ordinary chats could now inspect repos, edit code, run tests, and visually verify changes.

The orchestration layer is the fun part. ChatGPT chats can't spawn subagents, so he improvised: a stronger model (Astra in Codex or Pro in ChatGPT) writes a detailed implementation plan, breaks it into GitHub issues with enough context and acceptance criteria that any agent could pick one up independently, and then several parallel ChatGPT conversations (Sol 6 High) each take an issue and work it against the shared filesystem via MCP. GitHub issues become the task queue; premium-model usage is saved for architecture, hard decisions, and review. He built a lightweight dashboard to start/stop the MCP connection, watch server health, and inspect a live feed of tool calls.

The coda: OpenAI is killing unlimited chats, and the author is jumping ship — he prefers Claude models, gets more mileage from a Claude subscription, and is cheerfully unrepentant ("I'm not sorry about that. Claude here i come."). He insists he broke no rules, just used what OpenAI exposed.
