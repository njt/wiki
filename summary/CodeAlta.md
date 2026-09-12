---
url: https://github.com/CodeAlta/CodeAlta
title: CodeAlta
author: Alexandre Mutel (xoofx)
date_fetched: 2026-07-05
date_published: 2025
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

# CodeAlta — Terminal AI Coding Agent Workspace

CodeAlta is a terminal-based AI coding agent workspace (TUI) built in C#/.NET by Alexandre Mutel. It provides a single `alta` CLI command that integrates model-provider setup, project navigation, prompt attachments, durable sessions, delegated agent work, and trusted local plugins behind a keyboard-first terminal UI. Licensed under BSD-2-Clause.

## Architecture Deep Dive

CodeAlta is a .NET 10 monorepo of ~25 projects organized in a strict layered architecture.

### Layer Structure

```
CodeAlta (TUI frontend, XenoAtom.Terminal)
  ↓ depends on
CodeAlta.Orchestration (session lifecycle, runtime events, system prompts)
  ↓ depends on
CodeAlta.Agent (provider contracts, session runtime, compaction, tools)
CodeAlta.Catalog (project/session/skill catalogs, config store)
CodeAlta.Plugins (plugin runtime, source builds, load contexts)
  ↓ provider packages depend on Agent
CodeAlta.Agent.OpenAI | CodeAlta.Agent.Anthropic | CodeAlta.Agent.Copilot |
CodeAlta.Agent.GoogleGenAI | CodeAlta.Agent.Mistral | CodeAlta.Agent.Xai
  ↓ plugin packages use Plugins.Abstractions
CodeAlta.Plugin.GitHub | CodeAlta.Plugin.Mcp | CodeAlta.Plugin.Statistics
```

### Startup Flow

1. `Program.cs` opens XenoAtom terminal session, starts plugin runtime
2. `CodeAltaOwnedServices.CreateAsync` creates `~/.alta` root, logging, model catalog, global config
3. `CodeAltaHost.CreateAsync` composes shared runtime services: catalogs, plugin runtime, skill catalog, session catalog, provider registry, AgentHub, SessionRuntimeService
4. `CodeAltaFrontendComposition.Create` wires TUI around services with `ShellStateStore` (immutable UI snapshot), shell controllers, coordinators, and event pump
5. Provider initialization and session catalog loading run independently — providers can probe while local sessions are visible

### Key Files and Classes

- **`CodeAlta/Program.cs`**: Entry point. Deferred app startup so XenoAtom main thread keeps UI bound. Handles plugin pre-start, CLI options, crash reporting
- **`CodeAlta.Agent/Runtime/AgentSession.cs`**: Core session implementation. Conversation replay from JSONL journal, event streaming via `Channel<AgentEvent>`, subscriber pattern with `ConcurrentDictionary<Guid, Action<AgentEvent>>`, per-turn state gate (`SemaphoreSlim`), compaction integration, tool invocation
- **`CodeAlta.Agent/Runtime/AgentRuntime.cs`**: CodeAlta-owned session runtime for raw-API sessions. Provider registry, model catalog, filesystem journal store with lazy initialization
- **`CodeAlta.Agent/IAgentSession.cs`**: Session contract: `SendAsync`, `SteerAsync`, `AbortAsync`, `CompactAsync`, `GetHistoryAsync`, event streaming/subscription
- **`CodeAlta.Agent/IModelProviderContracts.cs`**: Provider contracts: `IModelProviderAdapter`, `IModelProviderRuntime`, `IAgentModelProviderRuntime`, `IModelProviderSessionRuntime`, `IModelProviderTurnExecutor`
- **`CodeAlta.Agent/AgentEvent.cs`**: Polymorphic normalized event model with 13 derived types (content, activity, session updates, plans, interactions, permissions, errors, user-input requests). JSON-serialized with `$type` discriminator
- **`CodeAlta.Orchestration/Runtime/AgentHub.cs`**: Active session facade. Creates/resumes sessions from provider registry, owns `SessionEntry` with reference-counted disposal, per-session `AgentSessionCoordinator` with separate run/control gates
- **`CodeAlta.Orchestration/Runtime/SessionRuntimeService.cs`**: Central session lifecycle service. Owns per-session mailbox actors, instruction composition, skill activation, runtime event stream, config store
- **`CodeAlta.Orchestration/Runtime/Actors/SessionActor.cs`**: Per-session mailbox actor for serializing same-session mutations
- **`CodeAlta/App/CodeAltaFrontendComposition.cs`**: TUI wiring — view models, state store, events, controllers, coordinators, plugin bridges, alta dispatcher
- **`CodeAlta/App/ShellStateStore.cs`**: Immutable UI-session projection snapshot for selection, tabs, prompt sessions
- **`CodeAlta/Frontend/Commands/ShellCommandRegistry.cs`**: Shared command registry for command palette, command bar, shortcuts, help. Plugin commands adapt into the same registry

