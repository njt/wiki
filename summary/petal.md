---
url: https://github.com/kiruna-labs/petal
title: "Petal — Low-latency, multi-window screensharing for macOS and Windows"
author: Kiruna Labs
date_fetched: 2026-10-24
date_published: 2026 (ongoing, pre-1.0)
topics:
  - developer-tools
  - guardrails-and-feedback-loops
---

Petal is an open-source (Apache-2.0), pre-1.0 screensharing app for small engineering syncs, built by Kiruna Labs. Its defining behavior: when someone shares a window, it appears on every other participant's desktop as a **real, movable, borderless native window** — not a tile locked in a meeting grid. Each shared window is its own hardware-encoded H.264 stream over WebRTC through a LiveKit SFU.

The stack is a Tauri 2 app (Rust core + Svelte 5 SPA) with deep native surfaces: ScreenCaptureKit capture and a zero-copy CVPixelBuffer → AVSampleBufferDisplayLayer path on macOS; Windows Graphics Capture, a D3D11 compositor, Media Foundation camera and WASAPI audio on Windows. A small Vercel backend mints scoped room tokens so clients never hold the LiveKit API secret; a browser client (web-harness) lets non-app users join.

Beyond sharing, Petal has telepointers (data-channel cursors), fading draw overlays, consent-gated remote control with per-grant capability tokens, and an opt-in Gemini Live AI chat that sits on a shared window — seeing the pixels and accessibility tree, hearing the room only over push-to-talk, and able to act on the window only behind a fail-closed, human-approved control gate (`PETAL_AI_CONTROL=1`, sticky refusals, single-use grants). The repo (~247K lines of Rust) is unusually well-documented: an honest architecture doc, a remote-control trust model, a validation page admitting what hasn't been tested, and a vendored plugin system with signed registries.
