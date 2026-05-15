---
title: "navaris"
url: https://github.com/erans/navaris
date_fetched: 2026-05-14
section: "Security"
---

# Navaris: Sandbox Control Plane

## Purpose
Unified control plane abstracting sandbox management across multiple isolation backends -- Incus (system containers) and Firecracker (microVMs) -- offering a single REST API and CLI regardless of the underlying runtime.

## Key Problem
"Running untrusted or experimental code safely requires strong isolation -- but the tooling to manage that isolation is fragmented."

## Core Capabilities
- Multi-backend support (Incus containers and Firecracker microVMs)
- Full lifecycle management (create, start, stop, destroy)
- Runtime resource resizing via PATCH endpoint
- Time-bounded CPU/memory boost with auto-revert
- Snapshot and restore functionality
- Copy-on-write cloning on btrfs/XFS/bcachefs
- Memory CoW fork for Firecracker
- Interactive persistent shell sessions
- Command execution with PTY support
- Port forwarding
- WebSocket event streaming
- OpenTelemetry metrics and tracing
- Project-based sandbox organization
- MCP server integration for AI agents

## Backend Comparison

**Incus (Containers):** ~1s startup, minimal overhead, namespace/cgroup isolation. Best for dev environments, CI runners, trusted workloads.

**Firecracker (MicroVMs):** 2-3s startup, ~30MB overhead, hardware (KVM) isolation. Best for untrusted code, multi-tenant environments.

## Architecture
CLI client, daemon (navarisd), SQLite persistence, async operation dispatcher, OpenTelemetry observability. Lightweight guest agent (navaris-agent) in Firecracker VMs communicating over vsock.

## Technology
Go (78.6%), TypeScript (17.1%). Docker image with both backends bundled. Apache License 2.0.
