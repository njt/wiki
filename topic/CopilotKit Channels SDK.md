# CopilotKit Channels SDK

The first SDK to treat messaging platforms as a compilation target for AI agent interactions, not as a protocol to implement. Channels is Apache 2.0 infrastructure that connects any AG-UI-compatible agent to Slack, Microsoft Teams, Discord, Telegram, and WhatsApp — rendering the same JSX-described UI natively on each platform — through a managed cloud tier that owns the platform credentials so developers never touch a Slack token.

---

## Architecture

Channels uses a **two-tier proxy architecture** where an open-source SDK runs in the developer's infrastructure and a managed service (CopilotKit Intelligence) handles platform ingress, credential management, and native UI delivery.

```
User (Slack/Teams) → Intelligence (managed) ←WebSocket→ Channel SDK (your process) → Agent (AG-UI)
```

The Channel SDK is a **long-running Node.js process** (Node 22+, ESM, global `WebSocket`). It cannot run serverless — it owns a persistent gateway connection. The lifecycle is managed by `@copilotkit/runtime` v2, not by direct `start()`/`stop()` calls: you attach the Channel to a `CopilotRuntime`, create a listener, and the listener starts the Channel. A `channels.ready({ timeoutMs })` call blocks startup until activation settles, and a `status().overall === "online"` gate is required because `ready()` resolves on `setup_required` too.

The key architectural files in the repo itself (`CoplotKit/channels-sdk`) are references, not the SDK source — the actual SDK lives in the [`CopilotKit/CopilotKit`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/channels) monorepo. This repo serves as documentation, examples, and an agent skill for building Channels apps. The `examples/minimal-channel/` directory (`server.ts`, `lib/channel.ts`, `lib/runtime.ts`, `lib/env.ts`) is the canonical smallest-complete-listener, and `.agents/skills/build-channels-agent/SKILL.md` is the API authority (1,000+ lines of typed, cross-version-verified documentation with nine evals).

### The Managed/Adapter Duality

Channels has two deployment paths that are explicitly not interchangeable:

1. **Managed** (primary): `createChannel({ name, identifyUser, agent })` — no adapter, no platform tokens in code. Intelligence holds the Slack/Teams credentials. `name` must match the Channel Code from the Intelligence dashboard. This is the documented default.

2. **Direct adapter** (secondary): `createChannel({ adapters: [slack({ botToken, appToken })] })` — you hold the tokens. Still requires Intelligence for lifecycle. The skill explicitly warns against reaching for this because a managed Channel reports `setup_required` — "swapping to a direct adapter to 'make it work' is a known failure mode, not a fallback."

`adapters` is an array — one Channel can span multiple platforms at once.

## Key Techniques

### JSX as Platform-Neutral IR

This is the SDK's most distinctive technical choice. UI is described as JSX imported from `@copilotkit/channels`, not as hand-built Slack Block Kit JSON or Teams Adaptive Cards. The JSX runtime is **not React** — the `tsconfig` must set `"jsxImportSource": "@copilotkit/channels"` so compiled JSX becomes `ChannelNode[]`, a platform-neutral intermediate representation.

Each `PlatformAdapter` (Slack, Teams, Discord, Telegram, WhatsApp) implements a renderer that maps `ChannelNode[]` types to its platform's native constructs. The renderer is **total** — it skips unsupported nodes rather than throwing, so a `<Chart>` degrades gracefully on a surface without chart rendering.

The component vocabulary includes layout (`<Message>`, `<Section>`, `<Header>`, `<Fields>`, `<Divider>`), interactive (`<Button>`, `<Select>`, `<Input>`, `<Actions>`), and data (`<Table>`, `<Chart>`, `<Image>`). `awaitChoice<T>(jsx)` posts a picker and **blocks the handler** until the user clicks, resolving to the typed `value` of the clicked control. This enables gating destructive tool calls on confirmation directly inside a `defineChannelTool` handler.

### Content-Stable Action IDs

Interactive handlers (button clicks, select submissions) are keyed by **content-stable IDs**: `"ck:" + sha1(name | path | stableStringify(props)).slice(0, 16)`. The same rendered control always produces the same ID, so a click maps back to the right handler even after the message scrolled off-screen. Inline handlers route **in-process only** — lost on restart. For durability across restarts, you register components via `createChannel({ components })` and provide a durable `StateStore` adapter (Redis, Postgres, etc.) — the SDK will re-bind handlers after a redeploy.

### Agent Cloning Per Turn

Concurrency defaults to `"parallel"`, but the developer doesn't manage isolation: Channels **clones the agent per turn** for every configuration shape — singleton, factory returning one object, factory returning a new object per call. What it cannot fix is a broken `clone()` on a custom `AbstractAgent` subclass. The agent factory receives `(threadId)` — prefer returning a fresh agent per thread so conversations stay isolated.

### Human-in-the-Loop as Library Feature

Two patterns, both first-class. `awaitChoice<T>` lets handler code ask the user and block on the answer. `onInterrupt` + `resume` handles agent-initiated pauses (e.g., LangGraph interrupts during `thread.runAgent()`). Both survive restarts when paired with a durable store and registered components. The reference docs (`hitl-patterns.md`) are unusually clear about the durability boundary: inline closures are ephemeral; registered components with a durable store are the production path.

### Memory as Explicit Grant

`thread.runAgent({ memory: { user: "read-write", project: "read" } })` — Intelligence Memory is **disabled by default** and must be explicitly granted per run. Each scope (`user`, `project`) can be `"none"`, `"read"`, or `"read-write"`. Omitting the option disables Memory entirely. This is a sharp contrast to frameworks that implicitly stuff everything into context — Channels makes the memory boundary explicit and per-invocation.

