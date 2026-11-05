# Hyperbutler

hyperbutler is an Apache-2.0, Docker-compatible container engine for Windows 11 that boots a small Alpine Linux VM directly through Hyper-V's Host Compute Service (HCS) — the API under WSL 2 — without WSL or Docker Desktop. It runs real dockerd and containerd in that VM, fronts them with a per-user Go engine, and exposes everything to the stock `docker` CLI and Compose via a docker context. What makes it worth studying is not "another Docker Desktop": it is a ~130K-line Go codebase built security-first around a strict privilege split, a hand-rolled FUSE file server, a gVisor network stack with egress policy, and an unusually thoughtful MCP server for AI agents — plus a proposed design for a prompt-injection-resistant agent sandbox.

---

## Architecture

Three processes, one privilege boundary (ADR 0004):

- **Engine** (`cmd/hyperbutler`, `internal/engine`, ~47K lines in `internal/`): runs as the user. Relays the Docker Engine API over a named pipe (`internal/dockerapi`, a minimal hand-written API client with bounded response bodies), runs the guest's network in user mode (`internal/netstack`), serves bind mounts (`internal/share`), and guards host disk space (`internal/spaceguard`).
- **Broker service** (`cmd/hyperbutler-broker`, `internal/broker`): the only elevated component. A Windows service that starts and stops the VM through HCS in response to exactly seven typed requests over a named pipe only the owner can open. Its `VMSpec` is built purely from broker state and typed request fields — "nothing in it is a caller-supplied path, command line or document" (`internal/broker/server.go`). Identity comes from the pipe's client token, never from a message.
- **Guest**: a small Alpine VM with dockerd/containerd on a read-only, hash-pinned system disk (`internal/vmtemplate`, `guest/`) and a sparse data disk that returns freed space to Windows.

## Key techniques

- **FUSE-over-hvsock file server** (`internal/share/proto.go`, `fs.go`, `inject.go`, ADR 0007). Rather than accept HCS Plan9 shares (measured 14–2000× slower than ext4, zero change notifications) or virtio-fs (not offered by HCS), hyperbutler relays the guest kernel's FUSE protocol unchanged over four hvsock connections, decodes every request with checked lengths/offsets, and serves NTFS relative to a share-root handle without ever following a link. Windows changes are pushed back as FUSE invalidations plus replayed operations so inotify fires in 2–4 ms — the reason Vite/webpack watch cycles work.
- **User-mode network stack** (`internal/netstack`, ADR 0006): a gVisor TCP/IP stack assembled from gvisor-tap-vsock parts *without* its VirtualNetwork, because hyperbutler needs a `Policy` check on every guest outbound connection (guest cannot reach host loopback/LAN/VPN/metadata addresses unless explicitly forwarded), full resolver-mediated DNS for VPN split-DNS, runt/oversized frame rejection, and bounded connection/UDP-flow counts. No DHCP — address, route and resolver ride the kernel command line.
- **Spaceguard** (`internal/spaceguard`): the subtlest piece. A sparse VHDX that fills its host volume gives the guest EIO, which aborts the ext4 journal and loses in-flight data. The guard keeps an `fallocate`'d reservation in the guest and maintains a proven invariant — guest "exposure" (device size minus reservation minus already-backed allocation) must stay ≤ host free space minus a margin — as pure overflow-checked uint64 arithmetic. The result is the README's "disk space comes back by itself."
- **MCP server as a security boundary** (`internal/mcpserver`, ADR 0008/0009). Read tools (status, containers, logs, disk) ship enabled; writes need opt-in flags; and every string produced by dockerd, the guest, a registry or Windows travels only in a dedicated `untrustedText` field (`untrusted.go`), capped at 4 KiB and redacted — so a malicious container's log output is visibly labeled as untrusted to the consuming model. `configpolicy.go` is a reviewed allowlist where a *test fails* until every new config key is classified; refused keys each carry a reason ("a proxy chosen by text the model read could see every pull").
- **Leak discipline** (`internal/leaktest`): every package's tests fail on a leaked goroutine and a 24-hour soak (261,338 docker commands, zero failures) gates releases — the measured justification for Go in ADR 0003 (engine idles at ~18 MB).

## Design decisions

- **Privilege minimization over convenience**: the whole reason for the broker is that daily use needs no admin rights and no Hyper-V Administrators membership. The cost is an extra process and a fixed seven-request protocol that cannot grow without review.
- **Own the VM, own the plumbing**: because hyperbutler controls HCS, it can reject Plan9 shares, skip DHCP, and pick FUSE itself — trading significant implementation effort (a FUSE server, a network stack) for measured 10×+ wins and inotify support neither WSL-based path provides.
- **Agent threat model = prompt injection, not filesystem damage** (ADR 0009, proposed): the argued insight is that isolating the filesystem doesn't stop an injected instruction from exfiltrating; only control of outbound traffic does. Hence a proposed one-VM-per-session sandbox keyed to egress policy, reusing the broker's hash-pinned template, the read-only system disk, and the policy-checked netstack. Notably they rejected container-in-the-Docker-VM precisely because handing an agent the Docker socket hands it the whole engine.
- **Bounded everything**: TCP buffers, MCP pull concurrency, response bodies, process counts in MCP-launched containers — a consistent "the guest and the model are untrusted" posture.

## Comparison notes

- Against Docker Desktop it is deliberately narrower (no Kubernetes, Windows containers, or extensions) and wins on footprint, disk reclamation, and `C:\` bind-mount latency; one machine of benchmarks, honestly flagged as such.
- Its MCP design — untrusted-text quarantine, config allowlist with per-key refusal reasons, capability flags that users must add — is a sharper-edged concrete instance of the enforcement themes in [[Reliability Is Outside the Model (O'Reilly)]] and the gateway-authorization pattern in [[Architectural Blueprints for AI Agents (Nokia Networks)]], and a working counterpoint to the tool-trust failure catalog in [[AI Agents In-Depth — Function Calling, MCP and Tool Use Under the Hood]].
- Its proposed agent sandbox answers the same "agents run untrusted commands" problem as [[Self-Hosted Sandboxed Agentic Software Factory]] from the opposite direction: that project isolates by giving the whole machine no ingress and sacrificing it; hyperbutler isolates per-session with kernel boundaries and egress control on a dev workstation. And where [[The Archaeologist's Copilot]] uses Docker as a convenient containment blob for modernization work, hyperbutler is building the containment into the engine itself.

---
*Sources: [[raw/hyperbutler]], [[summary/hyperbutler]]*
*Last updated: 2026-11-06*

#tool #project #agents #containers #windows #sandboxing #mcp
