---
url: https://microsoft.github.io/agent-host-protocol/
title: "Agent Host Protocol (AHP)"
author: Microsoft
date_fetched: 2026-07-25
date_published: 2026
---

AHP is a portable, standalone server protocol that gives multiple clients a
synchronized view of AI agent sessions. It follows the lineage of LSP and DAP,
with a single Agent Host sitting between multiple clients (IDE, Web, CLI,
Mobile) and multiple agent backends (Copilot, Claude, Codex, ACP).

State is modeled as an immutable Redux-style tree per channel, changed only by
actions flowing through pure reducers that run identically on server and client.
Every mutation is stamped with a monotonic sequence number and broadcast to all
subscribed clients, giving total ordering. Clients apply their own actions
optimistically, then reconcile against the server's echo via an `origin` field.

The wire protocol uses JSON-RPC 2.0 over any reliable, ordered, bidirectional
stream. Messages route through named channels (Root, Session, Chat, Terminal,
Telemetry, Resource Watch), each with its own URI pattern and state tree.

The spec is a draft — breaking changes to wire types, actions, and state shapes
are expected. TypeScript type definitions are the source of truth, with JSON
Schema 2020-12 artifacts generated for validation and code generation. Released
under the MIT License.
