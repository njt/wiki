# SmolVM

smolvm is a single-binary, open-source (4.6k GitHub stars) microVM runner that gives every workload its own hardware-isolated Linux VM — Hypervisor.framework on macOS, KVM on Linux, Windows Hypervisor Platform on Windows — with <200ms boot, a VMM that's a *library* (libkrun) rather than a daemon, and a portable `.smolmachine` artifact that runs identically in local dev, on the hosted cloud, or self-hosted.

---

## Key Quotes

> "Run any workload in a fast, hardware-isolated Linux VM. The same SDK and the same `.smolmachine` artifact run identically: Develop locally, then deploy to the hosted cloud — or self-host with smolvm."

The one-line pitch. The interesting word is *artifact*: containers have images but still need a daemon to run them, and Firecracker has no artifact story at all — smolvm is claiming a self-contained, portable unit of deployment that carries its own isolation.

> "network is off by default — untrusted code can't phone home"

Default-deny at the network layer, with per-host egress allowlisting as the escape hatch. This is the same conclusion [[How We Contain Claude]] reaches after its allowlisted-egress incidents, and the opposite of the "allow the provider API and hope" posture that produced [[Attack Review -- Claude Allowlisted-Egress Exfiltration]].

> "No daemon — the VMM is a library linked into the smolvm binary."

The architectural claim that separates smolvm from Firecracker and QEMU, both of which run as a separate process you have to supervise. A library VMM means there's no long-lived privileged daemon to compromise — the isolation boundary is a hypervisor, but the orchestrator's attack surface collapses to a single short-lived process.

> "GPU — Yes (Vulkan) … macOS native — Yes"

The two rows in the site's comparison table where smolvm outpaces Firecracker, which the rest of the industry is standardising on ([[Best Infrastructure Platforms for Coding Agents in 2026]]). Vulkan access inside an isolated VM, and first-class macOS support, are both things the Firecracker path simply doesn't offer.

## Key Themes

#tool #concept #sandboxing #virtualization #microVM #pattern

**Library VMM, not a daemon.** libkrun links into the binary; the isolation comes from the hypervisor, but there's no always-on privileged process to attack. A quieter blast radius than the daemon model.

**Portable artifacts.** `.smolmachine` is the "container image that runs without a daemon" — pre-baked dependencies, <200ms boot, one binary to ship.

**GPU in the isolated VM.** Vulkan passthrough is the missing piece in the Firecracker story, and it matters for anything agentic that wants to screenshot a browser or run a headless Chromium.

**Cross-platform parity.** macOS-native rather than macOS-second — relevant precisely because [[Security and Sandboxing]] notes almost every sandboxing tool in the ecosystem is Linux-first and macOS-second.

## Critical Analysis

The pitch lands hardest on the exact gap the broader sandboxing survey keeps flagging: **macOS parity**. [[Security and Sandboxing]] ends its "What's Missing" section on the observation that nearly every tool is Linux-first and macOS-second, and that the macOS/Linux guarantee gap is "a real vulnerability that no tool currently closes." smolvm's answer — Hypervisor.framework on the Mac, KVM on Linux, WHP on Windows, one binary across all three — is a direct shot at that gap. But the table is marketing, and should be read like the Modal survey in [[Best Infrastructure Platforms for Coding Agents in 2026]]: vendor-written, self-serving, with the vendor's product winning every row.

Two honest blemishes stand out. The macOS Intel path is marked "**untested**" — a one-word admission that's easy to miss and that undercuts the "runs anywhere" claim for a whole class of still-common hardware. And the comparison table's "Boot time: <200ms" vs. "Containers: ~100ms" quietly concedes that a container is still *faster* than smolvm's VM — the real trade being made is isolation and portability, not raw latency.

The `.smolmachine` artifact is the genuinely novel claim. Containers need a daemon; Firecracker has no artifact story; smolvm proposes a deployment unit that carries its own hypervisor boundary and boots in under 200ms. Whether that survives contact with real workloads — networking defaults, snapshotting, multi-machine orchestration — the landing page doesn't say. It's a primitive, not a platform.

The positioning against [[Smolbox]] is instructive: both are "smol," both are client-local, and they point in opposite directions. Smolbox compiles a whole VM to WebAssembly so the sandbox, model, and tool calls live in one browser tab with *no* server and *no* credentials. smolvm runs a real hypervisor on real hardware — more capable, GPU included, but still a thing you install and run. Smolbox dissolves the provisioning problem; smolvm gives you a better hammer for the machine you already have. [[QM (Multiplayer Agent Harness)]] already treats smolmachines as one of its four sandbox backends, which suggests the library shape — a binary you can route to, rather than a platform you rent — is the part teams actually want.

---

*Sources: [[raw/smolmachines-com]], [[summary/smolmachines-com]]*
*Last updated: 2026-08-25*
