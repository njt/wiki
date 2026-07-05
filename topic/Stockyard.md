# Stockyard

Jesse Vincent's greenfield design for an agent sandbox orchestration tool: spin up 1 to hundreds of Firecracker micro-VMs, inject credentials via cloud-init, run Claude Code in YOLO mode, and capture every filesystem state change via ZFS snapshots triggered over vsock. The entire architecture emerged from a single Claude Code conversation — the 484-line design doc was the first commit.

---

## Key Quotes

> "Spin up anywhere between 1 and, if the hardware supported it, hundreds of lightweight containers or micro-containers to run coding agents like Claude Code."

This isn't a sandbox for one agent. It's a launchpad for fleets. The ambition is industrial-scale agent execution on a single machine, which puts it somewhere between [[klaw.sh]] (Kubernetes-style agent lifecycle) and [[Crabbox]] (throwaway remote compute per run).

> "The Firecracker ecosystem has standardized on Go for client tooling."

A practical concession to ecosystem gravity. Jesse wanted Rust but the tooling reality — Flintlock, Firecracker SDK, the surrounding infrastructure — all speaks Go. Smart choice for a project that needs to integrate deeply with existing VM machinery.

> "ZFS snapshots triggered from inside the VM via vsock, called by agents between tool calls for full audit trails."

The killer feature. Most agent sandboxes track what the agent *outputs*. Stockyard tracks what the agent *did to the filesystem*, at near-instant granularity. This makes every tool call auditable and every state restorable. It's a stronger audit primitive than anything in [[yolo-cage]], [[OpenSandbox]], or [[Navaris]].

## Key Themes

#agent-sandboxing #microvm #orchestration #security #design-session

### Firecracker Over Docker

The design explicitly rejects Docker containers in favor of Firecracker micro-VMs via Flintlock. The research chain is documented in the transcript: Ignite (dead, archived Dec 2023) → raw Firecracker (viable but painful) → Flintlock (best option). This is the same hardware-isolation bet that [[OpenSandbox]] makes with its Firecracker support and that [[Navaris]] abstracts behind a unified API. Stockyard goes all-in on micro-VMs as the only primitive.

### vsock as the Secret Weapon

The vsock (virtual socket) channel between guest and host is what makes the snapshot architecture work. Agents inside the VM can't access the host filesystem directly — but they *can* signal over vsock to trigger a ZFS snapshot. This is an elegant separation of concerns: the agent owns execution, the host owns persistence. Compare with [[VTcode]]'s tree-sitter-based command validation — different layer of the stack, same philosophy of architectural constraint over prompt-level pleading.

### Secrets as Namespaced Infrastructure

Secrets follow a clean `op://Stockyard/<instance>/<key>` hierarchy in 1Password, with an abstraction layer for future migration to AWS Secrets Manager. Cloud-init injects them at VM boot. The agent never sees real credentials in its configuration — it just uses them. This is the same pattern as [[OneCLI]]'s transparent proxy injection, applied at the VM provisioning layer rather than the network layer.

### Agent-Designed Architecture

The design doc itself is a Claude Code artifact — 484 lines committed as the initial commit. This isn't just using AI to write code; it's using AI to design the system that runs AI agents. The recursion is the point: the tool that constrains and orchestrates agents was itself designed in conversation with an agent. See [[How Boris Uses Claude Code]] for the creator's perspective on the tool being used to design its own ecosystem.

## Critical Analysis

**The ZFS dependency is both the superpower and the lock-in.** ZFS snapshots are genuinely the right primitive for agent audit trails — near-instant, space-efficient, mountable, restorable. But ZFS on Linux has always been a second-class citizen compared to its FreeBSD/Illumos heritage, and requiring it narrows the deployment surface considerably. For a tool targeting "any Linux machine," this is a real constraint.

**The "hundreds of VMs" ambition may collide with Firecracker's memory model.** Firecracker VMs are lightweight (~30MB overhead) but not free. A hundred VMs means 3GB just in VM overhead before any workload runs. Stockyard needs a scheduler that understands memory pressure, not just a spawn loop. The transcript doesn't address this — it's the kind of gap that emerges when the design doc is a first-pass artifact rather than a battle-tested spec.

**Flintlock is a bet on a moving target.** It's the right choice among available options, but it's also a relatively young project in the Firecracker ecosystem. The decision matrix (Ignite dead, raw Firecracker painful, Flintlock best) is correct in January 2026, but the space is evolving fast. Compare with [[Navaris]] which hedges by abstracting over both Incus containers and Firecracker microVMs — a safer architectural bet if you're not sure which isolation primitive wins.

**What's genuinely novel here is the vsock→ZFS audit trail.** No other agent sandbox tool does filesystem snapshots triggered from inside the guest at per-tool-call granularity. This is the contribution that would survive even if the rest of the architecture changes. If I were building an agent sandbox in 2026, I'd steal this pattern and adapt it to whatever filesystem I had available.

**The transcript itself is a valuable artifact** — a real, unedited design conversation between a senior engineer and Claude Code, making concrete tradeoffs about language choice, runtime selection, and architecture. It's a primary source for how agent-assisted system design actually works in practice, and it's more honest than most "how I use AI" blog posts because it shows the dead ends and research chains.

---

*Sources: [[summary/stockyard-design-session]]*
*Last updated: 2026-05-15*
