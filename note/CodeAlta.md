# CodeAlta

A terminal-based AI coding agent workspace built in C#/.NET that integrates multi-provider LLM access, durable project-scoped sessions, trusted local plugins, and keyboard-driven project navigation behind a single `alta` CLI command. Created by Alexandre Mutel (xoofx), it's the most architecturally sophisticated open-source terminal agent workspace — notable for its provider-agnostic session model, JSONL journal-based persistence, in-process .NET plugin system, and compaction-as-provider-turn design.

---

## Architecture

CodeAlta is a .NET 10 monorepo of ~25 projects in a strictly layered architecture:

```
CodeAlta (TUI frontend, XenoAtom.Terminal)
  ↓
CodeAlta.Orchestration (session lifecycle, runtime events, instruction composition)
  ↓
CodeAlta.Agent (provider contracts, session runtime, compaction, tools)
CodeAlta.Catalog (project/session/skill catalogs, config store)
CodeAlta.Plugins (plugin runtime, source builds, load contexts)
```

The frontend is a terminal UI built on **XenoAtom.Terminal**, with views, dialogs, and a shared command registry. The frontend never calls runtime services directly — all interaction goes through immutable shell commands, coordinators, and an event pump.

**Session lifecycle** (`src/CodeAlta.Agent/Runtime/AgentSession.cs`, `src/CodeAlta.Orchestration/Runtime/SessionRuntimeService.cs`): Sessions are CodeAlta-owned, not provider-owned. The session id is generated before provider attachment. Sessions replay from JSONL journals at `~/.alta/sessions/yyyy/MM/dd/<session-id>.jsonl`. A SQLite projection cache at `~/.alta/cache/cache.sqlite3` provides fast listing, but journals remain the source of truth.

**Concurrency model** (`src/CodeAlta.Orchestration/Runtime/Actors/`): Per-session mailbox actors serialize same-session mutations. `SessionActor` owns command ordering and lifecycle. `BoundedRuntimeEventStream<TEvent>` uses a bounded channel with newest-event drop — slow readers don't create memory pressure. The codebase explicitly avoids Akka.NET: "Reconsider a full actor framework only after tests and measured complexity show that the local primitives no longer simplify orchestration ownership."

**AgentHub** (`src/CodeAlta.Orchestration/Runtime/AgentHub.cs`): The active session facade. Creates/resumes sessions from provider registry, owns `SessionEntry` with reference-counted disposal — waits for all references to drain before cleanup, with best-effort abort signaling. This avoids the "session disposed while tool is running" race.

**Provider abstraction** (`src/CodeAlta.Agent/IModelProviderContracts.cs`): Providers own protocol adaptation, credentials, and model metadata. They do NOT own persisted sessions. Nine built-in provider types: `openai-chat`, `openai-responses`, `azure-openai`, `codex` (ChatGPT subscription via WebSocket), `copilot` (direct HTTP), `xai` (PKCE OAuth), `anthropic`, `google-genai`/`vertex-ai`, and `mistral`.

**Plugin system** (`src/CodeAlta.Plugins/`, `src/CodeAlta.Plugins.Abstractions/`): Trusted local .NET source files at `~/.alta/plugins/<id>/plugin.cs`. Compiled at startup with CodeAlta-generated build files, loaded into collectible `AssemblyLoadContext`. Plugins extend `PluginBase` and contribute commands, agent tools, prompt parts, UI elements, skill roots, MCP manifests, and instruction processors. Safe mode (`--no-plugins`) disables all plugins.

**The `alta` live tool** (`src/CodeAlta.LiveTool/`): An in-process command gateway exposed to agent sessions as a tool. Accepts CLI-style JSON arguments, returns compact JSONL. Built-in groups: `project`, `session`, `skill`, `provider`, `model`, `plugin`, `tool`, `ask`, `notes`, `reminder`, `prompt`, `version`. Plugins extend via `PluginBase.GetAltaCommands()`.

## Key Techniques

### Provider-Agnostic Sessions

Sessions are CodeAlta-owned with a canonical id generated before provider attachment. Providers MUST preserve the id (returning a different one is a contract violation). This enables provider switching by replaying canonical history to a different compatible provider. Unlike Claude Code (tightly coupled to backend sessions) or Codex CLI (provider-owned sessions), CodeAlta's model is truly provider-independent.

### Compaction as Provider Turn

