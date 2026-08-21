---
url: https://github.com/magnitudedev/magnitude
title: "Magnitude"
author: magnitudedev
date_fetched: 2026-08-21
date_published: unknown
---

# Magnitude — précis

Magnitude is an open-source AI coding agent with local models built in. Apache 2.0 licensed. `npm install -g @magnitudedev/cli` gives you a terminal agent (plus web and Electron desktop clients) that profiles your hardware, recommends and downloads a model from a catalog, and runs it in-process — no Ollama, no model server, no API keys, fully offline after first install. It also supports custom GGUF models from Hugging Face and OpenAI-compatible endpoints.

The pitch ("works out of the box on any hardware") hides a serious engineering core. Under the hood it is a Bun/Turborepo monorepo, Effect-TS native, split across three layers:

- **Clients** — `cli/` (Ink/React terminal), `web/`, `desktop/` (Electron), sharing state through `packages/client-common`.
- **ACN** — the local daemon (`packages/acn`) hosting the agent runtime, sessions, file ops, and display streams. Clients obtain one canonical daemon via JIT "ensurance" coordinated through a single SQLite owner row — no resident coordinator process.
- **ICN** — the Inference Control Node (`inference/`), a Rust workspace built on a pinned fork of llama-cpp-rs/llama.cpp, exposing an OpenAI-compatible HTTP API.

The agent runtime itself (`packages/agent`) is **event-sourced**: a durable event log is the source of truth, and every screen of state — the context window, task graph, compaction state, conversation, display timeline — is a *projection*, a replayable query over committed history. Workers own effects; projections own derived truth. This is the single biggest architectural bet, and the thing that separates Magnitude from every other coding agent's imperative in-memory state.

Key techniques worth the read: context compaction as an event-sourced convergence process (bounded 4,096-token summaries, lossy elision as a deterministic liveness fallback), addressed projection state (small indexes in memory, large bodies in addressable entries, residency = exactly the pinned set), display views that page by reshaping rather than slicing, a single-executor inference scheduler with decode-first batching and exact-prefix KV reuse, speculative decoding (MTP/DFlash/DSpark), and hardware-derived memory accounting that separates *where* an allocation lives from *how much* it needs.
