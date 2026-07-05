# OpenMono Agent

A local-first, open-source coding agent that runs entirely on your hardware — a .NET 10 CLI paired with a bundled llama.cpp inference server in Docker. Zero cloud dependency, zero subscriptions, zero per-token billing. Built by StartupHakk, it combines 20 built-in tools, 5 specialist sub-agents, a dual-tier context management system (checkpoint + compact), a YAML-based playbook engine for repeatable workflows, and a capability-based permission model. Ships with self-hosted web search (SearXNG) and scraping (Scrapling + Camoufox), VS Code/Cursor extension via its own Agent Client Protocol (ACP), and distributed inference support for separating the agent laptop from the GPU machine.

---

## Architecture

OpenMono is structured as a .NET 10 CLI monolith with a clear internal separation of concerns. The agent core (`ConversationLoop`) is shared across three frontends: a Spectre.Console TUI, a classic scrolling terminal, and a VS Code/Cursor extension connected via ACP (SSE over HTTP on port 7475).

```
openmono CLI (C#) ──── llama.cpp server (Docker)
     │                         │
     ├─ TUI / Classic          ├─ Caddy gateway
     ├─ ACP server (:7475)     │   ├─ SearXNG (search)
     │   └─ VS Code extension  │   └─ Scrapling (scrape)
     └─ PermissionEngine       └─ frpc tunnel (relay)
```

### Startup sequence (`src/OpenMono.Cli/Program.cs`)

1. Parse CLI flags, probe the LLM server via `/props` (llama.cpp) or `/v1/models` (OpenAI-compat) to auto-detect model name and context size
2. Wire DI: config → renderer → permissions → memory → hooks → LSP/MCP managers → playbook registry → 20-tool registry → LLM client
3. Build system prompt: base instructions + `OPENMONO.md` + cross-session memory + git branch/status
4. Launch `ConversationLoop`, optionally with KV cache warmup (a "ping" message that pre-caches the full system prompt + tools)
5. On connection failure, offers to auto-start llama-server via Docker Compose and polls `/health` with detailed status feedback

### Conversation loop (`src/OpenMono.Cli/Session/ConversationLoop.cs`)

- Up to 25 iterations per turn (configurable)
- **Doom loop detection**: 3 identical tool call sequences → abort
- **Plan/Build mode**: Plan mode restricts to read-only tools. A mode banner is prepended to the system message each turn (ephemeral, not persisted) so the model never infers its mode from stale history
- The mode can be changed mid-turn by the agent (EnterPlanMode/ExitPlanMode tools) or by the user — both the TUI and extension UI stay in sync

### Tool execution pipeline

Every tool call goes through 12 steps before touching anything: parse JSON → schema validate → sanity check → plan mode guard → capability check → cache lookup → pre-hook → execute → post-hook → artifact store (results >10KB stored to file, reference returned) → cache write → cache invalidation.

**Concurrency model**: Read-only + concurrency-safe tools execute in parallel via `Task.WhenAll`, gated by `SemaphoreSlim` (default limit = `Environment.ProcessorCount`). Writeable tools run serially. For streamed tool calls, concurrency-safe tools begin execution immediately when their delta arrives — before the LLM stream completes. If one crashes, sibling tasks are cancelled via `CancellationTokenSource`.

### 20 built-in tools (`src/OpenMono.Cli/Tools/`)

FileRead, FileWrite, FileEdit, Glob, Grep, Bash, Agent, Todo, AskUser, MemorySave, WebFetch, WebSearch, ListDirectory, ApplyPatch, EnterPlanMode, CreatePlan, ImplementPlan, Lsp, Roslyn, Playbook — plus MCP tools dynamically registered at startup.

### Sub-agents (`src/OpenMono.Cli/Tools/AgentTool.cs`)

5 specialist sub-agents with isolated sessions, restricted tool sets, and turn budgets. Each spawns a fresh `ConversationLoop` with a dedicated system prompt. Sub-agents are non-interactive — permissions must be pre-approved in the parent session. A global semaphore gates concurrency:

| Agent | Max turns | Tools | Purpose |
|-------|-----------|-------|---------|
| Explore | 15 | read-only | discovery |
| Plan | 10 | + TodoWrite | architecture |
| Coder | 30 | file ops + bash | implementation |
| Verify | 20 | + Roslyn, LSP, MCP | adversarial testing |
| general-purpose | 25 | all | generic |

Configurable limits: `MaxConcurrentAgents` (default 2), `MaxNestingDepth` (default 3), `MaxQueuedAgents` (default 4), `MaxConcurrentPerParent` (default 2).

### Permission engine (`src/OpenMono.Cli/Permissions/PermissionEngine.cs`)

Dual system. **Capability system (primary)**: Tools declare fine-grained capabilities — `FileReadCap(path)`, `FileWriteCap(path, op)`, `ProcessExecCap(binary, args)`, `NetworkEgressCap(host, port)`, `VcsMutationCap(repo, op)`, `AgentSpawnCap(type, task)`. Decision order: session deny-all → config deny patterns → session allow-all → config allow patterns → interactive prompt. **Legacy system (fallback)**: `PermissionLevel` enum (`AutoAllow`, `Ask`, `Deny`). Hard-coded protections: blocked binaries (`sudo`, `chmod`, `chown`), protected paths (`/etc/`, `/usr/`, `/System/`). Sub-agents inherit parent allow/deny sets but are marked non-interactive. Playbook runs push a pre-approved tool set onto the permission stack (LIFO-enforced).

