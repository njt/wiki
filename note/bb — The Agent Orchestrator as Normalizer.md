# bb — The Agent Orchestrator as Normalizer

bb is an open-source orchestrator that wraps multiple AI coding harnesses (Claude Code, Codex, Pi, Cursor, and others) behind a unified JSON-RPC interface, normalizing their heterogeneous output into a shared thread-event timeline. It doesn't call LLM APIs itself — it shells out to existing agent harnesses and translates their events. Think of it as a **protocol adapter layer** for the coding-agent ecosystem, with projects, worktrees, permissions, and skills as the orchestration surface.

---

## Key Quotes

> "bb does **not** call Anthropic/OpenAI HTTP APIs itself for coding threads. It shells out to **existing agent harnesses** and normalizes their traffic into a shared bb thread-event model."

This is the architecture thesis in one sentence. bb isn't another framework competing with Claude Code or Codex — it's infrastructure *above* them, treating harnesses as interchangeable backends. Compare [[Introducing Omnigent]] (wraps agents in a uniform API), [[Traycer]] (wraps 17+ agents in a desktop app), and [[Warp Agent CLI]] (delegates to harnesses as workers). All four are converging on the same insight from different angles: the next frontier is **inter-harness infrastructure**, not better prompts or models.

> "Two sanctioned adapter shapes: in-process protocol adapter (Codex app-server) and bridge-process adapter (Node bridge hosting SDK/ACP client)."

The adapter taxonomy is what makes bb's architecture more disciplined than the typical "just shell out to a CLI" approach. In-process adapters speak the provider's native JSON-RPC directly; bridge adapters host an SDK or ACP client in a Node process that exposes bb's common surface on stdio. This two-shape model means bb can add providers without rewriting its core, and each adapter owns its own event translation — the agent runtime never interprets provider-specific wire content.

> "The accurate answer is: **JSON-RPC into a Node bridge that drives the Claude Agent SDK's query() against the local claude binary** — not a direct Anthropic Messages API call from bb itself."

This precision matters. It means bb piggybacks on each harness's existing auth, session management, and tool schemas rather than reimplementing them. The Claude bridge literally forwards raw `SDKMessage` objects, leaving the adapter to translate them into bb's `ThreadEvent[]`. Skills become local plugins, dynamic tools become an in-process MCP server named `bb-bridge`. This is the opposite of a lowest-common-denominator wrapper — bb preserves each harness's full capabilities but makes them speak a common event language on the way out.

> "Codex processes are thread-scoped. Claude Code, Pi, and ACP bridges are provider-scoped. One bridge process can host multiple bb threads."

The process-lifecycle distinction is a performance decision disguised as an architectural one. Thread-scoped processes isolate state but cost spawn latency. Provider-scoped bridges amortize startup but couple threads through a shared Node process. The choice maps to each harness's own session model: Codex's JSON-RPC server is cheap and stateless-per-thread; Claude Code's SDK session carries state across turns, making a long-lived bridge sensible.

> "ACP support is a **subset** of the protocol — enough for session/prompt/tools/permissions/fs used in practice."

Selective protocol implementation, not spec maximalism. bb implements the ACP verbs that coding agents actually use: initialize, authenticate, session/new, session/prompt, session/update, session/request_permission, fs/read_text_file, fs/write_text_file, session/cancel. No attempt to cover the full spec. This is pragmatic protocol engineering — implement what's needed, leave the rest as future work.

## Architecture Diagram

```
UI / CLI / HTTP API
  → bb Server (SQLite state, product policy)
  → Host daemon WebSocket RPC (thread.start / turn.submit)
  → AgentRuntime (process lifecycle, JSON-RPC framing)
  → ProviderAdapter (buildCommandPlan + translateEvent)
  → Provider process (codex app-server | bridge → SDK/ACP)
  → events stream back → daemon → server → clients
```

The daemon is the boundary between bb's world (projects, environments, permissions) and the provider's world (tool calls, responses, agent reasoning). Every event flowing back through this pipeline is stamped with both a bb `threadId` and a `providerThreadId`, making the timeline navigable from either side.

## Key Themes

- **#tool** bb — open-source agent orchestrator and event normalizer
- **#pattern** Protocol-adapter architecture — two-shape adapter model (in-process + bridge) for wrapping heterogeneous agent harnesses
- **#pattern** Cross-harness normalization — each harness keeps its own session identity; bb adapts them into one timeline
- **#concept** Provider-scoped vs. thread-scoped process lifecycle — the architectural choice that maps to each harness's session model
- **#concept** Minimal ACP client — implement only the protocol subset coding agents actually use
- **#pattern** Tool-proxy MCP — bb's dynamic tools exposed as in-process MCP servers to provider agents

