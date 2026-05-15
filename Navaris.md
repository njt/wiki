# Navaris

A unified control plane for sandbox management that abstracts over multiple isolation backends -- Incus containers and Firecracker microVMs -- through a single REST API and CLI. Pick the isolation level you need without changing your code.

---

## Key Quotes

> "Running untrusted or experimental code safely requires strong isolation -- but the tooling to manage that isolation is fragmented."

## Key Themes

#sandboxing #security #agent-architecture

The core insight: containers and microVMs serve different trust levels, but the management API should be the same. Navaris lets you choose:

- **Incus (containers)**: ~1s startup, minimal overhead, namespace/cgroup isolation. Good for dev environments, CI, trusted workloads.
- **Firecracker (microVMs)**: 2-3s startup, ~30MB overhead, hardware KVM isolation. Good for untrusted code, multi-tenant environments.

The feature set goes beyond basic lifecycle management: runtime resource resizing, time-bounded CPU/memory boosts (auto-revert after the burst), snapshot/restore, copy-on-write cloning, and WebSocket event streaming. The MCP server integration means AI agents can programmatically create and manage sandboxes -- essential for [[agentic coding]] workflows where agents need to run untrusted code.

Connects to [[OpenSandbox]] (Alibaba's broader sandbox platform with more language SDKs but heavier), [[onecli]] (credential management for agents running in sandboxes), and [[llm-guard]] (complementary defense at the prompt level rather than the execution level).

## Critical Analysis

Navaris solves the "right tool for the right trust level" problem elegantly. The unified API over heterogeneous backends is the right abstraction -- teams shouldn't have to choose between containers and VMs at the API level when their trust requirements vary per workload. The Go implementation and Docker packaging make it operationally straightforward. The OpenTelemetry integration is a nice touch for production observability. The main gap is that Navaris focuses on compute isolation -- network isolation, filesystem access control, and credential injection are adjacent concerns that would need complementary tools. The `--privileged` Docker requirement for Firecracker support is a deployment wrinkle that limits some hosting environments.

---
*Sources: [[raw/navaris]]*
*Last updated: 2026-05-14*
