---
url: https://github.com/vercel-labs/screen-relay
title: "screen-relay"
author: Vercel Labs
date_fetched: 2026-11-20
date_published: 2026
topics:
  - developer-tools
  - agent-coding-workflow
---

Vercel Labs' screen-relay streams a desktop — a Linux VM, sandbox, or container, a Mac's logged-in display, or a signed-in Windows PC's desktop — to a browser as H.264, and gives AI agents one-command control of the same screen: screenshot, click, type, key. It is a Vercel Labs "experiment" (v0.1.2, Apache-2.0), built as a Node 22+ CLI/SDK plus a hand-written C++ engine.

The architecture is deliberately thin: `screen-relay up` starts a virtual display and desktop (X11+XFCE or headless Wayland+sway on Linux), then a single C++ engine process captures the screen, encodes it with x264 at `ultrafast`/`zerolatency` presets, and serves viewer, stream, and input API on one loopback port (default 127.0.0.1:8080; remote via SSH tunnel or token). Input uses a binary 16-byte-record protocol (`POST /input`, up to 256 records per batch, big-endian, HID usage codes, coordinates as 0–65535 fractions) rather than JSON — cheap to parse in C++ and to emit from any agent. WebSocket paths (`/stream`, `/control`) carry a 20-byte `NSL1` binary header with per-connection generation/sequence numbers.

The engineering depth is in congestion control: a custom flow-control module (`src/engine/core/flow.hpp`) reads Linux `TCP_INFO` statistics, tracks the minimum capture-to-ACK depth over a sliding 10-second window, distinguishes viewer-side hold time from network queueing, and pauses capture when the standing queue exceeds base RTT plus two frame intervals — a BBR-adjacent design tuned for screen streaming rather than throughput. A token bucket (`capture-bucket.hpp`) paces capture at the stream rate with a two-frame burst, and a CPU-budget module reads cgroup quotas so the engine degrades politely in containers.

For agents, the interesting design is that the CLI is the agent interface: `screen-relay skills get core` ships a versioned skill teaching the screenshot–act–screenshot loop, coordinates-as-fractions, and the crucial caveat that a successful input command means the engine applied the input, not that the app reacted. Every command prints JSON with `--json`, and errors carry a `reason` plus the next command to run. Sessions default to loopback with no token, one input owner at a time (viewers can take/release control), and scoped, expiring grants for watch-only sharing.
