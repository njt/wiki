---
url: https://microsoft.github.io/agent-host-protocol/
title: Agent Host Protocol (AHP)
author: Microsoft
date_fetched: 2026-07-25
date_published: 2026
---

# Agent Host Protocol

AHP is a portable, standalone server protocol that gives multiple clients a synchronized view of AI agent sessions. It achieves this through immutable state, pure reducers, and write-ahead reconciliation. Released under the MIT License, Copyright © 2026 Microsoft.

## Problem

An agent session is traditionally bound to a single application. "The conversation, the turns, the pending tool approvals — they're tied to a single app or harness." AHP transforms a session into a shared resource, allowing it to "live once and be driven from anywhere."

## Architecture

AHP follows the lineage of LSP (Language Server Protocol) and DAP (Debug Adapter Protocol). A single Agent Host sits between multiple clients and multiple agents. The host holds "authoritative session state · sequencing · reconciliation." Clients communicate via AHP; agent backends integrate directly.

Four client types (IDE, Web, CLI, Mobile) and four agent backends (Copilot, Claude, Codex, ACP).

## Key Concepts

1. **Synchronized multi-client state** — Any number of clients can attach to a single session at once.
2. **Total ordering via monotonic sequence** — The host stamps each mutation with a monotonic sequence and broadcasts it to every subscribed client.
3. **Envelope-based messaging** — Every change is one envelope, totally ordered.
4. **Optimistic apply + reconcile** — Client applies its own action locally immediately, then cross-references the server's echo (stamped with an `origin` field) back to its optimistic copy.

## Wire Protocol

- **Base framing:** JSON-RPC 2.0
- **Transport-agnostic:** "any reliable, ordered, bidirectional message stream can carry AHP messages"
- **Channel routing:** Every message carries `params.channel: URI` — the central routing key
- **Bidirectional RPC:** Both client and server can issue requests (resource* ops are symmetrical)

## Channel Model

| Channel | URI Pattern | Content |
|---|---|---|
| Root | `ahp-root://` | Agents catalog, terminals catalog, host config, session catalogue events |
| Session | `ahp-session:/<uuid>` | Per-session state, chats catalog, active clients, customizations |
| Chat | `ahp-chat:/<cid>` | Per-chat conversation, turns, streaming, tool calls |
| Terminal | per-terminal | PTY state, data flow, claims, command detection |
| Telemetry | `ahp-otlp:` | OpenTelemetry logs, traces, metrics |
| Resource Watch | separate | File/resource change watching |

## Message Categories

- **Client → Server (notification):** `unsubscribe`, `dispatchAction`
- **Client → Server (request):** `initialize`, `reconnect`, `subscribe`, `createSession`, `disposeSession`, `listSessions`, `fetchTurns`, `resourceRead`/`Write`/`List`/`Copy`/`Delete`/`Move`/`Resolve`/`Mkdir`, `createResourceWatch`, `authenticate`
- **Server → Client (request):** All resource* ops + `resourceRequest`, `createResourceWatch` (reverse direction)
- **Server → Client (notification):** `action`, `root/sessionAdded`, `root/sessionRemoved`, `root/sessionSummaryChanged`, `auth/required`
- **Server → Client (response):** Success result or JSON-RPC error

## State Model

Each state-bearing channel maintains an immutable, Redux-style state tree changed only by actions flowing through pure reducers. Subscriptions return snapshots with `resource`, `state`, and `fromSeq`. Subsequent action envelopes incrementally update the state.

**Actions** — "Discriminated union of typed mutations. The sole mechanism for state change." Delivered inside `ActionEnvelope`s with `channel`, `action`, `serverSeq`, `origin`.

**Reducers** — "Pure functions: `(state, action) → newState`. Run identically on server and client."

## Design Decisions

- Immutable state tree per channel
- Pure reducers run identically on server and client
- Monotonic sequence numbers for total ordering
- Optimistic local apply, then server echo with `origin` matching for reconciliation
- Forward-compatible versioning (SemVer, newer clients connect to older servers)
- Lazy loading (subscribe to channels, large content stored by reference)
- TypeScript origin (schemas generated from TS type definitions)
- JSON Schema 2020-12 artifacts for validation and code generation

## Status

**DRAFT** — "Breaking changes to wire types, actions, and state shapes are expected." No backward compatibility until production status.
