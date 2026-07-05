---
url: https://github.com/StartupHakk/OpenMonoAgent.ai
title: OpenMono Agent
author: StartupHakk
date_fetched: 2026-07-05
date_published: 2025
tags: [agents, coding, local-llm, dotnet, tools]
---

# OpenMono Agent — Full Analysis

OpenMono is a local-first coding agent built in C#/.NET 10 that pairs a CLI agent loop with a llama.cpp inference server running in Docker. It is designed to run entirely on local hardware with zero cloud dependency, no subscriptions, and no per-token billing. The project is ~30K lines of C# with ~100 test files, licensed under AGPL-3.0.

## Architecture

### High-level topology

```
openmono CLI (C#)  ←─→  llama.cpp server (Docker)
       │                        │
       ├─ VS Code / Cursor      ├─ Caddy gateway
       │  extension (ACP)       │   ├─ SearXNG (search)
       │                        │   └─ Scrapling (scrape)
       ├─ TUI (Spectre.Console)
       └─ Classic terminal
```

The agent core is the same regardless of interface — CLI TUI, classic terminal, or VS Code extension. The extension communicates over ACP (Agent Client Protocol) via SSE on port 7475.

### Three-box deployment model

1. **Agent box** (laptop): runs the C# CLI or extension
2. **Inference box** (GPU machine): runs llama.cpp + Caddy gateway + optional web services
3. **Relay**: `app.openmonoagent.ai` provides a free frpc relay for dual-box setups

### Startup sequence (Program.cs)

1. Parse CLI flags (`--tui`, `--classic`, `--endpoint`, `--model`, `--acp-only`, etc.)
2. Probe LLM server via `/props` (llama.cpp) or fall back to `/v1/models` (OpenAI-compat) to auto-detect model name and context size
3. Wire dependency injection: config → renderer → permissions → memory → hooks → LSP/MCP managers → playbook registry → tool registry → LLM client
4. Build system prompt: base instructions + project OPENMONO.md + cross-session memory + git branch/status
5. Launch ConversationLoop

### Conversation loop (ConversationLoop.cs)

- **25 iterations per turn** (configurable via maxIterations)
- Context window management: checkpoint at 65% fill, compact at 80% fill
- **Doom loop detection**: 3 identical tool call sequences in a row triggers abort
- **Plan mode**: restricts to read-only tools; mode banner prepended to system message every turn
- Mode can be changed mid-turn by both user and agent

### Tool execution pipeline

Every tool call passes through 12 steps:
1. Parse JSON arguments
2. Schema validation (required fields, types, enums)
3. Sanity check (path outside workspace → reject)
4. Plan mode guard (read-only only in plan mode)
5. Capability check → PermissionEngine
6. Result cache lookup (read-only tools)
7. Pre-tool hook
8. Execute
9. Post-tool hook
10. Artifact store (>10 KB stored, reference returned)
11. Cache write
12. File cache invalidation

**Concurrency model**: Read-only + concurrency-safe tools run in parallel (`Task.WhenAll`). Writeable tools run serially. In-flight execution can start while the LLM is still streaming (for concurrency-safe tools received via streamed tool call deltas).

### 20 built-in tools

FileRead, FileWrite, FileEdit, Glob, Grep, Bash, Agent, Todo, AskUser, MemorySave, WebFetch, WebSearch, ListDirectory, ApplyPatch, EnterPlanMode, CreatePlan, ImplementPlan, Lsp, Roslyn, Playbook

### Sub-agents (AgentTool.cs)

5 specialist sub-agents with isolated sessions, restricted tool sets, and turn budgets:

| Agent | Max turns | Allowed tools | Purpose |
|-------|-----------|---------------|---------|
| general-purpose | 25 | all | generic tasks |
| Explore | 15 | FileRead, Glob, Grep, MCP | read-only discovery |
| Plan | 10 | + TodoWrite (no writes) | architecture planning |
| Coder | 30 | FileRead/Write/Edit, Glob, Grep, Bash | implementation |
| Verify | 20 | FileRead, Glob, Grep, Bash, Roslyn, LSP, MCP | adversarial testing |

Sub-agents have configurable concurrency limits:
- `MaxConcurrentAgents` (default: 2) — semaphore-gated
- `MaxNestingDepth` (default: 3) — prevents infinite recursion
- `MaxQueuedAgents` (default: 4) — queue cap
- `MaxConcurrentPerParent` (default: 2) — per-session fan-out limit

### Permission engine (PermissionEngine.cs)

Dual system:
**Capability system (primary)**: Tools declare fine-grained capabilities (`FileReadCap(path)`, `FileWriteCap(path, op)`, `ProcessExecCap(binary, args)`, `NetworkEgressCap(host, port)`, `VcsMutationCap(repo, op)`, `AgentSpawnCap(type, task)`).

Decision order: session deny-all → config deny patterns → session allow-all → config allow patterns → interactive prompt (allow once / session / deny once / session)

**Legacy system (fallback)**: Tools declare `PermissionLevel`: `AutoAllow`, `Ask`, or `Deny`.

Protected paths (`/etc/`, `/usr/`, `/bin/`, `/System/`, `/Library/`) and blocked binaries (`sudo`, `chmod`, `chown`) are hard-coded. Sub-agents are non-interactive — permissions must be pre-approved in the parent session.

### Context management

Two-tier strategy:

