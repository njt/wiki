# Resident (ESP32 Sandbox)

A Lua-based sandbox runtime for ESP32 microcontrollers, purpose-built for AI agents operating in the physical world. Hot-reloadable apps over WebSockets, hardware drivers exposed through a controlled Lua API, and Claude Code skills for the full app creation lifecycle. Alpha-stage, MIT licensed, from INANIMATE.

---

## Key Quotes

> "Devices need sandboxes for agents. AI is coming into the real world."

The project's thesis in one sentence. Most sandboxing conversations are about containing agents. Resident inverts this: the sandbox is a *host* that gives agents a safe place to run code on physical hardware. This is the same architectural inversion that [[Stockyard]] makes (sandbox as launchpad, not cage), applied to embedded devices instead of cloud VMs.

> "AI agents are how we code now, and also the emerging interface for end users."

This is the most compressed statement of the two-audience bet: agents as both the *builders* of device behavior and the *interface* that end users interact with. It echoes [[Intent Is the Interface]] — if the interface is derived from context, then an agent-mediated device doesn't need a designed UI at all.

> "Easy for agents to hermit crab behavior into the world."

The most original framing on the page. A hermit crab doesn't build a shell — it finds one and inhabits it. The implication is that agents shouldn't generate hardware interfaces from scratch; they should *occupy* existing hardware through a sandbox layer. This connects directly to [[Agent Identity]] — the hermit crab has a stake in its shell because its body is shaped to it. Identity isn't memory; it's the fit between agent and substrate.

---

## Architecture

Five cleanly separated layers:

1. **Device** — ESP32 with connectivity and custom hardware
2. **Driver** — C++ extensions exposing specific hardware peripherals to Lua
3. **Sandbox** — Lua runtime with isolation and hot-reload
4. **App** — Lua apps pushed over the network
5. **Events** — JSON over WebSockets or MQTT from hardware and network

The layer separation is the architecture's strength. The Driver layer is a capability whitelist — the ESP32 has GPIO pins, I2C, SPI, but the sandbox only sees what the C++ drivers expose. This is the same pattern as [[A Deep Dive on Agent Sandboxes]] (Seatbelt, Landlock, seccomp as capability filters), implemented in firmware rather than OS syscalls.

The Event layer is worth noting: JSON over WebSockets or MQTT. These are protocols agents already speak. No custom binary protocol, no RPC framework to learn. This is [[Smart Models Dumb Pipes]] applied to hardware — the device emits JSON events, the agent interprets them, no translation layer needed.

Built on [Courier](https://github.com/inanimate-tech/courier) for "batteries-included connectivity including Wi-Fi config and JSON messaging."

---

## Key Themes

#tool #sandbox #embedded #esp32 #lua #agent-infrastructure

- **Sandbox as host, not cage** — The semantic inversion from "sandboxes contain dangerous agents" to "sandboxes give agents safe hardware access" is the page's intellectual contribution. Related: [[Security and Sandboxing]], [[Stockyard]], [[yolo-cage]].
- **Hermit crab architecture** — Agents inhabit existing hardware through a controlled interface layer rather than generating new interfaces. This is a concrete implementation of the identity-as-participation argument in [[Agent Identity]].
- **Browser simulator as development loop** — The M5StickS3 simulator means you can develop and test agent-driven hardware apps without physical devices. This is the same principle as [[Agentic Manual Testing]] (verify via execution), ported to embedded hardware.
- **Agent-first tooling** — Claude Code skills, plugin marketplace, agent-facing docs. The project treats agents as first-class developers, not an afterthought. Consistent with the [[Components of a Coding Agent]] finding that the harness matters more than the model.

---

## Critical Analysis

**What's genuinely novel:** The hermit crab framing. Most IoT/embedded agent projects think in terms of "agent controls device." Resident thinks in terms of "agent inhabits device." The difference is that control is episodic (send command, receive response) while inhabiting is continuous (the agent's code lives on the device and reacts to events without round-tripping to the cloud). If agents are going to operate in the physical world, the control model doesn't scale — you can't round-trip every sensor reading. Inhabitation does.

**The Lua choice is smart and underappreciated.** Lua was designed as an embedded scripting language. It's tiny, fast, and has decades of battle-testing in game engines. For an ESP32, where every byte of RAM matters, Lua is a better fit than JavaScript, Python, or WASM. The hot-reload capability is trivial in Lua (replace a function reference) but would require contortions in most alternatives.

**The browser simulator is the sleeper feature.** Hardware development has a vicious feedback loop: you need the device to test, but you can't iterate quickly because flashing is slow. A browser-based simulator breaks this loop. The fact that it's an M5StickS3 specifically (a common, cheap dev board with a screen) suggests they're targeting makers and prototypers rather than industrial deployments — at least for now.

**What's missing:** No mention of security implications for hot-reloading code onto physical devices. If an agent can push Lua to a running ESP32, what prevents a compromised agent from pushing malicious code? The sandbox isolates the Lua runtime from the hardware, but the attack surface between "agent pushes code" and "code runs on device" is unexamined. This is the same class of problem that [[yolo-cage]] and [[Crabbox]] address for cloud agents, but the physical safety implications are higher — a compromised smart outlet is worse than a compromised CI runner.

**The alpha caveat is doing real work.** "Alpha" signals ambition more than readiness. The design philosophy is well-articulated, but there's no evidence of production deployment, no security audit, no community. The Claude Code skills and plugin marketplace are described in future tense. This is a vision document with a working prototype, not a shipped product.

**The ESP32 bet is simultaneously too narrow and exactly right.** Too narrow because it excludes the billions of non-ESP32 embedded devices. Exactly right because ESP32 is the Raspberry Pi of microcontrollers — cheap, ubiquitous, well-documented, and already in the hands of the people most likely to experiment with agent-driven hardware. The "makers, hardware startups, and mass production" framing acknowledges the full spectrum from hobbyist to industrial.

---

## Connections

- [[MimiClaw]] — the other ESP32 AI project in the wiki. MimiClaw runs an AI *on* the ESP32 (inference via cloud, orchestration local). Resident runs AI-written code *on* the ESP32. Complementary architectures: MimiClaw is an agent in a chip, Resident is a chip that agents inhabit.
- [[Security and Sandboxing]] — the broader containment context. Resident's Driver-as-capability-whitelist is the embedded equivalent of seccomp profiles.
- [[Stockyard]] — Firecracker micro-VM design that shares the "sandbox as host" inversion.
- [[Agent Identity]] — the hermit crab metaphor maps directly to identity-as-participation.
- [[Intent Is the Interface]] — if the interface is derived, Resident's agent-facing layer is what derivation looks like for hardware.
- [[Components of a Coding Agent]] — harness over model. Resident is a harness for agent-driven hardware.
- [[Compound Engineering]] — "a system that produces code is more valuable than any individual piece of code." Resident is a system for producing device behavior.

---
*Sources: [[raw/resident-esp32-sandbox]]*
*Last updated: 2026-05-22*