Rather than a separate summarization API, compaction runs as an ordinary provider turn through the same executor. The summarizer prompt is a structured 9-section template (Objective → Active User Request → Constraints → Progress → Decisions → Next Steps → Critical Context → Relevant Files). Any provider that supports chat completions can compact. Multi-pass shrinking handles oversized summaries (up to 4 recursive passes). Post-compaction, activated skills are rehydrated into composed instructions so skill guidance survives.

### Source Plugin Compilation

CodeAlta generates MSBuild files, invokes `dotnet build plugin.cs`, and loads the output assembly into a collectible context. This gives plugins full process access (trusted model) while supporting unload and rebuild. Build manifests skip recompilation when sources are unchanged. Most agent frameworks use scripting or MCP for extensibility; CodeAlta's approach is unique.

### Dual JSON+TOML MCP Configuration

MCP server connections live in JSON files (`mcp.json`) for editor compatibility. Enablement, tool policy, timeouts, and output caps live in TOML (`config.toml`). This separation means policy changes don't rewrite JSON, and JSON files stay compatible with Claude Desktop and VS Code. Progressive tool exposure: `alta mcp activate <server>` marks servers for the current session, tools are enumerated on the next agent run.

### Instruction Processing Pipeline

Plugin instruction processors (`PluginBase.GetInstructionProcessors()`) can inspect or replace final system/developer instructions before provider submission. Processors run in deterministic contribution order, receive full context metadata, and return `Continue`, `Replace(...)`, or `Cancel(...)`. Replacement results include audit-safe change summaries only — no secret-bearing text in the manifest. This is an extensibility/audit mechanism, not a security boundary.

### Bounded Event Streams

`BoundedRuntimeEventStream<TEvent>` drops oldest events when readers fall behind, counting drops for diagnostics. This prevents runaway memory in long sessions — a deliberate choice against unbounded buffering that plagues many agent frameworks.

## Design Decisions

**In-process over isolation**: Plugins are trusted code loaded in-process. Fast and powerful, but no sandbox. Safe mode is binary (all on/off), not granular. Contrast with WASM-based plugin systems (Paca) or OS-level sandboxing (Zeroclaw).

**Terminal-first over GUI/web**: The entire workspace is a TUI — keyboard-first, works over SSH, no browser required. This is a deliberate aesthetic choice shared with Claude Code and Pi, but CodeAlta's multi-pane workspace (tabs, sidebar, timeline, command palette) is richer than either.

**Journal as source of truth**: JSONL journals are append-only and provider-independent. The SQLite cache is derived and rebuildable. This is event sourcing applied to agent conversations — human-readable, replayable, portable.

**Strict layer enforcement**: Architecture guardrail tests (~2,100 lines) verify that no UI code calls runtime services directly, legacy terminology is banned, and frontend/runtime separation holds. Unusually rigorous for a solo-maintainer project.

**Mailbox actors without a framework**: Three internal classes (`OrchestrationMailboxActor`, `SessionActor`, `SessionActorRegistry`) provide per-session serialization without pulling in Akka.NET. Public APIs use request records and events — actor internals are never exposed.

## Comparison Notes

- **vs. [[Claude Code Mastery]]**: CodeAlta is provider-agnostic; Claude Code is Anthropic-first. CodeAlta has explicit session management (create, queue, steer, compact commands); Claude Code has an implicit model. CodeAlta's plugin system loads .NET in-process; Claude Code uses MCP tools and hooks.

- **vs. [[Pi Coding Agent]]**: Both terminal-based. Pi is minimal (four tools, no MCP, YOLO mode). CodeAlta is feature-rich (many built-in tools, MCP, plugins, multi-provider). They represent opposite ends of the minimal-vs-comprehensive spectrum.

- **vs. [[Paca]]**: Both have multi-agent session management. Paca uses WASM sandboxing and Scrum metaphors; CodeAlta uses in-process .NET plugins and provider-agnostic sessions. CodeAlta's live tool and delegated work model are more deeply integrated into the coding workflow.

- **vs. [[Zeroclaw]]**: Zeroclaw is Rust with OS-level sandboxing; CodeAlta is C# with trust-based plugins. Zeroclaw prioritizes security; CodeAlta prioritizes developer experience.

- **vs. [[Components of a Coding Agent]]**: CodeAlta exemplifies the "harness matters more than the model" thesis. Its session journal, compaction, plugin, and tool systems form a sophisticated harness that works across nine provider types.

---
*Source: [[summary/CodeAlta]]*
*Last updated: 2026-07-05*
*Tags: #tool #project #agents #terminal #coding-agent*