## Critical Analysis

**What's real and novel.** The two-adapter taxonomy (in-process for protocol-native providers, bridge for SDK/ACP providers) is genuinely well-designed. It separates concerns cleanly: the agent runtime multiplexes without interpreting provider content, each adapter owns translation for its provider, and bridges are thin JSON-RPC shells that forward SDK messages. Compare this to [[Traycer]] (normalizes via discriminated unions in TypeScript) and [[Warp Agent CLI]] (normalizes via PTY multiplexing) — bb has the most principled separation of concerns, but at the cost of requiring bridge development per-provider.

**The Omnigent comparison is instructive.** Both are meta-harnesses that wrap existing agents. But [[Introducing Omnigent]] wraps at the user-facing input/output layer ("messages and files in, text streams and tool calls out"), while bb operates at a lower level — it speaks each provider's native protocol (JSON-RPC, ACP, SDK) and translates events into a shared model. Omnigent bets on abstraction; bb bets on adaptation. The trade-off: Omnigent works with any agent that takes text input, but loses fidelity; bb preserves full harness capabilities, but requires per-provider adapter engineering.

**The "normalizer" framing is underrated.** Most orchestration discussions focus on task decomposition, assignment, and verification. bb focuses on a different problem: **how do you make five different coding agents produce output that looks like one conversation?** This is the event-model problem — each harness has its own event taxonomy, streaming semantics, and permission model. bb's answer (provider-specific adapters → shared `ThreadEvent[]`) is the same pattern that [[Traycer]] uses (normalized `RuntimeEvent` discriminated union) and that the Agent Client Protocol itself aims to solve at the protocol level. If ACP succeeds, bb's adapter layer shrinks; if it doesn't, bb's approach becomes more valuable.

**Process lifecycle is a microcosm of a larger tension.** Thread-scoped vs. provider-scoped processes mirror the trade-off between isolation and efficiency that appears everywhere in distributed systems. Thread-scoped Codex processes are clean (one thread, one process, no cross-contamination) but expensive at scale. Provider-scoped Claude bridges amortize costs but couple threads through a shared Node process. The right answer depends on the harness's own architecture — which is why bb doesn't pick one model and force it on every provider. This is good design: let the adapter choose the lifecycle that matches the harness.

**What's missing.** The research notes don't cover failure modes. What happens when a bridge process crashes mid-turn? How does thread state recovery work when the Claude SDK's `resumeSession` fails? What's the observability story for debugging a cross-harness thread? These are the hard parts of meta-harness engineering, and they're absent from the architecture overview. Every meta-harness ([[Introducing Omnigent]], [[Warp Agent CLI]], [[Traycer]]) faces them, and none has published a comprehensive failure-mode analysis.

**The MCP tool-proxy pattern is emerging as a standard.** bb exposes its dynamic tools as an in-process MCP server named `bb-bridge`, with tool names like `mcp__bb-bridge__<name>`. Claude Code's bridge sees them as MCP tools; the adapter translates calls back through bb's plugin system. This is the same pattern as [[Introducing Omnigent]]'s MCP integration and the broader ecosystem trend of MCP as the universal tool-proxy layer. When every meta-harness converges on the same pattern independently, it's worth paying attention.

## Related

- [[Agent Orchestration]] — hub page; bb is cross-harness orchestration infrastructure
- [[Introducing Omnigent]] — same meta-harness concept, different layer: Omnigent wraps at the I/O layer, bb speaks native protocols
- [[Warp Agent CLI]] — cross-harness orchestration as a product; bb is the open-source architectural reference for the same pattern
- [[Traycer]] — wraps 17+ agents with a versioned RPC protocol; bb normalizes events through adapter-per-provider translation
- [[Agent Host Protocol (AHP)]] — Microsoft's JSON-RPC protocol for multi-client agent sessions; bb's server-daemon-runtime RPC pipeline is a convergent design
- [[Components of a Coding Agent]] — harness > model; bb is the harness one level up
- [[Open Source Agent Toolkit 2026]] — orchestration as a distinct layer; bb fits the orchestration slot
- [[Paperclip]] — normalizes one level up from bb: where bb translates in-thread provider events into a shared timeline, Paperclip translates whole-run outcomes (done/blocked/needs_review/yielded) and layers org charts, budgets, and governance on top

---
*Sources: [[raw/73beb12a47851dc0b3ce34aef4d8d529]], [[summary/73beb12a47851dc0b3ce34aef4d8d529]]*
*Last updated: 2026-08-07*
