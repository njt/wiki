# Tau (τ) — Educational Coding Agent

Tau is an MIT-licensed terminal coding agent by **Alejandro Ao** designed to be read, not just run. It's a working Python agent (installable via `uv tool install tau-ai`) split into three layered packages — `tau_ai` (provider adapters), `tau_agent` (reusable harness), and `tau_coding` (coding environment + TUI) — each independently studyable. The core design bet: **events are the contract**. The agent harness emits a typed event stream that any frontend can consume, making the same core drive both the built-in Textual TUI and programmatic use via `harness.prompt()`. Inspired by Pi but not a port, Tau trades Pi's "four tools and no MCP" minimalism for pedagogical clarity: "No hidden machinery. Every moving part is on the page."

---

## Key Quotes

> "A small Python coding agent you read like a textbook."

This is the project's thesis. Tau isn't trying to win benchmarks — it's trying to be the reference implementation you wish existed when you first tried to understand how coding agents work. The 438 commits and phase-by-phase `dev-notes/` journal are part of the product.

> "No hidden machinery. Every moving part is on the page."

A direct rebuttal to frameworks that hide the agent loop behind abstractions. Tau's three-package split enforces this: to understand tool execution, you read `tau_agent`; to understand provider normalization, you read `tau_ai`. No magic.

> "Separate the brain, the environment, and the face."

The project's one-sentence architecture lesson. **Brain** = AgentHarness (messages, tools, events, loop). **Environment** = CodingSession (files, shell, sessions, skills). **Face** = TUI (one possible frontend consuming the event stream). This maps directly onto [[Components of a Coding Agent]]'s taxonomy: LLM → reasoning model → agent → harness. Tau's distinction is making the boundaries so crisp you can study each in isolation.

> "Events are the contract."

Providers, renderers, the TUI, and custom frontends all interface through a typed event stream. No tight coupling between model calls and screen rendering. This is the architectural insight that makes Tau both teachable and embeddable: the same `AgentHarness` that drives the TUI can be imported and iterated over programmatically.

> "Tools are ordinary typed functions."

A schema plus an async executor returning structured results. No LangChain tool abstractions, no MCP protocol negotiation. This is the same philosophy as [[What I learned building an opinionated and minimal coding agent]]: tools should be boring infrastructure, not a framework.

---

## Key Themes

#tool #coding-agents #educational #architecture #agent-harness #python #minimal-agents #sessions

- **Educational architecture** — Tau is to coding agents what nand2tetris is to CPUs: a working implementation small enough to understand in a weekend. The three-package split and dev-notes journal make the learning path explicit.

- **Event-driven agent loop** — The harness emits typed events rather than rendering UI. This is the architectural insight that separates a coding agent's brain from its face, and it's what makes the same core reusable across TUI, print mode, and programmatic embedding.

- **Durable sessions** — Append-only JSONL under `~/.tau/sessions/` with resume and branching. Every interaction is logged; compaction creates a new active context without rewriting history. This is [[Coding Agents Continuity Not Memory]] in practice: continuity through immutable event logs, not bigger context windows.

- **Provider neutrality** — `tau_ai` normalizes OpenAI, Anthropic, Hugging Face, OpenRouter, and local endpoints into a single event stream. The same pattern as [[Mirage (VFS)]] but for model providers: one interface, many backends.

---

## Critical Analysis

**The textbook approach is the differentiator, not the agent.** Tau won't beat Claude Code or Codex on benchmarks. It's not trying to. What it offers that no other coding agent does is a deliberate learning path from provider adapters through agent loop through TUI — with commit-level granularity in the dev notes. This is the project that answers "how do I build my own coding agent?" when "read the Claude Code source" isn't practical.

**Events-as-contract is the reusable insight.** Most agent frameworks couple model calls to UI rendering. Tau's typed event stream — consumed by TUI, print mode, or custom code — is the cleanest separation of brain and face I've seen in a small codebase. If you're building an agent and want to swap frontends later, steal this pattern.

**The Pi comparison is instructive.** [[Pi Coding Agent]] and Tau share a lineage (Tau was inspired by Pi) but diverged in philosophy. Pi went from "four tools, no MCP" to a TypeScript extension ecosystem; Tau went from "inspired by Pi" to a Python textbook. One became a platform; the other became a curriculum. Both are valid, but Tau's path is more interesting for learners.

**Sessions as JSONL is underrated.** The append-only JSONL format with resume and branching is both simpler and more powerful than most session management schemes. You can fork a session at any point without corrupting history. Compaction keeps context manageable. This is the same insight behind [[claude-replay]] and session analysis tools — immutable event logs are the right primitive.

**What's missing.** The docs don't cover eval methodology, security sandboxing, or production deployment — but that's consistent with the educational framing. Tau teaches you how the machinery works; production hardening is your job. The question is whether learners will know what they don't know after finishing the textbook.

---

*Source: [[summary/tau]], https://twotimespi.dev/, https://github.com/alejandro-ao/tau*
*Date fetched: 2026-07-03*
