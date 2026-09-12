---
url: https://github.com/agent-substrate/substrate/
title: Agent Substrate
author: Google (not officially supported)
date_fetched: 2026-08-08
topics:
  - agent-orchestration
  - security-and-sandboxing
---

# Agent Substrate — Summary

Agent Substrate is Google's open-source Kubernetes-based runtime for deploying AI agents at scale. Rather than running each agent as a persistent pod (which wastes resources on idle workloads), it multiplexes many "actors" onto a smaller pool of pre-started "worker" pods using **sub-second suspend/resume** via gVisor or micro-VM checkpoint/restore.

The system has six components: an **ate-api-server** (gRPC control plane backed by Redis/ValKey for high-frequency state), an **atecontroller** (Kubernetes controller reconciling WorkerPool and ActorTemplate CRDs), an **atelet** (node-level DaemonSet supervising snapshot lifecycle and OCI image preparation), an **ateom** (sandbox herder running inside each worker pod, driving runsc or Cloud Hypervisor), **atenet** (DNS + Envoy-based proxy for actor-aware routing and on-demand wakeup), and **kubectl-ate** (CLI).

Key architectural decisions: (1) dual-layer state model — CRDs for slow-changing config (WorkerPool, ActorTemplate) vs. Redis for fast-changing dynamic state (Actor, Worker records); (2) sandbox binaries not baked into worker images — fetched at runtime and pinned into snapshot manifests for reproducible restores; (3) self-describing snapshot manifests that include the exact sandbox binary versions; (4) concurrent download + OCI unpack during restore to minimize latency; (5) golden snapshots for template-level fast-start, plus DATA_ON_GOLDEN restore for micro-VMs where durable data is served atop a shared golden memory image.

North star metrics: 100ms P95 activation latency, 1 billion actors per cluster, 1,000 wakeups/second. The project is in early development (not production-ready, APIs will change).
