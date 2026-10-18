# SPACE — Perplexity's Firecracker Sandbox Platform

SPACE (Sandboxed Platform for Agentic Code Execution) is Perplexity's open-source (Apache 2.0) infrastructure product for running agent code in hardware-isolated Firecracker microVMs. Applications create sandboxes from templates over an HTTP API, run command sessions, and manage lifecycle (pause/suspend/wake/delete) — the same job E2B sells, but self-hostable and with a far deeper control plane. What makes it worth reading is not "another microVM wrapper": it is the operating discipline around the VMs — durable desired state, snapshot-based warm starts, encrypted suspend/resume, and a network gateway that treats agent egress as a first-class policy problem.

---

## Architecture

Rust workspace, ~800K lines across `crates/apps/` (service binaries) and `crates/libs/` (shared libraries), orchestrated as a microservice fleet:

- **Control plane**: `crates/apps/api/svc` records desired state in PostgreSQL (`docs/architecture.md`: "an accepted request is not a readiness signal"); `crates/apps/cluster/controller/svc` reconciles it. Reconciliation is pluggable — `crates/libs/api/src/controllers/substrate/direct.rs` calls worker services directly (no Kubernetes), while `substrate/kubernetes/mod.rs` drives sandbox pods through a SPACE runtime shim. Kind-based local dev uses the latter; AWS EKS is the reference deployment.
- **Workers**: `node/mgr`, `sbx/mgr`, `vol/mgr`, `template/mgr` manage VMs, btrfs CoW root filesystems, volumes, and template builds. The sbx-manager alone is ~185K lines, including `snapshot_encryption.rs`, `rootfs_encryption.rs`, `auto_lifecycle.rs`, and `restore_operation.rs`.
- **In-guest daemon**: `crates/apps/spaced/guest/svc` runs inside every microVM, bridging host and guest via a gRPC protocol (`proto/`). Its main.rs has a telling detail: a `--hold-cgroup` mode that parks a thread to keep the systemd cgroup alive because `sleep` may not exist in the guest image.
- **Data plane**: `crates/apps/data/plane/svc` connects clients to guest processes with session-scoped attachment tokens, deliberately separate from management tokens and from the API's lifecycle.
- **Network gateway**: `crates/libs/net/gateway/server/src/tcp_proxy/` is a full transparent proxy with TLS interception, egress policy, and credential injection.
- A Next.js dashboard (`ui/`) for operators; API clients don't need it.

Core abstractions from `docs/architecture.md`: **template** (immutable starting environment, referenced by release or digest), **sandbox** (a microVM instance with writable filesystem), **command session** (connection info + authorization; creating assigns a process ID, attaching starts it), and **operation** (async lifecycle record you poll). The insight baked into the design: a sandbox, a lifecycle operation, and a command stream have three separate lifetimes — a sandbox outlives both the request that created it and the connection carrying its output.

## Key techniques

- **Snapshot-based fast start.** Two accelerators: Kubernetes **warm pools** of pre-booted sandboxes per template release/VM size (`docs/warm-pools.md`), and **base snapshots** — a booted template's memory and VM state saved so creates restore instead of booting (`docs/base-snapshots.md`). Suspend serializes VM memory + runtime state + rootfs to object storage; wake restores it, including cross-node (`space_smoke/tests/test_cross_node_restore_scenario.py`).
- **Encryption at rest for VM state.** `snapshot_encryption.rs` and `rootfs_encryption.rs`; the smoke suite tests "encrypted resuspend preserves CoW" and "encrypted double restore retires prior pods" — suspend/restore is treated as a security boundary, not just a performance feature.
- **Credential injection at the egress proxy.** `crates/libs/net/gateway/server/src/tcp_proxy/tls_http/credentials.rs` injects credential headers into inspected guest HTTP requests from a lease-based credential service — the sandbox never stores the secret. Combined with TLS interception with SNI exemptions that require a *live observed DNS binding* before exempting a hostname, this is defense against a prompt-injected guest exfiltrating credentials or pinning a fake DNS entry.
- **`NetworkPolicyV1` egress model**: deny-before-allow ordering, platform guardrails that precede tenant rules, domain allowances requiring guest DNS attribution, and rules that survive snapshot restore — all covered by dedicated smoke scenarios.
- **Exhaustive drain semantics.** The architecture doc's SIGTERM section specifies per-request-type drain behavior down to "a `Process.Start` that hasn't reported itself gets up to 2 seconds… otherwise ends with `unknown`, since the command may be running and a retried start would run it twice" — idempotency reasoning applied to process spawning.
- **Lifecycle fencing.** Timeout updates use "durable execution ownership" and execution-generation fences, comparing observed generation atomically with resume admission, so a stale controller can't reset a retired sandbox's clock.
- **Testing as scenario library.** `space_smoke/` is ~100 named scenario tests (`native_recovery_command_survives_data_plane_restart.py`, `local_btrfs_send_restore_rejects_corrupted_stream.py`) asserting invariants over the whole deployed system, plus an opt-in containment benchmark against real nftables rules.

## Design decisions

- **Control/data-plane split at the API boundary.** Command output never traverses the management API; the session-scoped attachment token means a leaked management credential cannot attach to a running process stream, and vice versa. Cost: clients juggle two connections and an attach step.
- **Desired state in Postgres, not in-memory orchestration.** Every lifecycle action is an asynchronous, pollable operation. This buys restartability and auditability at the price of "create" not being synchronous — most simpler sandboxes (E2B, SmolVM) return ready VMs.
- **Kubernetes optional, not required.** The direct adapter runs workers bare, and single-node users get base snapshots instead of warm pools. Deploying this on a laptop without k8s is a supported path, rare for this category.
- **Separation of template builds from user sandboxes** — builders are isolated services, and `SPACE_TRUSTED_TEMPLATE_STORAGE_BUCKETS` pins which S3 buckets may publish releases (supply-chain control on the template path).
- **Committed breadth over a minimal core.** This is a platform (UI, admin CLI, operator docs, egress profiles, ECW), not a library. The cost is weight: ~800K lines where [[SmolVM]] is a single binary.

## Comparison notes

- [[NVX Micro-VM Sandbox]] is the closest architectural cousin — Microsoft's OpenVMM microVM-per-agent with tiered snapshots — but SPACE goes further on the network layer (egress policy + TLS interception + credential injection as built-ins rather than adjacent systems).
- [[Nango — Running Untrusted Customer Code at Scale]] ended up on tenant-pinned Firecracker (Lambda); SPACE is essentially the productized version of that endpoint, with warm pools attacking the cold-start problem Nango inherits from Lambda.
- [[SmolVM]] optimizes for <200ms boot and library-style embedding; SPACE optimizes for fleet operation — durable state, snapshot restore, warm pools — accepting cold-boot latency mitigated by snapshots rather than eliminated by a tiny VMM.
- [[Designing Agent-First Platforms (Microsoft Foundry)]] describes per-execution hardware-isolated microVMs with pause/resume as a managed service; SPACE is the open-source, self-hosted expression of the same control-plane/execution-split thesis, with the E2B Compatibility Wrapper as its adoption wedge.
- [[How We Contain Claude]] catalogs three isolation patterns and their unanticipated failures; SPACE's smoke suites read like the engineering response — testing cross-node restore races, corrupted snapshot streams, and credential-digest probing as first-class scenarios rather than afterthoughts.

#tool #project #agents #sandboxing #firecracker #virtualization #networking

---
*Sources: [[raw/space]], [[summary/space]]*
*Last updated: 2026-10-18*
