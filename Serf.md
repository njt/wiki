# Serf

A non-interactive coding agent from Prime Radiant (Jesse's company). Give it a task, it does the work -- file operations, command execution, code search -- in a structured loop until the objective is met. Go-based, multi-provider (OpenAI, Anthropic, Google, OpenRouter, Ollama), with persistent sessions and a web hub for concurrent management.

---

## Key Themes

#coding-agent #non-interactive #go #multi-provider #session-management

Serf's differentiator is the non-interactive model: no human in the loop during execution. This puts it closer to CI/CD automation than to conversational coding assistants. The session management (auto-save, resume, carry-forward context) and the web hub for concurrent sessions suggest it's designed for running many agents on many tasks simultaneously.

The lineage from Kilroy (Dan Shapiro, StrongDM Attractor project) gives it a pedigree in production agent deployment. The hub architecture with loopback-only daemons and rendezvous files is a thoughtful security choice -- agents communicate through the filesystem, not network sockets.

Sits alongside [[Ralph]] (also iterates until done, but driven by PRD stories) and [[Dorothy]] (visual orchestration of multiple agents). Serf is the simplest of the three: no orchestrator, no PRD, just "here's a task, go."

## Critical Analysis

Strong: the non-interactive design forces clean task specification, which improves reproducibility. Session persistence with cross-provider model switching is useful for long-running tasks where you might want to switch from a fast model to a capable one mid-stream. The Go implementation keeps the binary small and fast.

Weak: "non-interactive" means you can't course-correct. If the agent goes down a wrong path, you only find out when it's done (or stalled). This is fine for well-specified tasks but risky for exploratory work. Compare with [[What I learned building an opinionated and minimal coding agent]] which argues for full observability during execution.

---
*Sources: [[raw/serf]]*
*Last updated: 2026-05-14*