### Session Lifecycle

1. User/plugin/tool creates a global or project session
2. `SessionRuntimeService` resolves provider, project roots, instructions, tools, skill advertisements
3. `AgentHub` starts/resumes the CodeAlta session with selected provider runtime
4. Prompt sent through `IAgentSession.SendAsync`, normalized to `AgentEvent` values
5. Events persisted to sharded JSONL journal at `~/.alta/sessions/yyyy/MM/dd/<session-id>.jsonl`
6. Runtime events projected into tabs, timelines, sidebars, usage indicators, plugin projections
7. Busy-session sends queued; steering requests fall back to normal send when unsupported

### Mailbox Actor Model

CodeAlta uses small internal mailbox actors rather than a general actor framework:

- `SessionActor` owns same-session command ordering, lifecycle cancellation, supervisor decisions
- `OrchestrationMailboxActor` uses a bounded channel, completes replies on validation failures
- `BoundedRuntimeEventStream<TEvent>` uses newest-event drop policy when readers fall behind
- Different session actors run independently; commands for one session are serialized
- Akka.NET intentionally avoided — "Reconsider a full actor framework only after tests and measured complexity show that the local primitives no longer simplify orchestration ownership"

### Provider Architecture

Providers own protocol adaptation, credentials, readiness, and model metadata. They do NOT own persisted sessions:

- **`openai-chat`**: Streaming chat completions with strict function schema normalization
- **`openai-responses`**: Responses streaming over HTTP
- **`azure-openai`**: Azure OpenAI chat completions using deployment names as model ids
- **`codex`**: ChatGPT/Codex subscription via WebSocket with HTTP/SSE fallback. Per-provider `max_concurrent_requests` (default 16) prevents unbounded parallel requests
- **`copilot`**: Direct HTTP with device-flow or GitHub-token auth
- **`xai`**: Direct HTTP with PKCE browser OAuth or device-code OAuth against xAI API
- **`anthropic`**: SDK chat streaming with adaptive thinking support and model metadata enrichment
- **`google-genai`/`vertex-ai`**: Google GenAI SDK integration
- **`mistral`**: Mistral chat completions with streaming, tool calls, multi-turn replay

### Compaction System

Compaction is implemented in `CodeAlta.Agent.Runtime.Compaction` as a provider-call workflow, not a separate remote API:

- **Triggers**: Manual (`CompactAsync`), threshold (active context reaches `ratio * inputLimit`), overflow recovery (after context-limit failures)
- **Settings**: Enabled by default, trigger ratio 0.95, post-compaction target 10% of input limit, summary share 40% of target, file context 15% of summary target, keep last user message, allow split turns
- **Summarizer**: Ordinary provider turn with a specialized system prompt. Returns structured markdown with 9 required sections (Objective, Active User Request, Constraints, Progress, etc.)
- **Multi-pass**: Can recursively chunk and shrink oversized summaries (up to 4 passes)
- **Rehydration**: Activated skills can be rehydrated into composed instructions after compaction

### Plugin System

Trusted local .NET code loaded in-process:

