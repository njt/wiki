---
url: https://github.com/perplexityai/SPACE
title: "SPACE (Sandboxed Platform for Agentic Code Execution)"
author: Perplexity AI
date_fetched: 2026-10-18
date_published: 2026-10
topics:
  - security-and-sandboxing
  - coding-agents-and-frameworks
---

Perplexity's open-source (Apache 2.0) sandbox platform for running agent code in isolated Firecracker microVMs. Applications use an HTTP/SSE API to create Linux environments from **templates**, run command **sessions** against them, and manage the sandbox lifecycle (create, pause, suspend, wake, delete) — with lifecycle changes as asynchronous operations you poll and command output on a separate data-plane connection with session-scoped attachment tokens.

The architecture is a control/data-plane split over a Rust microservice fleet (~800K lines): API + PostgreSQL hold desired state; controllers reconcile it through either a direct adapter or a Kubernetes adapter (Kind for local dev); worker services drive Firecracker VMs, btrfs copy-on-write root filesystems, volumes, and networking; a separate data plane connects clients to guest processes. An in-guest daemon (`spaced`) bridges host and VM. Notable pieces: warm pools and base snapshots (restore from a booted-VM snapshot instead of cold boot), encrypted suspend/restore of VM memory, and an **E2B Compatibility Wrapper (ECW)** that translates the E2B Python SDK onto the native API so existing agents switch backends.

The network layer is unusually deep for a sandbox product: a gateway applies `NetworkPolicyV1` egress rules (CIDR, domain-with-DNS-attribution, UDP/TCP), intercepts TLS with SNI-based exemptions, and **injects credentials into outbound guest HTTP requests** at the proxy — the sandbox never holds the secret. Smoke suites in `space_smoke/` test scenario invariants like cross-node snapshot restore, credential injection, and network-policy ordering.
