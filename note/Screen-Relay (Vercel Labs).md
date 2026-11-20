# Screen-Relay (Vercel Labs)

Vercel Labs' screen-relay gives humans and AI agents one shared desktop to look at and act on: a C++ engine captures a Linux VM, container, Mac display, or Windows desktop, encodes it as H.264, and streams it to a browser over WebCodecs, while the same session exposes a CLI, Node SDK, and HTTP/WebSocket API for an agent to screenshot, click, and type. It is computer-use infrastructure built the honest way — as a small (about 2,800 lines of core C++ plus a thin TypeScript layer) systems project, not an SDK wrapper around a cloud API.

---

## Architecture

Three layers, one process boundary:

- **Engine (`src/engine/`, C++17)** — the whole runtime. `server/main.cpp` and `server/server.cpp` (790 lines) serve everything on one port: viewer HTML, the `/stream` WebSocket, `/control`, `/input`, `/screenshot.png`, clipboard, grants, and `/metrics`. Backends (`backends/linux/x11`, `backends/linux/wayland`, `backends/macos`, `backends/windows`) implement a common capture/input/clipboard surface — Wayland via the `wlr-screencopy`, `wlr-virtual-pointer`, and `virtual-keyboard` protocols; Windows via Desktop Duplication; macOS via CoreGraphics on the logged-in display.
- **TypeScript layer (`src/cli/`, `src/sdk/`, `src/protocol/`)** — the npm package. `protocol/index.ts` defines the wire types and error taxonomy shared by CLI and SDK; `sdk/index.ts` (401 lines) exposes a `Client` (one request per call) and a `Session` (owns input over `/control` until closed or taken). The npm release ships no binaries: `screen-relay install` builds the engine on each machine against the system's x264.
- **Skills (`skills/`)** — versioned agent-facing documentation served by the CLI itself: `screen-relay skills get core` emits the control-loop skill; `--full` inlines every reference file. The repo even dogfoods this in its own `AGENTS.md`, which routes prose work through an internal `technical-writer` skill.

`screen-relay up` composes the session: virtual display → desktop → engine, with a supervisor in `src/cli/host/supervisor.ts`.

## Key techniques

- **Binary input records, not JSON.** `POST /input` takes 1–256 fixed 16-byte records: event type, HID usage code, Unicode scalar or wheel delta, 16-bit fractional X/Y coordinates, button and modifier masks. Big-endian, no JSON — trivially parsed in C++ and cheap to emit from any agent. The server validates in order, reports `{"applied":N}`, and releases keys held by a failed batch.
- **BBR-flavoured flow control for a screen stream (`src/engine/core/flow.hpp`).** The engine samples `TCP_INFO` from the kernel, tracks sliding-window extremes (a hand-rolled monotonic deque in the header) of capture-to-ACK depth over 10 seconds, subtracts the viewer's reported hold time (viewers ACK each frame with how long it waited before drawing, capped at 100 ms so a slow viewer cannot widen the limits), and pauses capture when standing queue exceeds `min RTT + 2 frame intervals`. Target bitrate moves between 1 and 40 Mbps with a 3-second stall timeout. This is congestion control designed for latency-of-perception, not throughput.
- **Token-bucket capture pacing (`capture-bucket.hpp`)** — capture runs at the stream rate with a two-frame burst, so motion stays smooth without overspending the encoder.
- **Container awareness (`cpu-budget.hpp`)** — the engine parses `/proc/self/mountinfo` and cgroup files (including octal-escaped mount fields) to know its CPU quota and degrade frame rate accordingly. A screen encoder in a 2-CPU sandbox is a real constraint and the engine plans for it.
- **Ownership as a protocol, not a lock.** One input owner at a time; `?view=1` watches without ownership, `?take=1` evicts the current owner with close code 4001, `/control` gets 4003 `busy` so a headless client can distinguish refusal from unreachability. Scoped grants (watch/control, TTL ≤ 3600 s, max 32) let you share a viewer link without handing over the session token — and grant tokens are never put in URLs, only in WebSocket subprotocols (`bearer.TOKEN` offered alongside `nsl1`).

## Design decisions

- **No shipped binaries.** Every release builds from source on the target machine via `screen-relay install`, linking the system's GPL x264. Slower first run, cleaner licensing, smaller attack surface, and it works across five Linux distros plus macOS and Windows.
- **The CLI is the agent surface.** Rather than an MCP server or a vision-model API, screen-relay teaches agents a loop with a shipped skill: screenshot → act → screenshot to verify. Its sharpest honest caveat is encoded in the skill: a successful input command means the engine applied the input, not that the application reacted — the agent must close the loop itself.
- **Simplicity of transport over features.** One port, one WebSocket subprotocol, one 20-byte header for every message type, four viewer slots. No audio, no multiplexed control plane, no adaptive codec switching — just H.264 Annex B with keyframes on connect and recovery.
- **Defaults safe, escape hatches explicit.** Loopback, no token, single owner by default; `--public` generates a token that every remote request must send.

## Comparison notes

- Where [[Tmux for AI Agents (crystl)]] gives agents a text-scrapable terminal, screen-relay gives them pixels and pointer events — the visual analogue one layer up. Both share the same philosophy: expose the primitive (capture-pane, screenshot+click) and let the agent loop.
- [[Windows VM Skill for Claude Code]] solves the same "agent needs a GUI desktop" problem with a heavyweight Docker-Windows VM over SSH; screen-relay instead attaches to a desktop that already exists (or a slim headless sway/XFCE session), which is far lighter per session.
- Against [[A Deep Dive on Agent Sandboxes]], screen-relay is a missing piece rather than a competitor: sandboxes give agents files and shells, but GUI verification — "did the dialog actually open?" — needs a screen. Its screenshot-to-verify loop is exactly the externally checkable "done" that [[How agent.ceo Fixes Its Own Bugs]] credits for autonomous bug fixing, applied at the display level.
- The take-control/release ownership model echoes the multiplayer UX of [[Whiteboard (dev.fast)]], but for a live desktop instead of a document canvas; and its JSON-plus-`reason`-plus-`next`-command error contract is a textbook implementation of [[10 Principles for Agent-Native CLIs]].

The honest weakness: the control loop trusts the agent to interpret screenshots — there is no semantic verification, no diffing, no assertions. Screen-relay deliberately stops at the pipe and pushes interpretation to the model. That is the right boundary for infrastructure, but it means quality lives entirely in the skill text.

---
*Sources: [[raw/screen-relay]], [[summary/screen-relay]]*
*Last updated: 2026-11-20*