- Source plugins at `~/.alta/plugins/<id>/plugin.cs` (global) or `<project>/.alta/plugins/<id>/plugin.cs` (project)
- Built with `dotnet build plugin.cs` using CodeAlta-generated build files
- Loaded into collectible `AssemblyLoadContext` for unload support
- Extends `PluginBase`, overrides contribution methods for commands, agents tools, prompt parts, UI, skill roots, MCP manifests, background tasks
- Instruction processors can inspect/replace final system/developer instructions before provider submission
- Built-in plugins: GitHub (`#` issue lookup), Statistics (transient per-turn stats), MCP (progressive tool exposure)

### The `alta` Live Tool

An in-process command gateway exposed to agent sessions as a tool:

- CLI-style arguments: `{"args": ["session", "status", "<id>"], ...}`
- Built-in command groups: `project`, `session`, `skill`, `provider`, `model`, `plugin`, `tool`, `version`, `ask`, `notes`, `reminder`, `prompt`
- Plugins extend via `PluginBase.GetAltaCommands()`
- Returns compact JSONL headed by `alta.result` record
- `alta ask` command for structured user questions with file review support
- `alta session create --same-model-as <id>` for delegated child sessions

### Skills

Agent Skills-compatible directory format with `SKILL.md` frontmatter:

- Roots: project/user filesystem, built-ins, plugin resources
- Shadowing: higher-precedence skills hide lower (project > user > built-in)
- Enablement: enabled by default, disable in TOML `[skills].disabled = ["name"]`
- Activation: resolves valid unshadowed skill, builds payload, records in session journal, injects into agent session
- Post-compaction rehydration: keeps skill guidance without duplicating context

### Tool System

Built-in tools with JSON Schema definitions:

- `read_file`, `list_dir`, `grep`, `webget`, `shell_command`
- `write_file`, `replace_in_file`, `delete_file_or_dir`, `rename_file_or_dir`, `apply_patch`
- Mutation/shell tools flow through host permission handling
- MCP tools registered as `mcp__<server>__<tool>` aliases via progressive session activation
- Tool schemas normalized for OpenAI strict function-schema requirements

### State and Persistence

- Session journals: `~/.alta/sessions/yyyy/MM/dd/<session-id>.jsonl` — replayable normalized events
- SQLite projection cache: `~/.alta/cache/cache.sqlite3` — fast listing, journals remain source of truth
- Config: `~/.alta/config.toml` (global) + `<project>/.alta/config.toml` (project overlay)
- MCP config: `~/.alta/mcp.json` + `<project>/.alta/mcp.json` (JSON, separate from TOML policy)
- UI state: `~/.alta/ui-state.yaml`
- Plugin state: plugin-owned via `IPluginStateStore`

### Tests

~55K lines of tests across multiple test projects. Key test files:
- `AgentSessionTests.cs` (4,551 lines) — core session behavior
- `OpenAIRawApiModelProviderRuntimeTests.cs` (4,652 lines) — OpenAI provider integration
- `AltaLiveToolTests.cs` (4,342 lines) — in-session command gateway
- `ArchitectureGuardrailTests.cs` (2,121 lines) — enforces layer boundaries, forbids legacy patterns, enforces terminology conventions
- `AgentToolsTests.cs` (1,623 lines) — built-in tool behavior

## Key Techniques

### Provider-Agnostic Session Ownership

Sessions are CodeAlta-owned, not provider-owned. The session id is generated before provider attachment, and providers must preserve it (returning a different id is a contract violation). This enables:
- **Provider switching**: Sessions can replay canonical CodeAlta history to a different compatible provider
- **Independent lifecycle**: Session listing doesn't require provider readiness; local history loads before providers probe
- **Queue and delegate**: Busy sessions queue prompts; child sessions created with `--same-model-as` inherit model config

This is unlike Claude Code's agent sessions which are tightly coupled to a provider session, or Codex CLI which treats sessions as provider-owned. CodeAlta's approach is the most provider-agnostic in the open-source agent workspace space.

### Compaction as Provider Turn

Rather than implementing a separate summarization API or requiring a "compaction model," CodeAlta runs compaction as an ordinary provider turn through the same turn executor. The summarizer prompt is a carefully structured 9-section template:

```markdown
## Objective
## Active User Request
## Constraints
## Progress (Done / In Progress / Blocked)
## Decisions
## Next Steps
## Critical Context
## Relevant Files
```