**Checkpointer** (65% threshold, preferred): LLM generates a summary of messages up to N recent turns. The summary is stored as a checkpoint entry with a cutoff message index. Future context windows skip compressed messages and include only the summary + recent turns.

**Compactor** (80% threshold, fallback): Summarize all messages except the last 4 turns. Replace with summary message + recents. Large tool outputs (>2000 chars) are evicted with a placeholder.

Context size is read from the live llama.cpp `/props` endpoint at startup, so thresholds track the real window.

### ACP (Agent Client Protocol)

REST API on port 7475 for the VS Code/Cursor extension. Endpoints:
- `GET /api/v1/discovery` — agent info
- `GET/POST /api/v1/sessions` — list/create sessions
- `GET /api/v1/sessions/{id}/messages` — message history
- `POST /api/v1/sessions/{id}/turn` — send message, resume permission, plan decision, mode toggle, abort

Turns are streamed as SSE events. Session state includes plan mode tracking, and the frontend can toggle mode independently.

### Web services

Self-hosted behind Caddy gateway with auto-detection:
- **SearXNG** for `WebSearch` (DuckDuckGo fallback)
- **Scrapling + Camoufox** for `WebFetch` — real browser with Cloudflare bypass, three-tier engine selection (fast HTTP → auto-escalate → stealthy real browser)

Gateway capabilities are probed once via `GET /services` and cached per URL for the session.

### Playbooks (PlaybookExecutor.cs)

YAML-defined multi-step automation workflows inspired by Ansible. Key features:
- Typed parameters with validation (String, Number, Boolean, Array; enum, min/max)
- Dependency-ordered step execution with topological sort
- Gates (Confirm, Review, Approve) for human checkpoints
- Checkpoint/resume — state persisted to disk after each step
- Template variables: `{{parameters.x}}`, `{{state.x}}`, `{{shell:cmd}}`, `{{file:path}}`
- Tool allowlists per playbook and per step
- Playbook scoped permissions via push/pop on the permission engine stack
- Composable — one playbook can call another

Built-in playbooks: commit (auto-triggered conventional commits), release (end-to-end release pipeline), file-scan (two-step shell integration demo).

### Code intelligence

- **RoslynTool**: Loads `.cs` files into in-memory `AdhocWorkspace` with .NET runtime refs. 5-min compilation cache. 8 actions: overview, find-references, callers, diagnostics, search, type-hierarchy, blast-radius, get-symbol.
- **LSP**: Lazy-started language servers for TypeScript, Python, Go, Rust, C# (via OmniSharp). 5 actions: hover, definition, references, completion, diagnostic.
- **Auto-detected MCP servers**: `code-review-graph` and `graphify` registered automatically if found in PATH with appropriate data files.

### Memory system (MemoryStore.cs)

Cross-session persistent memory using YAML frontmatter markdown files in `~/.openmono/memory/`. Entries have name, description, type, and content. An index file (`MEMORY.md`) is maintained. The MemorySave tool creates new entries; LoadAll reads them on startup.

### Rendering

- **AnsiTuiRenderer**: Full-screen TUI using Spectre.Console + Terminal.Gui, with thinking panel, streaming text, live tok/s indicator
- **TerminalRenderer**: Scrolling REPL for redirected I/O or `--classic` mode

### Dependencies

- Spectre.Console 0.50.0 (rich terminal UI)
- Terminal.Gui 2.0.0-develop (TUI framework)
- Markdig 0.40.0 (Markdown rendering)
- YamlDotNet 16.3.0 (YAML parsing for playbooks, config)
- Microsoft.CodeAnalysis.CSharp.Workspaces 4.12.0 (Roslyn)
- TiktokenSharp 1.2.1 (token counting)
- SixLabors.ImageSharp 3.1.11 (vision/image processing)
- Microsoft.AspNetCore.App (ACP HTTP server)

### Session persistence

JSONL format: line 1 is header, subsequent lines are messages. Path: `~/.openmono/sessions/{date}_{sessionId}.jsonl`. Checkpoints stored alongside as `{sessionId}.checkpoints.json`.

## Key implementation details

1. **Concurrency-safe tool streaming**: ToolDispatcher starts executing concurrency-safe tools immediately when their deltas arrive in the stream, before the stream completes. If a tool crashes, sibling tasks are cancelled via `CancellationTokenSource`.

2. **Permission engine child scoping**: Child engines for sub-agents inherit the parent's allow/deny sets but are marked non-interactive — they can't prompt the user.

3. **Playbook permission scopes**: Playbook runs push a tool name set onto the permission engine stack, pre-approving all listed tools for the duration of the run. Enforced LIFO with validation.

4. **KV cache warming**: On startup, a "ping" message with full system prompt and tools is sent to pre-warm the llama.cpp KV cache. The `/metrics` endpoint is checked first to avoid redundant warmup.

5. **llama-server recovery**: On connection failure, Program.cs attempts to start llama-server via Docker Compose and polls the health endpoint. Even offers to recover from inside Docker by prompting the user to run commands on the host.

6. **Mode banner injection**: Every turn, the current mode (Plan/Build) is prepended to the system message rather than stored in it — ensuring the model never infers its mode from stale history.

7. **Doom loop detection in two places**: Both ConversationLoop and ToolDispatcher implement identical 3-repeat doom-loop checks at different phases.
