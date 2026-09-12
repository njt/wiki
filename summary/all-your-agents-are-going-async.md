---
url: https://zknill.io/posts/all-your-agents-are-going-async/
title: All your agents are going async
author: Zak Knill
date_fetched: 2026-05-15
date_published: 2026-04-20
topics:
  - agent-architecture
---

# All your agents are going async

Author: Zak Knill (publishing under /dev/knill)
Date: April 20, 2026

## Summary

AI agents are shifting from synchronous chat interfaces to background-async operations. When that change happens, "the transport breaks" — HTTP is inadequate for async agent communication. The article identifies four scenarios HTTP can't handle (agent outlives caller, unprompted push, caller changes, multiple humans), splits the problem into durable state and durable transport, and argues that current solutions from Anthropic and Cloudflare only solve the state half.

## Full Content

### The Shift Away from Chat

LLMs have traditionally been used through chat-style windows with streaming responses. This is not the full picture. The core argument: "all your agents are going async." Agents are gaining crons, webhooks, WhatsApp integrations, remote control from phones, scheduled tasks, and routines via platforms like Temporal, Vercel WDK, and Relay.app. A human at a terminal is "just one mode now."

### OpenClaw's Async Step

OpenClaw demonstrated async agents by embedding one in WhatsApp, showing users didn't need to stay glued to a browser. Anthropic responded with Channels (MCP-based, async message pushing into Claude Code sessions), along with /loop, /schedule slash commands, Routines, and Remote Control (continuing sessions from a phone). ChatGPT introduced scheduled tasks. Cursor launched background agents running in the cloud.

These all break "the coupling between a human sitting at a terminal or chat window and interacting turn-by-turn with the agent."

### The Transport Mismatch

The core problem: "the lifetime of an agent's work is decoupled from the lifetime of a single HTTP connection." In chatbot demos, processing only occurs while the HTTP connection is open. "A chatbot's worst enemy is page refresh."

Four scenarios HTTP can't handle:

1. **Agent outlives the caller** — A cron-triggered routine finishes with results but "no one's listening anymore." Results end up in a database requiring polling via session URL.
2. **Agent wants to push unprompted** — A nightly backlog review finishes and needs to notify you, or a workflow hits a human approval step. Currently handled via email or Slack.
3. **Caller changes** — Starting at a desk, checking from a phone. Remote Control handles this with "custom backend session storage and management," not as a first-class transport feature.
4. **Multiple humans in one session** — A team of five working with one agent needs updates pushed to all and input accepted from any.

OpenClaw solved all four by separating the lifetime of the agent's work from the lifetime of the connection to the human, using async chat systems like WhatsApp, iMessage, Telegram, or Discord to push results.

### How the Industry is Responding

Some follow the OpenClaw model (external chat provider as conversation history). More interestingly:

- Anthropic is pulling session state into its hosted platform with Routines and Remote Control — consolidating agent lifecycle and connection state beyond just being an LLM inference API.
- Cloudflare launched its Agents platform with a Sessions API for session/conversation storage over HTTP, plus Email for Agents to fix async notifications.

### These Solutions Solve Only One Half

The problem splits into two:

1. Durable state — where agent state lives across restarts and async tasks.
2. Durable transport — how response bytes travel between agents and humans across disconnects, device switches, fan-out, and server-initiated push.

Anthropic and Cloudflare focus on durable state storage. Their approach to delivering bytes still relies on polling or HTTP GETs. Cloudflare has WebSocket support but "it doesn't survive disconnections for streaming LLM responses." This approach "half works, but it's not 'art of the possible'."

### Durable Transport, Durable State

Currently, "the session and the transport are all wrapped up in a single HTTP request-response." Cloudflare and Anthropic make session state durable but don't fix transport — leaving users with "HTTP gets, or polling."

The OpenClaw model (conversation history in the chat channel, agent and LLM provider separated) has no "enterprise" version runnable on one's own infrastructure. There's no combined durable transport + durable state solution.

### Ably's Approach

The author works for Ably and discloses they are building a durable transport for AI agents around a "session" concept, on top of their existing realtime messaging platform.

The vision: "A 'session' with an AI should be a thing that humans and agents can connect to, and disconnect from at any time." Sessions should survive WiFi issues, device disconnections, and should support multi-device and multi-user scenarios. Conversation state should be accessible through the durable session. Because Ably already has a bidirectional, durable, realtime messaging transport, they claim to be building session state and conversation history onto it to solve both halves.

## Key Quotes

- "the lifetime of an agent's work is decoupled from the lifetime of a single HTTP connection"
- "a chatbot's worst enemy is page refresh"
- "Right now, they go in a database and you have to poll for them with some session URL"
- "These solutions solve only one half"
- "the session and the transport are all wrapped up in a single HTTP request-response"
- "A 'session' with an AI should be a thing that humans and agents can connect to, and disconnect from at any time."
