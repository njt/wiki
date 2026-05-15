# acpx

Headless CLI client for the Agent Client Protocol (ACP) that replaces PTY scraping with structured, protocol-based communication between AI agents and orchestrators. One command surface wrapping 16+ coding agents -- Claude Code, Codex, Gemini, Cursor, Copilot, and others -- with persistent sessions, prompt queueing, and an experimental TypeScript flow runtime for multi-step workflows. This is the plumbing layer that makes agent-to-agent delegation possible without the brittleness of terminal scraping.

---

## Key Quotes

> "Your agents love acpx! They hate having to scrape characters from a PTY session."

The entire value proposition in two sentences. PTY scraping is the duct tape holding together every orchestration system that spawns coding agents as subprocesses. acpx replaces that with typed messages: thinking events, tool calls, diffs. This is the difference between parsing ANSI escape codes and reading structured JSON.

> "Persistent sessions: multi-turn conversations that survive across invocations, scoped per repo."

Session persistence scoped to git roots is the right primitive. It means an orchestrator can fire off a prompt, go do other work, come back later, and continue the conversation. Combined with named sessions (`-s backend`, `-s frontend`), you get parallel workstreams within a single repo without session collision. This is what [[Dorothy]] and [[Agent of Empires]] are trying to solve at a higher layer.

> "Prompt queueing: submit prompts while one is already running, they execute in order."

Queue semantics matter more than they look. Without this, every orchestrator has to implement its own "wait for the agent to finish" polling loop. With it, you can fire-and-forget (`--no-wait`) and trust the queue. The queue owner TTL (default 300s, configurable) is a pragmatic touch -- keep the process alive for quick follow-ups without leaking indefinitely.

> "`acp` steps keep model-shaped work in ACP ... `action` steps handle deterministic mechanics like shell commands"

The flow runtime draws a clean line between what the model does and what the machine does. ACP steps for reasoning, action steps for shell commands, decision steps for branching. This is the same separation that [[Scaling Long-Running Agents]] found essential: planners plan, workers work, and deterministic code handles the glue.

## Key Themes

#tool #agent-orchestration #protocol #coding-agents #cli

**Protocol over scraping.** The Agent Client Protocol is to coding agents what LSP was to editors: a standard wire format that decouples the client from the server. acpx is the first serious CLI client for it, wrapping 16 agents behind one interface. The protocol gives you typed events (thinking, tool_call, diff) instead of ANSI character streams.

**Session as a first-class resource.** Sessions are scoped to git roots, named for parallel work, queue-aware for concurrent access, and crash-resilient with auto-reconnect. This is the session management layer that [[klaw.sh]] provides through Kubernetes abstractions and [[Agent of Empires]] provides through Rust -- acpx does it through the protocol itself.

**Flow runtime.** The experimental `flow run` command executes TypeScript modules that chain ACP prompts with shell actions, branching decisions, and checkpoints. Per-step working directories keep agent work isolated in disposable worktrees. This connects to [[workgraph]]'s persistent task graphs and [[Cord]]'s dynamic task trees, but scoped to a single orchestrator rather than a distributed system.

**Agent-agnostic.** Built-in adapters for 16 agents plus an `--agent` escape hatch for custom ACP servers. The adapter table reads like a census of the coding agent ecosystem: Pi, OpenClaw, Codex, Claude, Gemini, Cursor, Copilot, Factory Droid, iFlow, Kilocode, Kimi, Kiro, OpenCode, Qoder, Qwen, Trae.

## Critical Analysis

**What's strong.** The core abstraction is right: sessions, queues, and typed events are exactly the primitives an orchestrator needs to drive coding agents programmatically. The NDJSON output format means you can pipe acpx into jq and build automation without writing an SDK. The crash-reconnect behavior (detect dead pid, respawn, attempt session/load, fall back to session/new) is the kind of operational resilience that most agent tools skip entirely. Permission controls (`--approve-all`, `--approve-reads`, `--deny-all`) show awareness that unattended agents need mechanical guardrails, not just prompt instructions -- the same insight behind [[claude-ctrl]] and [[Feedback Loop is All You Need]].

**What's missing.** There's no story for distributed orchestration. acpx is a single-machine CLI, not a network service. If you want multiple machines driving agents, you need something above it. The flow runtime is explicitly experimental and the spec coverage is incomplete (per their own roadmap). There's also no built-in eval or quality gate -- you can build one with action steps and decision branching, but [[Gambit]] and [[Woodshed]] offer purpose-built solutions for that.

**The real significance.** acpx is a bet that ACP becomes the LSP of coding agents. If that bet pays off, acpx is the curl of the agent world -- the universal CLI client that every orchestration system shells out to. If ACP doesn't achieve protocol-level adoption, acpx is just a nice wrapper around 16 different agent CLIs. The 2.7k stars and the breadth of the adapter table suggest momentum, but alpha status means the interfaces are still shifting. The comparison to [[Components of a Coding Agent]] is instructive: Raschka identifies six components of a coding agent, and acpx doesn't try to be any of them -- it's the wire between them.

**Connection to the wiki's orchestration theme.** [[Agent Orchestration]] identifies the planner/worker/judge pattern as the convergent architecture. acpx doesn't implement that pattern, but it provides the transport layer for it. An orchestrator using acpx can spawn named sessions for workers, queue prompts, read structured results, and make decisions -- all without parsing terminal output. This is the missing "how do I actually talk to the workers" layer that [[maestro]], [[Cord]], and [[Dorothy]] each solve differently. acpx solves it once, at the protocol level.

---
*Sources: [[raw/acpx]]*
*Last updated: 2026-05-14*
