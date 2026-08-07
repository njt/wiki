---
url: https://github.com/CopilotKit/channels-sdk
title: Channels SDK
author: CopilotKit
date_fetched: 2026-08-07
---

# CopilotKit Channels SDK — Summary

The Channels SDK (`@copilotkit/channels`) is a platform-agnostic engine for putting any AG-UI-compatible agent into Slack, Microsoft Teams, Discord, Telegram, and WhatsApp with native interactive UI. You write the logic once — handlers, tools, and JSX-rendered messages — and it runs on any supported messaging platform, rendering as native Block Kit (Slack), Adaptive Cards (Teams), or platform-specific components.

## Architecture at a Glance

The developer runs a long-running Node.js process that registers with **CopilotKit Intelligence** (a managed service that holds platform credentials and handles ingress). The Channel receives events via a persistent WebSocket, runs the developer's agent over the AG-UI protocol, renders the response into a platform-neutral IR (`ChannelNode[]`), and Intelligence delivers native UI back to the conversation. The SDK is MIT-licensed; Intelligence can be hosted by CopilotKit or self-hosted.

## Key API Surface

- `createChannel({ name, agent, identifyUser, ... })` — creates a Channel with handlers for mentions, messages, commands, and interactions
- `thread.runAgent()` / `thread.post(jsx)` / `thread.awaitChoice(jsx)` — render and drive the conversation
- `defineChannelTool({ name, parameters, handler })` — tools the agent can call; handlers get a live `thread`
- JSX components (`<Message>`, `<Section>`, `<Button>`, `<Select>`, `<Table>`, `<Chart>`, `<Modal>`) — render natively per platform
- `@copilotkit/runtime` (v2) — required to start a Channel; owns the lifecycle via `CopilotRuntime({ channels: [channel] })`

## Notable Design Decisions

- **Managed vs. direct adapters**: The primary path is managed — Intelligence holds platform tokens, your code holds none. A direct-adapter path exists for when you own the platform connection, but it's explicitly secondary.
- **No `channel.start()`**: The runtime owns lifecycle. Attach to `CopilotRuntime` + create a listener, and the Channel starts.
- **JSX as IR**: UI is described as JSX from `@copilotkit/channels`, not hand-built Block Kit JSON. The JSX runtime is not React — it compiles to a platform-neutral `ChannelNode[]` IR that each adapter renders (or gracefully degrades) for its surface.
- **Batteries-included**: `@copilotkit/channels` is one package containing the engine, UI vocabulary, all adapters, and testing tools.
- **Agent-agnostic**: Any AG-UI-compatible agent works (BuiltInAgent, HttpAgent, LangGraph, CrewAI, Mastra, Pydantic AI, Google ADK).

The reference application is **OpenTag**, an open-source on-call triage assistant with a Python LangGraph agent, native Slack/Teams UI, human approval gates, and a production-shaped Node runtime.
