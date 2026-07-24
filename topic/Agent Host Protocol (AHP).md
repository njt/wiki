# Agent Host Protocol (AHP)

Microsoft's draft protocol standard for decoupling AI agent sessions from any single client application — think LSP, but for AI agent sessions instead of language intelligence. AHP turns a session into a shared resource that "lives once and can be driven from anywhere."

---

## Pr&eacute;cis

The Agent Host Protocol is Microsoft's JSON-RPC 2.0-based protocol that lets multiple clients (IDE, web, CLI, mobile) concurrently attach to a single AI agent session with synchronized state. It's built on an immutable Redux-style state tree, pure reducers that run identically on server and client, monotonic sequencing for total ordering, and optimistic write-ahead reconciliation. Released MIT-licensed in 2026, it's still a draft but structurally ambitious: the first serious attempt at a vendor-neutral, transport-agnostic protocol for agent sessions that positions itself in the LSP/DAP lineage. Microsoft is the author, but the protocol is explicitly agent-backend-agnostic (Copilot, Claude, Codex, ACP all listed as first-class targets).

## What It Actually Specifies

AHP defines six channel types (`ahp-root://`, `ahp-session:/`, `ahp-chat:/`, `ahp-terminal:/`, `ahp-changeset:/`, `ahp-otlp:`) with per-channel state trees, 23 client-to-server commands, 10 server-to-client commands, 4 client notifications, and 9 server notifications. Every message carries `params.channel: URI`, enabling routing by `(method, channel)` without per-method deserialization — a clean separation that LSP never achieved.

The resource subsystem is comprehensive: `resourceRead`, `resourceWrite`, `resourceList`, `resourceCopy`, `resourceDelete`, `resourceMove`, `resourceResolve`, `resourceMkdir` — all bidirectional, meaning the server can reach into the client's filesystem just as the client reaches into the host. This symmetry is the protocol's most interesting architectural choice and its clearest divergence from LSP's one-way model.

Reconnection is first-class: `reconnect` returns either a replay of missed actions or a fresh snapshot, keyed on `lastSeenServerSeq`. Authentication follows RFC 9728 (OAuth 2.0 Protected Resource Metadata) with an `auth/required` push notification — the protocol doesn't authenticate *you*, it authenticates access to protected resources within the session.

## Key Quotes

> "Every change is one envelope, totally ordered."

This is the protocol's thesis statement. Not "here's a chat stream" — a single ordered stream of typed mutations that every client replays deterministically. It's event sourcing for agent sessions.

> "Clients optimistically apply their own actions locally, then reconcile when the server echoes those actions back alongside any concurrent changes from other clients."

The write-ahead reconciliation pattern. You type, it appears immediately, the server echoes back with `origin { clientId, clientSeq }`, you swap your optimistic copy for the authoritative one. Concurrent edits from other clients fold into the same stream. This is the hard part of multi-client sync and AHP solves it with sequencing, not CRDTs.

> "Pure functions: `(state, action) → newState`. Run identically on server and client."

Deterministic state convergence through reducer purity. This is Redux for agent infrastructure. The protocol bets that making the reducer the invariant is simpler than making the state-sync protocol the invariant.

> "Every command's params and every notification's params carry a top-level `channel: URI`."

A single routing key on every message. No per-method dispatch tables needed. This is the kind of architectural clarity that comes from building the second or third version of something, not the first.

## Relationship to Existing Protocols

AHP explicitly positions itself in the LSP/DAP tradition: a protocol that created an ecosystem by decoupling tools from implementations. LSP separated editors from language servers; AHP separates clients from agent sessions. But AHP goes further — bidirectional RPC, multi-client by design, reconnection as a first-class primitive. LSP took a decade to get to this level of ambition.

The relationship to MCP and ACP is acknowledged but not yet fully specified. MCP is the tool-access protocol (how agents call external services); AHP is the session protocol (how clients observe and drive agents). ACP is the agent-to-agent protocol. The three form an emerging stack: AHP for session sync, MCP for tool integration, ACP for inter-agent communication.

## Critical Analysis

**The good:** AHP is the first protocol I've seen that takes multi-client agent sessions seriously as a distributed systems problem rather than a UI problem. The immutable-state-with-pure-reducers model is exactly right — it's how collaborative editing should have been built from the start. The channel-based routing keeps the protocol parseable without a full deserialization layer. The resource subsystem being fully bidirectional is smart: the server asking the client for files is the use case that every agent protocol eventually needs and most bolt on later.

**The concerning:** This is a Microsoft protocol. That's not inherently bad — TypeScript, LSP, and VS Code all started here — but the "agent host" as central authority running Microsoft's state tree is a play for infrastructure ownership. If AHP succeeds, the agent session becomes a Microsoft-defined artifact, and the clients become views on Microsoft-defined state. The protocol is MIT-licensed, sure, but the reference implementation will set de facto standards, and the complexity of building a compatible host means most teams will just use Microsoft's.

**The missing:** Agent-to-agent communication is acknowledged but unspecified. Multi-agent orchestration — the thing everyone is actually building — has no channel type. There's no permissions model for which client gets to dispatch which actions, no capability negotiation beyond `protocolVersion`, and no mention of how approval flows (human-in-the-loop tool confirmations) work across multiple clients. The HITL problem — two humans on different clients both approving the same tool call — is the kind of edge case that kills multi-client protocols in production, and AHP doesn't address it yet.

**The bet:** AHP is betting that agent sessions become infrastructure — shared, durable, multi-surface — the way source code became infrastructure with git. If that bet is right, the protocol that standardizes session state wins. If agent sessions stay ephemeral and single-client, AHP is over-engineered. Given how fast the field is moving toward multi-agent, multi-client, persistent-session architectures, the bet looks sound.

## Cross-Links

- [[Components of a Coding Agent]] — AHP is the protocol layer of the harness stack
- [[Open Source Agent Toolkit 2026]] — AHP would sit in the "protocols" layer alongside MCP and ACP
- [[The Log is the Agent]] — AHP's action-log-plus-reducers model is the protocol-level equivalent of event-sourced agent architecture
- [[Real-Time Multiplayer Interfaces]] — AHP is the protocol infrastructure for "durable agents make every interface multiplayer"
- [[All Your Agents Are Going Async]] — "HTTP is the wrong transport for agents" — AHP agrees and goes transport-agnostic
- [[Traycer]] — multi-client agent protocol with independent client/host release cadences; convergent evolution
- [[Agent-Native Architectures (Every)]] — five design principles for agent-native systems; AHP is a protocol implementation of several of them
- [[Building Agents for Production Systems with MCP]] — MCP as tool layer; AHP as session layer; they're complementary
- [[10 Principles for Agent-Native CLIs]] — AHP's channel model as an agent-native design pattern
- [[Omnigent]] — Databricks' uniform API for agent composition with real-time session sharing; similar problem, different layer
- [[Coding Agents Continuity Not Memory]] — AHP's session persistence is one answer to the continuity problem
- [[Smart Models Dumb Pipes]] — AHP is the definitive "dumb pipe" for agent sessions

---
*Sources: [[raw/agent-host-protocol]]*
*Last updated: 2026-07-25*
