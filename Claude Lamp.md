# Claude Lamp

Control a Moonside LED lamp via Bluetooth based on Claude Code's state. Your lamp becomes a physical status indicator -- animated themes while Claude works, solid green when idle, purple when it needs your input.

---

## Key Themes

#hardware #claude-code #developer-experience #physical-computing

The concept is delightful: ambient awareness of what your AI agent is doing, without staring at a terminal. Working? The lamp breathes. Waiting for you? Purple. Done? Warm mango glow. It's the opposite of notification fatigue -- peripheral vision picks up state changes without context switching.

The architecture is simple and clever. A shell hook writes state to a temp file every time Claude Code fires an event (session start, tool use, permission request, etc.). A background daemon polls that file every 200ms and sends BLE commands. The persistent BLE connection avoids the multi-second reconnection delay that would make the lamp useless.

This connects to the broader theme of making AI agent state visible. In [[Agentic Coding]], knowing when an agent is stuck, waiting, or done is crucial for the human-in-the-loop workflow. Most solutions are screen-based. This one uses physical space.

## Critical Analysis

Charming but niche. macOS-only (CoreBluetooth), requires a specific lamp brand (Moonside), and the 200ms polling is a reasonable tradeoff between responsiveness and CPU usage. The 30-minute auto-exit is sensible -- you don't want a daemon running forever.

The real question is whether physical indicators scale. One lamp for one Claude session works. But if you're running multiple agents in parallel (the direction things are heading), you'd need multiple lamps or a different approach entirely. A dashboard is more practical; a lamp is more human.

---
*Sources: [[raw/claude-lamp]]*
*Last updated: 2026-05-14*