## Design Decisions

### You Must Use Intelligence

There is no standalone or DIY way to run a Channel. Even the direct-adapter path requires a `CopilotKitIntelligence` connection for lifecycle management. This is the single biggest trade-off: **operational simplicity at the cost of platform dependence**. You don't manage Slack tokens, reconnection logic, or ingress — but you also can't run Channels without a CopilotKit API key (free tier available). Compare this to [[clawdBot]] (fully self-hosted, no external dependency) or [[Pi-msg — XMPP Bridge for Pi Coding Agent]] (thin bridge, no managed service).

### Long-Running Process, Not Serverless

Channels cannot run in a Lambda or Vercel function — it owns a persistent WebSocket. This is an architectural commitment to **connection ownership**. The trade-off is deployment complexity (you need a process monitor, health checks, restart logic) vs. the reliability guarantees of a persistent connection. This validates the thesis in [[All Your Agents Are Going Async]] — "HTTP is the wrong transport for agents that outlive connections."

### Batteries-Included Package

`@copilotkit/channels` is one npm package containing the engine, all UI components, all platform adapters, and testing tools. This is simpler than modular sub-packages but means you can't tree-shake unused adapters. The umbrella also creates a **deduplication hazard**: `@copilotkit/channels` and `@copilotkit/runtime` both pin one exact version of `@ag-ui/client`, but a transitive dependency (`@ag-ui/mcp-middleware`) pulls an older one. Two copies → two `AbstractAgent` declarations → every `createChannel({ agent })` fails with a confusing private-property error. The fix is an explicit `overrides`/`resolutions` pin.

### Agent-Agnostic Design

Channels doesn't care what agent you use — `BuiltInAgent`, `HttpAgent`, LangGraph, CrewAI, Mastra, Pydantic AI, Google ADK — as long as it speaks AG-UI. The agent factory returns `AbstractAgent`, and the runtime owns the loop. This is a clean separation: Channels owns the channel, the agent owns the reasoning. It's infrastructure, not a framework.

### The Name Migration

An earlier pre-release used `Bot`-prefixed names (`createBot`, `defineBotTool`, `BotToolContext`). The shipped packages use `Channel`-prefixed names exclusively. This is a minor detail but reveals design intent: "bot" implies a single-purpose automaton; "channel" implies a persistent presence that meets users across surfaces. The README's hero image caption is "Any agent. Any channel." — not "Any bot. Any chat."

## Comparison Notes

**vs. [[Introducing Claude Tag]]**: Claude Tag is a product (Anthropic's own agent in Slack, closed, single-platform). Channels SDK is infrastructure (open-source SDK, any agent, multi-platform). Both share the premise that agents belong in chat, but Channels separates the agent from the channel while Claude Tag bundles them. Claude Tag's ambient/passive context absorption from channel activity is something Channels could implement but doesn't prescribe.

**vs. [[clawdBot]] / [[Personal Agents]]**: clawdBot is a consumer product — one-line install, runs your personal AI on every platform. Channels is an SDK for engineering teams building custom chat agents. clawdBot's multi-platform coverage (WhatsApp, Telegram, Slack, Discord, Signal, iMessage) is broader than Channels' current set, but clawdBot renders plain text/markdown while Channels renders platform-native interactive UI.

**vs. [[Pi-msg — XMPP Bridge for Pi Coding Agent]]**: Both connect coding agents to chat. Pi-msg is a thin Go bridge (4,700 lines) for one agent via XMPP. Channels is a full SDK. Pi-msg bets on open federated protocol (XMPP); Channels bets on managed connections to proprietary platforms. Different philosophies about agency: Pi-msg pipes text; Channels renders native UI.

**vs. [[Agent Host Protocol (AHP)]]**: AHP is session sync (multi-client); Channels is the bridge from agent to platform. Both are protocol-layer infrastructure that separate models from surfaces. Channels' platform-agnostic IR (`ChannelNode[]`) is the same conceptual move as AHP's channel-based state trees — define a neutral representation, then compile to surface-specific output.

**vs. [[Hermes]]**: Hermes (149k stars) connects to every messaging platform as a consumer product with self-improving skills. Channels is the SDK for building custom agents that render native UI. Hermes does plain text; Channels renders Block Kit, Adaptive Cards, and native components. Hermes is batteries-included for the end user; Channels is batteries-included for the developer.

## Considerations

**The platform dependency is real.** Requiring Intelligence means Channels apps have a hard dependency on CopilotKit's infrastructure. If Intelligence goes down, your agent stops responding. If CopilotKit changes pricing or deprecates the free tier, every Channels app is affected. This is a different risk profile from fully self-hosted solutions like [[clawdBot]].

**The JSX abstraction leaks at the edges.** `<Table>` and `<Chart>` degrade gracefully, but `<Modal>` throws `ModalRenderError` if the view uses an element the surface can't express — unlike message rendering, modal rendering is not skip-and-degrade. The `Modal({ children: [...] })` function-call syntax (rather than `<Modal>`) is a workaround for the JSX runtime declaring `JSX.Element = ChannelNode`, which erases the `ModalView` type narrowing.

**The Agent SDK landscape is consolidating around AG-UI.** Channels' bet on AG-UI as the interop standard is shared by a growing number of frameworks. This positions Channels as infrastructure that benefits from ecosystem convergence rather than as a framework that needs to win a format war.

---
*Sources: [[raw/channels-sdk]], [[summary/channels-sdk]]*
*Tags: #tool #project #agents #chat #infrastructure #multi-platform*
*Last updated: 2026-08-07*
