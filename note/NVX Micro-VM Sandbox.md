# NVX — Microsoft's Micro-VM Sandbox for Untrusted Workloads

NVX (https://github.com/microsoft/nvx) is a research-grade micro-VM sandbox from MSR Systems and Azure Research – Systems, built on OpenVMM and descended from Nanvix. It boots a Linux-direct kernel with no firmware into a single-workload container with hardware-enforced isolation, and it is explicitly aimed at the problem this wiki tracks constantly: how to run agent-produced or agent-adjacent code without trusting it. Unlike most sandbox write-ups here, this one ships the code, the ABI, and the threat model.

---

NVX's four stated goals: boot the same uncompressed Linux kernel with Alpine or Ubuntu userland on Linux *and* Windows; keep the guest-visible machine independent of the hypervisor (KVM, MSHV, WHP); expose only a fixed, allowlisted device set; and snapshot a running VM into immutable artifacts restorable in a new process without serializing host handles. Everything in the design follows from treating the "microVM machine profile" — ABI 2, boot layout 2 — as a versioned contract selected explicitly, never inferred.

## Architecture

Three layers:

- **Harness** — `scripts/nvx.py` (~1,300 lines of Python) is the supported entry point: download prebuilt packages, build, run, benchmark. It resolves releases by "nearest released first-parent ancestor" of the cloned revision, so a checkout always maps to a usable build without local compilation.
- **Rust lifecycle crate** — `aci_edge_sandboxes/src/lib.rs` exposes exactly five operations: `provision`, `start`, `exec`, `stop`, `deprovision`, behind a pluggable `Backend` trait whose default implementation drives the `openvmm` binary. Validation is layered and ordered — structural errors, then policy (`PolicyValidation` via a `Capabilities` honor matrix, `src/capabilities.rs`), then backend failures — and "rejected requests never run anything."
- **Guest** — a shell/C initramfs agent assembles the sandbox and supervises the workload; the C agent (`guest/common/nvx-managed-agent.c`, ~2,300 lines) is the control plane inside the VM.

The guest filesystem bootstrap (doc/design/sandbox-filesystem-and-agent-architecture.md) is the cleverest bit: one to three role-bearing EROFS lower images (`distro`, `runtime`, `custom`) plus a preformatted ext4 scratch are assembled into an overlayfs rootfs. The init agent resolves each block device by *its MMIO address from `/proc/iomem` and the virtio sysfs tree* rather than assuming `/dev/vdX` naming — enumeration-order bugs are eliminated by construction. The workload runs in private mount/PID/UTS namespaces behind a FIFO barrier, with cgroup v2 placement (`memory.low` 16 MiB default, optional `memory.max`/`pids.max`), cleared capabilities, `no_new_privs`, and one host-owned non-root numeric identity. The agent stays in the initramfs, outside the workload's namespaces — the supervisor never shares a fate with the supervised.

## Key techniques

- **Hypervisor-agnostic machine contract.** The profile owns boot protocol, memory map, device topology, and snapshot compatibility; hypervisor backends only supply partition creation, vCPU execution, and interrupt injection. Portability across KVM/MSHV/WHP is a compile-free consequence, not a porting effort.
- **Tiered snapshots.** Three tiers — `platform` (clone, fresh scratch, invariants only), `workload-start` (clone, paired scratch, image binding), `instance-checkpoint` (resume, single-use claim) — validated as strict combinations. Capture is guest-initiated via PMIO `0x605` and coordinated as a bounded transaction: the vCPU stops at the instruction immediately after the `out`, so saved state is deterministic. Failure after the vCPU stops is *terminal* — the VM is torn down rather than resumed with uncertain state.
- **A framed control protocol with credit-based flow control.** The managed agent speaks a two-layer protocol (44-byte outer frames with 1 MiB receive window; 24-byte app messages: PING/EXEC/STOP/CANCEL/FEATURES, with STDOUT/STDERR/EXIT/ERROR responses) and advertises a feature bitmask so hosts refuse images lacking features they need — version negotiation by refusal, cheap and honest.
- **Trust boundary discipline.** doc/design/concurrency-and-trust-boundaries.md treats the guest's own PMIO accesses, virtio descriptors, and even the host's control-console clients as untrusted: checked address arithmetic, bounded buffers, rate-limited guest-triggerable logs, fail-closed record validation, and a launcher-provided capability that never appears in arguments, env, logs, or snapshots.
- **Copilot as a bounded adversary.** The adversarial-testing design (doc/design/copilot-adversarial-testing.md) is the most directly relevant thing here for agent practitioners: an LLM (Copilot CLI) selects stress probes, but it may only return `{"schema_version":1,"case_id":"..."}` — a catalogued case ID, nothing else. No tools, no MCP, no repo instructions, truncated base64-encoded guest output in prompts, credit and wall-clock budgets, a typed broker rejecting duplicate/extra/unknown properties, and a separate watchdog. The LLM proposes from a menu; deterministic code decides and executes. It's a template for "model-as-strategist, broker-as-enforcer" testing of any containment system.

## Design decisions

The central bet is **specialization over generality**: exactly one workload container per VM, so no pod infrastructure, no pause container, no dynamic rootfs injection, no container-creation RPCs. That trades flexibility (no multi-container groups, no confidential computing — the host is trusted with image content and guest memory) for a radically smaller attack surface and simpler lifecycle.

Second: **fail closed, everywhere.** Capture destinations must not pre-exist; malformed records fail closed; partial restore means teardown. They gave up resumability in edge cases for the guarantee that no ambiguous state ever executes.

Third: **honesty about maturity.** Design docs rigorously mark Proposed vs implemented — the production conversion service, Rust agent, and replaceable launch config are all "Proposed," and the docs explicitly warn against inferring production features "from the presence of block devices alone." Most sandbox projects oversell; NVX documents its own incompleteness as part of the spec.

## Comparison notes

- Against [[A Deep Dive on Agent Sandboxes]]'s survey of gVisor/Firecracker/container tiers, NVX is a fourth data point: a purpose-built microVM profile that is *lighter* than Firecracker's feature set but heavier than a namespace jail, and — uniquely here — Windows-hostable via WHP.
- [[How We Contain Claude]] argues containment must move from prompts to infrastructure; NVX is the infrastructure end of that argument taken to its logical conclusion, down to denying the guest trustworthy time and logging.
- [[Stockyard]] also drives Firecracker-style micro-VMs for agents but optimizes for fleet-scale filesystem auditing via vsock→ZFS; NVX optimizes for a minimal, versioned ABI and snapshot tiers instead — fleet orchestration is deliberately out of scope.
- The Copilot-strategist harness complements [[LLM-as-a-Verifier]]-style ideas: instead of the model judging output, the model selects adversarial inputs under a broker that makes arbitrary model creativity structurally impossible.

#tool #project #agents #security #sandboxing #microvm

---
*Sources: [[raw/nvx]], [[summary/nvx]]*
*Last updated: 2026-10-08*