### Context management (`src/OpenMono.Cli/Session/Compactor.cs`, `Checkpointer.cs`)

Two-tier strategy with real token tracking from the llama.cpp `/props` endpoint:

| Threshold | Action | Mechanism |
|-----------|--------|-----------|
| 65% | **Checkpoint** | LLM-generated summary of older messages; future windows include summary + recent turns only |
| 80% | **Compact** (fallback) | Summarize all but last 4 turns; evict tool outputs >2000 chars with placeholders |

### Playbooks (`src/OpenMono.Cli/Playbooks/`)

YAML-defined multi-step automation workflows, similar in spirit to Ansible playbooks but executed by the LLM:
- **Typed parameters** with validation (String/Number/Boolean/Array; enum, min/max constraints)
- **Topological step ordering** via `requires` dependencies
- **Gates**: Confirm, Review, Approve — pause for human input at critical steps
- **Checkpoint/resume**: State persisted to disk (`~/.openmono/playbook-state/`) after each step
- **Template engine**: `{{parameters.x}}`, `{{state.x}}`, `{{shell:cmd}}`, `{{file:path}}`
- **Tool allowlists** per playbook and per step with wildcard support
- **Permission scoping**: Push tool list onto permission engine stack for the duration of the run
- Three built-in playbooks: commit (auto-triggered conventional commits), release (end-to-end pipeline with changelog, version bump, test gate), file-scan (shell integration demo)

### Web services

Self-hosted search and scraping behind a Caddy gateway:
- **WebSearch**: SearXNG primary, DuckDuckGo fallback
- **WebFetch**: Scrapling + Camoufox with three-tier engine selection — fast HTTP → auto-escalate on 403/429/503 → stealthy real browser with Cloudflare bypass
- Gateway capabilities probed once via `GET /services` and cached per URL for the session

### Code intelligence

- **RoslynTool**: In-memory `AdhocWorkspace` with .NET runtime refs, 5-min compilation cache, 8 analysis actions (overview, find-references, callers, diagnostics, search, type-hierarchy, blast-radius, get-symbol)
- **LSP**: Lazy-started servers for TypeScript, Python, Go, Rust, C#; 5 actions (hover, definition, references, completion, diagnostic)
- **MCP**: Servers spawned as subprocesses, tools registered as `mcp__{server}__{tool}`; `code-review-graph` and `graphify` auto-detected

### Memory (`src/OpenMono.Cli/Memory/MemoryStore.cs`)

Cross-session persistent memory using YAML frontmatter markdown files in `~/.openmono/memory/`. Each entry has name, description, type (`user`/`project`/`feedback`/`reference`), and content. An index (`MEMORY.md`) is maintained. Similar in concept to Claude Code's memory system.

### ACP (`src/OpenMono.Cli/Acp/`)

REST API on port 7475 for the VS Code/Cursor extension. Sessions support turn-level locking (409 Conflict if busy), SSE streaming, mode toggling, abort, permission resume, and plan decision routing. The lock file mechanism prevents multiple agents from serving the same workspace simultaneously.

### Session persistence

JSONL format with a header record on line 1: `~/.openmono/sessions/{date}_{sessionId}.jsonl`. Checkpoints stored alongside as `{sessionId}.checkpoints.json`.

---

## Key techniques

**Stream-parallel tool execution**: Unlike most agent loops that wait for the full LLM response before dispatching tools, OpenMono starts executing concurrency-safe tools immediately when their deltas arrive in the stream. This is implemented in `ConversationLoop.RunTurnInternalAsync()` by launching `Task.Run` for each `ToolCallDelta` that maps to a concurrency-safe tool, tracking them in a `Dictionary<string, Task<ToolResult>>`, and awaiting them after the stream completes. If any tool crashes, sibling tasks are cancelled.

**LLM-driven checkpoint summarization rather than rule-based truncation**: The `Checkpointer` uses the LLM itself (temperature 0.1, max 4096 tokens) to summarize compressed messages rather than applying a fixed truncation rule. This preserves semantic meaning — decisions, errors, and context that a simple "keep last N messages" approach would lose.

**Capability-based permissions with pattern matching**: Instead of a simple allow/deny list against tool names, `PermissionEngine` evaluates individual `Capability` objects. For example, `FileWriteCap` carries the specific path, and both global deny rules (protected paths) and user-configured patterns are checked against it. This gives finer-grained control than "allow Bash" — you can allow `git *` while denying `rm -rf *`.

**Playbook permission stack**: When a playbook executes, its allowed tools are pushed onto the permission engine's scope stack. All listed tools (and `*`) are pre-approved for the duration. The stack is LIFO-enforced — `PopPlaybookScope` validates the run ID matches the top of the stack and throws `InvalidOperationException` if it doesn't, preventing scope corruption.