This means:
- Any provider that supports chat completions can do compaction
- No separate model or API endpoint required
- Compaction is replayable from the journal like any other turn
- Multi-pass shrinking handles oversized summaries (recursively chunking up to 4 passes)
- Post-compaction skill rehydration preserves skill guidance

The trade-off: compaction consumes provider tokens, but the simplicity is worth it.

### Bounded Event Streams with Drop Policy

`BoundedRuntimeEventStream<TEvent>` uses a bounded channel with newest-event drop. Slow readers (UI, plugins) don't create unbounded memory pressure. Dropped events are counted for diagnostics. This is a deliberate choice against unbounded buffering — prevents the "runaway memory in long sessions" problem that plagues many agent frameworks.

### Lightweight Mailbox Actors Without a Framework

Instead of pulling in Akka.NET or Orleans, CodeAlta implements just three internal classes:
- `OrchestrationMailboxActor` with bounded channel and structured completion
- `SessionActor` for per-session serialization
- `SessionActorRegistry` for lifecycle management

Public APIs use request records, session ids, snapshots, handles, and events. Actor internals are never exposed. The decision was explicit: "Reconsider a full actor framework only after tests and measured complexity show that the local primitives no longer simplify orchestration ownership."

### Reference-Counted Session Disposal with Idle Drain

`AgentHub.SessionEntry` uses reference counting for safe concurrent access. `DisposeAsync` waits for all references to drain before cleaning up, and aborts the session first as a best-effort signal. This pattern avoids the "session disposed while a tool is still running" race condition that many agent runtimes hit.

### Source Plugin Build System

Plugins are C# source files that CodeAlta compiles at startup:
1. Generates `Directory.Build.props`, `Directory.Build.targets`, `Directory.Packages.props`, `global.json`
2. Runs `dotnet build plugin.cs` from the plugin package directory
3. Loads output assembly into collectible `AssemblyLoadContext`
4. Records build manifest for future loads (skip rebuild when source unchanged)

This is unusual — most agent frameworks use scripting (Python, JavaScript) or MCP for extensibility. CodeAlta's approach gives plugins full access to the host process (trusted model) while maintaining the ability to unload and rebuild.

### Dual JSON+TOML Configuration for MCP

MCP server connection fields live in JSON (`mcp.json`), while policy lives in TOML (`config.toml`). This separation means:
- Enable/disable is a TOML mutation, not a JSON rewrite
- JSON files stay compatible with Claude Desktop, VS Code, etc.
- Policy overlay: global → project, with per-server enablement, tool allow/deny lists, timeouts

### Progressive MCP Tool Exposure

MCP tools aren't all loaded at startup. Instead:
1. Prompt shows compact active/inactive server inventory
2. `alta mcp activate <server>` marks servers active for the session
3. On next agent run, activated servers are connected, tools enumerated, policy filters applied
4. Tools exposed as `mcp__<server>__<tool>` aliases

This avoids connecting to every configured MCP server on every agent run.

## Design Decisions

### In-Process Over Isolation (Deliberate)

CodeAlta loads plugins and skills as in-process code. This is fast but trusts the code. The docs are explicit: "Source plugins are trusted code. Building a plugin can execute SDK, NuGet, and MSBuild logic." There's a safe mode (`--no-plugins`, `CODEALTA_DISABLE_PLUGINS=1`) for recovery, but no sandbox. Contrast with Claude Code's MCP-based tool isolation or OpenCodeReview's per-file concurrent subagents — CodeAlta chooses power and simplicity over security boundaries.

### Terminal-First Over GUI/Web

The entire workspace is a terminal UI (XenoAtom.Terminal). This is a deliberate aesthetic and functional choice: keyboard-first navigation, no browser required, works over SSH, fits the developer workflow. The TUI approach means every visual element (tabs, sidebar, timeline, command palette) is rendered in terminal cells. This is similar to Claude Code's terminal approach but with a much richer multi-pane workspace.

### Session Journal as Source of Truth

The JSONL journal is the canonical record. The SQLite cache is a projection — it can be rebuilt from journals. This is the same pattern as event sourcing: journals are append-only, the cache is derived, and if the cache corrupts, journals rebuild it. Unlike Claude Code which stores session state in a proprietary format, CodeAlta's JSONL journals are human-readable and provider-independent.

