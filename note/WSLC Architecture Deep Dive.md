# WSLC Architecture Deep Dive

Microsoft's technical companion to the WSL Containers GA announcement: how `wslc.exe` actually works under the hood, through three architectural changes — a privilege-splitting session model, a virtiofs/VHD storage design, and the Consommé networking mode.

---

## What it argues

WSLC is not WSL with a container CLI bolted on; it restructures the trusted-computing boundary. The privileged Windows service (`wslservice.exe`) creates the VM but immediately hands ownership to a per-user child process (`wslcsession.exe`) that does all the actual work — container creation, mounts, port binding. Sessions are isolated from each other by process boundaries and from the system by privilege boundaries.

Storage is per-session VHDs under `%AppData%\Local\wslc\sessions`, with container volumes implemented as virtiofs shares (claimed ~2× plan9) bind-mounted into containers, and VHD-backed volumes for cases that need a native ext4 filesystem or a size cap.

Networking runs through "Consommé": the Linux VM emits ethernet frames into a virtio queue, and a per-user Windows process provides DNS, routing, and port mapping — so container egress traffic flows as if from an ordinary Windows process, inheriting VPN and firewall compatibility.

---

## Key quotes

> "wslservice.exe does not retain ownership of the virtual machine. Instead, it creates a child process, wslcsession.exe, which runs on behalf of the calling user."

Commentary: this is the least glamorous but most consequential paragraph in the post. A service that creates VMs but never operates them is a textbook privilege-separation pattern — the kind of thing you expect from OpenSSH, not from a developer convenience tool. It signals Microsoft is designing WSLC for enterprise threat models, not just developer ergonomics.

> "Compared to plan9, virtiofs is about twice as fast."

Commentary: the quiet admission that WSL's famous filesystem pain was an architectural dead end. Plan9 served a protocol purpose (SMB-style interop) but virtiofs is purpose-built for hypervisor-to-VM sharing. The 2× figure is modest as benchmarks go — but this is Windows-to-Linux file I/O, where 2× moves workflows from "painful" to "usable."

> "The traffic that's routed out of the virtual machine is sent on behalf of the user owning the wslc session, so traffic flows as it was emitted by a regular Windows process, which provides extensive compatibility with VPNs and firewalls."

Commentary: this is the real product insight. Corporate VPN/firewall incompatibility has been the single most common reason IT departments block WSL and Docker Desktop. By making container egress indistinguishable from ordinary user traffic, Consommé dissolves the objection — and it's also an accountability win: network policy applies to containers because containers can't opt out of the host's identity.

---

## Key themes

#tool #pattern #microsoft #container #sandboxing #windows

**Privilege separation as a first-class design goal.** The session model is the same discipline [[A Deep Dive on Agent Sandboxes]] and the isolation papers argue for: keep the privileged component minimal, do work in a less-trusted context. Microsoft arrived at this for container sessions; agent-sandbox designs arrive at the same shape for agent sessions.

**Egress-as-user networking.** Consommé's trick — traffic exits with the user's identity and inherits host network policy — is the same conceptual move as capability-scoped networking for agents: make the boundary ambient rather than configured. It's also why [[WSL Containers]] can credibly claim enterprise readiness where Docker Desktop needed per-VPN workarounds.

**The filesystem fix is substrate, not feature.** virtiofs doubles Windows↔Linux I/O, which matters for anyone bind-mounting working directories into containers — including agent harnesses that mount a repo into a containerized workspace.

---

## Critical analysis

The post is honest about architecture and thin about trade-offs. Per-user session processes improve isolation, but the post never addresses resource accounting: what happens to a machine with dozens of concurrent sessions, each with its own VHD and its own networking relay process? The comment thread's headline question — compose support deferred — suggests the team is still filling out the workflow surface even as the substrate ships.

The networking design also has an unexamined cost: because all egress flows through a single per-user Windows process, that process is both a performance bottleneck and a privileged-ish chokepoint for every container the user runs. The post presents "traffic looks like it came from a normal Windows process" purely as a compatibility win; it is also a loss of per-container network identity, which matters if you wanted per-container firewall rules.

For the agent-sandbox angle, WSLC is quietly significant: a per-user, API-accessible, privilege-separated Linux container runtime shipping in the OS means Windows-based coding agents get container isolation without third-party tooling — the Windows counterpart to the Linux sandbox stacks described elsewhere in this wiki.

---

This deep dive substantiates the architectural claims of [[WSL Containers]]: the GA announcement promised enterprise-ready isolation and Consommé networking, and this post supplies the mechanism (privilege-split session processes, virtiofs, per-user egress). It complicates [[Windows in Docker]]: where Vincent's stack runs Windows inside Linux containers to test Windows-specific behavior, WSLC is the inverse play — Linux containers as an OS-native Windows capability — and the two together complete the cross-platform container matrix. And it strengthens [[A Deep Dive on Agent Sandboxes]] with a concrete production example of the privilege-separation pattern (privileged creator, unprivileged operator) that agent-sandbox designs converge on.

---
*Sources: [[raw/wslc-architecture-deep-dive]], [[summary/wslc-architecture-deep-dive]]*
*Last updated: 2026-10-03*