**Gateway capability auto-detection with graceful degradation**: `GatewayCapabilities` probes `GET /services` on the Caddy gateway once per session, caching the result in a `ConcurrentDictionary` keyed by URL. If the probe fails or a service is absent, the tool silently falls back to DuckDuckGo / direct fetch — the user never sees an error, the tool just uses the degraded path.

**KV cache warming via metrics endpoint**: Before the first real turn, `Program.cs` checks `/metrics` for `llamacpp:prompt_tokens_total > 0`. If the cache is cold, it sends a non-streaming `max_tokens=1` "ping" request with the full system prompt + tool definitions to pre-populate the KV cache. This is a practical optimization that a surprising number of local-LLM harnesses omit.

---

## Design decisions

**Optimized for local models first, cloud providers second**: The default provider is llama.cpp running locally. OpenAI, Anthropic, and Ollama are supported but marked as WIP. The entire configuration system auto-detects model capabilities from `/props` rather than requiring manual configuration. Sampling defaults (`temperature: 0.7`, `top_p: 0.8`, `top_k: 20`, `presence_penalty: 1.5`) are tuned for Qwen 3.6 models specifically. This is the reverse of most coding agents (Claude Code, Codex, Cursor) which are cloud-first with local as an afterthought.

**Monolith over microservices, internal abstraction over framework**: Despite having many subsystems (20 tools, 5 sub-agents, playbooks, ACP, MCP, LSP, Roslyn, hooks, permissions), everything lives in a single .NET CLI project. There's no plugin SDK, no gRPC between components, no message queue. The trade-off is clear: simpler deployment (one binary) and easier debugging (one stack trace) at the cost of less horizontal scalability and language heterogeneity.

**Dual context management is pragmatic redundancy**: Why have both checkpointing (65%) and compacting (80%) when they do similar things? Because checkpointing uses the LLM (better quality, slower) while compacting is the safety net. If the LLM is unavailable or the checkpoint call fails, compaction still works. If checkpointing succeeds, you get a semantically rich summary. This is defense-in-depth for context management — something most agents don't do.

**Playbooks as Ansible for LLMs**: Rather than inventing a new workflow language, the playbook system directly borrows Ansible's conceptual model: YAML definitions, typed parameters, idempotent steps, dependency ordering, and state persistence. This is a smart choice because it makes playbooks immediately understandable to DevOps engineers while being expressive enough for LLM-driven automation. The template engine (`{{shell:cmd}}`, `{{file:path}}`) bridges the gap between declarative YAML and live system state.

**Capability over tool-name permissions**: Most coding agents permission at the tool level ("allow Bash"). OpenMono's capability system operates at the *intent* level — `FileWriteCap(path)` carries the specific path, `ProcessExecCap(binary, args)` carries the specific command. This enables rules like "allow Bash for git commands" without granting full shell access. The trade-off is implementation complexity: every tool must declare its capabilities, and the permission engine must evaluate them per-call.

**C# choice is unconventional but coherent**: Most AI/agent tooling is Python or TypeScript. OpenMono uses C# with .NET 10 AOT compilation. This gives it strong typing, excellent IDE support (Roslyn integration is a natural fit), mature async/await patterns, and single-binary deployment. The trade-off is a smaller contributor pool and fewer reusable AI/LLM libraries compared to the Python ecosystem.

---

## Comparison notes

**vs. Claude Code**: Both use a TUI coding agent loop with sub-agents, hooks, and permission management. Claude Code is cloud-only (Anthropic API), JavaScript-based, and deeply integrated with the Anthropic ecosystem. OpenMono is local-first, C#-based, and model-agnostic. OpenMono's playbook system has no equivalent in Claude Code's skills/hooks — playbooks are more structured, with typed parameters and checkpoint/resume. Claude Code's CLAUDE.md and memory system are more mature.

**vs. DeerFlow**: Both are comprehensive agent frameworks with sub-agent delegation and sandboxed execution. DeerFlow is Python/LangGraph with 175K lines and targets enterprise deployment with IM channel bridges. OpenMono is C#/.NET with 30K lines and targets individual developers running locally. DeerFlow has a richer middleware pipeline (27 middleware steps), while OpenMono has a more focused tool execution pipeline (12 steps) with tighter security (capability-based permissions).

**vs. MiMo Code**: Both optimize for long-horizon coding tasks with checkpointing and cross-session learning. MiMo Code uses independent writer sub-agents for memory extraction and Dynamic Workflow for orchestration; OpenMono uses LLM-generated checkpoint summaries and YAML-based playbooks. MiMo Code targets cloud deployment with Claude-compatible APIs; OpenMono targets local deployment with llama.cpp.

**vs. Components of a Coding Agent**: The taxonomy framework from that page (LLM/reasoning-model/agent/harness) applies cleanly to OpenMono. The harness is the dominant component — 30K lines of C# tool pipeline, context management, and permission infrastructure surrounding the LLM. This aligns with the argument that "the harness matters more than the model."

---

#tool #project #agents #coding #dotnet #local-llm

---
*Sources: [[raw/openmono-agent]]*
*Last updated: 2026-07-05*