### Provider Type Registry with Forced Compatibility

Provider types are registered in `ConfiguredModelProviderRegistryBuilder` by string key (e.g., `openai-chat`, `anthropic`). The code explicitly checks for provider compatibility and refuses to start a session that would produce a different id. This is stricter than most agent frameworks which silently fall back or remap.

### Skills as Filesystem Directories, Not Code

Skills are `SKILL.md` files, not executable code. They're read, validated, shadowed, and injected as context. This is compatible with the Agent Skills specification and keeps skills portable across tools. The UI provides scaffold, browse, enable/disable, and activation. Contrast with Claude Code's skills which are executable markdown files with tool access.

### Monorepo with Strict Layer Enforcement

Architecture guardrail tests (2,121 lines in `ArchitectureGuardrailTests.cs`) enforce:
- No legacy `IAgentBackend` terminology (must use `ModelProvider`/`Session`)
- No UI code calling runtime services directly
- No session state mutation outside mailbox actors
- No broad UI callback regressions
- No `WorkThread`/`ThreadId` terminology in conversation concepts
- Frontend/runtime separation

This is unusually rigorous for a solo-maintainer project and reflects the codebase's maturity.

## Comparison Notes

### vs. Claude Code
CodeAlta is provider-agnostic where Claude Code is Anthropic-first. Both use terminal UIs and file-based configuration. CodeAlta has more explicit session management (create, queue, steer, compact as separate commands vs. Claude Code's implicit session model). CodeAlta's plugin system loads .NET code in-process; Claude Code uses MCP tools and hooks.

### vs. Codex CLI
Both are CLI-based coding agents, but CodeAlta is a full TUI workspace while Codex CLI is a simpler command-line tool. CodeAlta's session model is more sophisticated (provider switching, journal replay, compaction). CodeAlta supports Codex as a *provider type* rather than being a Codex-specific tool.

### vs. Open Interpreter / OpenCodeReview
CodeAlta has a richer extensibility model (plugins, skills, live tool) and provider abstraction layer. Open Interpreter is Python-based and simpler. OpenCodeReview focuses on code review specifically.

### vs. Pi Coding Agent
Both are terminal-based. Pi is minimal (four tools, no MCP, full YOLO mode). CodeAlta is feature-rich (many built-in tools, MCP support, plugin system, multi-provider). They represent opposite points on the minimal-vs-comprehensive spectrum.

### vs. Paca
Both have multi-agent session management. Paca uses WASM plugin sandbox, Docker-sandboxed agent execution, and Scrum metaphors. CodeAlta uses in-process .NET plugins, provider-agnostic sessions, and a more traditional development workflow model. CodeAlta's live tool and delegated work model is more integrated into the coding workflow.

### vs. Browser Use / Webwright
CodeAlta is terminal-native, not browser-based. It's designed for code editing and project work, not web automation.

### vs. Zeroclaw
Zeroclaw is Rust-based with OS-level sandboxing. CodeAlta is C#/.NET with trust-based plugin loading. Zeroclaw prioritizes security; CodeAlta prioritizes developer experience and integration depth.

## Weaknesses and Gaps

1. **Windows/.NET dependency**: Requires .NET 10 SDK. Non-Windows support exists but the plugin build system targets .NET specifically.
2. **Trust model**: Plugins are trusted code loaded in-process. No sandboxing beyond "trust the author." Safe mode is binary (all plugins on/off) not granular.
3. **Single maintainer**: Despite the codebase's maturity, it's primarily a solo project by Alexandre Mutel.
4. **Pre-release**: Explicitly marked as pre-release software with possible breaking API/config changes before 1.0.
5. **MCP tool-list-changed**: Notifications are deferred/follow-up work. Currently requires explicit re-activation to refresh tools.
6. **No built-in eval framework**: Unlike some agent platforms, there's no built-in evaluation harness.
7. **Provider coverage gaps**: No direct Ollama/local provider type (though OpenAI-compatible endpoints can bridge this).
