---
url: https://github.com/stacklok/mecatl
title: "Mecatl"
author: Stacklok
date_fetched: 2026-10-02
date_published: 2026 (open source, Apache-2.0)
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

Mecatl is Stacklok's open-source, cloud-native agent harness: the loop, tools, permissions, hooks, delegation, and service boundaries for running AI agents as production workloads on infrastructure you operate. Unlike a terminal coding agent, it treats the harness as a server problem — durable sessions, append-only event logs, leases, and a Kubernetes-native runtime (`mecak8s`) with Redis-backed state and disposable replicas.

The core claim is seam separation: the agent loop (`engine/`) is provider-neutral, client-neutral, and environment-neutral, and everything else — model provider, state store, filesystem, UI — plugs in through explicit ports (`engine/port/`: `LLMProvider`, `SessionStore`, `SessionLease`, `PermissionPolicy`, `HookRunner`, `EventLog`, `Compactor`). Shipped binaries include `mecated` (server), `mecatui` (terminal client that can host or connect), `mecatequi` (one-shot headless automation), `mecak8s` (K8s runtime), plus a Go SDK for embedding the engine.

Security posture is a first-class design axis: deny-dominant permissions with approval flows, read-parallel/mutate-serial tool dispatch, delegated runs whose derived capabilities only narrow, an audit trail with durable attribution, and an opt-in local microVM execution backend that keeps provider credentials on the host while filesystem and shell tools run inside the VM. The README is candid that cross-process cryptographic identity and tenant isolation are still open design work.

The architecture docs are unusually deep — a 2,200-line `docs/architecture.md`, per-subsystem guides, frozen ADRs, a domain model with conformance-test adapters for every port — and the repo is itself developed with Claude Code (`.claude/agents/` and `.claude/skills/` ship reviewer subagents and skills).
