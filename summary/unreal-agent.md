---
url: https://github.com/unreallabsai/unreal-agent
title: "Unreal Agent"
author: Unreal Labs
date_fetched: 2026-09-23
date_published: 2026 (ongoing open-source project)
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

Unreal Agent is an async-first agent harness written in Go (~11K lines of hand-written code plus generated OpenAI types) from Unreal Labs. Its core bet is that tool calls should be *asynchronous operations*, not synchronous blocking steps: the model issues calls, they run in the background concurrently, and each completed result wakes a new LLM turn.

The codebase is organized as a library (`harness/`) plus executables (`cmd/`) and a Harbor benchmark adapter (`benchmarks/`). The coordinator's event loop (`harness/coordinator/loop.go`, ~980 lines) is the heart: it accepts inbox inputs, runs LLM turns, resolves tool translators through a registry, and dispatches committed operations to a swappable actor-runtime operation manager. Tool translators are deliberately pure — they validate a call and emit serializable operations synchronously on the event loop, with no I/O allowed — so the same persisted operation stream can be replayed, forked, or shipped to a remote sandbox (the README's example: a proxy operations manager forwarding operations to a process inside a remote sandbox).

Persistence is append-first: a session is an append-only, versioned, forkable item log stored as JSONL by the localfile session store, and tool-call status is atomically recorded alongside the operations it spawned. Recovery replays the log to rebuild coordinator state; a `TurnCompaction` turn type and context-builder commit/stage split handle long-context management. The built-in tool surface is minimal — Bash, ViewImage, SkillUse — backed by composable "primitives" (process, file, timer, remote SSE) rather than a large tool catalog. Five LLM providers (OpenAI, OpenAI Codex, OpenRouter, Fireworks, Ollama) plug in through a normalized responses-API adapter with retry policies. Testing is notable: fuzz tests on the coordinator log, fault-injection fakes, and recovery-sequence tests treat crash-resume as a first-class behavior.
