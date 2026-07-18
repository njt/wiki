---
url: https://github.com/xai-org/grok-build
title: Grok Build
author: SpaceXAI (xAI)
date_fetched: 2026-07-18
date_published: 2025
---

# Grok Build — Deep Architectural Analysis

Grok Build (`grok`) is SpaceXAI's terminal-based AI coding agent. Written in Rust, it syncs periodically from the SpaceXAI monorepo. It runs as a full-screen TUI, headlessly for scripting/CI, or embedded in editors via the Agent Client Protocol (ACP). This analysis is based on reading the source code directly — not just the README.

## Project Scale

The repository is a Rust workspace with ~80+ crates organized across three directories:
- `crates/codegen/` — 50+ product crates (the actual application)
- `crates/common/` — 9 shared infrastructure crates
- `third_party/` — 4 vendored dependencies (Mermaid diagram stack, graph libraries)
- `prod/mc/` — shared types crate for the monorepo chat proxy

Total code: well over 1M lines of Rust. The largest crates: xai-grok-pager (425K lines), xai-grok-shell (338K lines), xai-grok-tools (112K lines), xai-grok-workspace (78K lines), xai-grok-agent (21K lines), xai-grok-markdown (21K lines).

## Architecture

### Crate Hierarchy (Bottom-Up)

**Foundation layer** (`crates/common/`):
- `xai-tool-protocol` (6.6K lines): Wire-protocol types. JSON-RPC 2.0 envelope, identifier newtypes (ToolId, SessionId, ServerId, ConnectionId), handshake protocol, hook events, notification schemas, MCP block types. The protocol is the contract between tool servers and the runtime.
- `xai-tool-runtime` (5.4K lines): The `Tool` trait and its streaming execution contract. Defines `ToolStream<T>` as `[Progress*, Terminal]` invariant. Provides `ToolDyn` for type erasure and `ArcTool`/`ArcToolFamily` convenience aliases.
- `xai-tool-types` (3.6K lines): Shared tool taxonomy, task description builders, schema utilities, compat layer.
- `xai-circuit-breaker` (2.2K lines): Circuit breaker pattern for tool calls — config, registry, window, retry policy, observer.
- `xai-tracing` (2K lines): OpenTelemetry/fastrace integration covering gRPC, HTTP, and tokio spans.
- `xai-grok-compaction` (6.8K lines): Transport-agnostic compaction engine with three styles: code_compaction (full-replace for coding agents), intra_compaction (Gro chat tail-keep), inter_compaction (Gro chat between-turn).
- `xai-interjection-core` (320 lines): Mid-turn interjection buffer. PendingInterjection → FormattedInterjection, drained at safe points as synthetic user messages.
- `xai-computer-hub-mcp-adapter`: Bridges the Computer Hub protocol to MCP.

**Product layer** (`crates/codegen/`):

*Agent Definition & Assembly*:
- `xai-grok-agent` (21K lines): The `Agent` type — bundles definition, system prompt, tool bridge, policies. `AgentBuilder` is the fluent 10-step construction API. `AgentDefinition` parses from markdown files with frontmatter. Tool allowlist/denylist resolution mapping compat tool names (Claude's "Read", "Bash", "Grep") to Grok ToolKind. Subagent type classification via regex pattern `Agent(type1, type2)`.
- `xai-grok-subagent-resolution` (2.6K lines): Pure resolution logic for subagent spawning — effective runtime config via precedence (explicit > role > persona > parent), persona loading, resume identity validation.
- `xai-agent-lifecycle` (608 lines): Agent lifecycle management.

*Shell & Session*:
- `xai-grok-shell` (338K lines): The agent runtime. Session management, model interaction (sampling), subagent spawning, ACP sessions, terminal integration, Claude import, plugin system, leader/stdio/headless entry points. The largest and most complex crate.
- `xai-grok-shell-base` (2.8K lines): CPU profiling, env utilities shared with xai-grok-shell.
- `xai-grok-shell-session-support`: Session support utilities.

*Chat State*:
- `xai-chat-state` (13K lines): Actor-based conversation state management. `ChatStateActor` runs in a dedicated tokio task. Commands: push_user, pop, insert_system, replace_system, get_conv, usage. Events emitted on state changes. Compaction mode management. Persistence via `ChatPersistence` trait.

*TUI*:
- `xai-grok-pager` (425K lines): The full-screen terminal UI — scrollback, prompt, modals, rendering. Built on ratatui.
- `xai-grok-pager-bin`: Composition-root package, builds the `xai-grok-pager` binary.
- `xai-grok-pager-render`: Rendering subsystem.
- `xai-grok-pager-minimal`: Minimal pager variant.
- `xai-ratatui-inline`, `xai-ratatui-textarea`: Ratatui widget extensions.

*Tools*:
- `xai-grok-tools` (112K lines): Tool implementations — terminal (bash), file read/write/edit, search (grep/glob), web search, web fetch, LSP, image generation, video generation, deploy, memory search/get, task (subagent spawning), plan mode, ask user question. Also the `ToolBridge` — central registry routing tool calls, managing tool state.
- `xai-grok-tools-api`: Tool API definitions.
- `xai-grok-tools`: Implementations for opencode and codex compat tools included.

*Workspace*:
- `xai-grok-workspace` (78K lines): Host filesystem, VCS, execution, checkpoints. Permission system, folder trust, file watcher (fs_notify), session management, recovery, daemonization, worktree support, hub (multi-session coordination), upload, telemetry.
- `xai-grok-workspace-client`, `xai-grok-workspace-types`: Client and type definitions.

*Memory*:
- `xai-grok-memory` (9.9K lines): Markdown-based persistent memory. Layout: `~/.grok/memory/MEMORY.md` (global) + `{workspace_hash}/MEMORY.md` (project) + session logs. Features: SQLite vector index (sqlite-vec), chunking, embedding, MMR (maximal marginal relevance) retrieval, query expansion, dream (consolidation), archive, watcher.

*Codebase Intelligence*:
- `xai-codebase-graph` (9.7K lines): Tree-sitter based code graph. Go-to-definition, go-to-references, full and incremental indexing, parallel parsing via rayon, memory-mapped I/O, channel-based incremental updates via IndexManager.
- `xai-grok-markdown` (21K lines): Markdown parsing and rendering.
- `xai-grok-markdown-core`: Core markdown utilities.
- `xai-grok-mermaid` (in-code): Mermaid diagram support, backed by the vendored mermaid-to-svg crate.

*MCP & Protocol*:
- `xai-grok-mcp` (10.5K lines): MCP (Model Context Protocol) server integration.
- `xai-acp-lib` (2.3K lines): Agent Client Protocol library — channel, gateway, message, line reader, stdin reader, normalization.

*Config & Auth*:
- `xai-grok-config` (6.9K lines): Multi-layer config merging: `/etc/grok/` → `$GROK_HOME/` → macOS MDM. Signed policy support (Ed25519). Campaign management, version overrides.
- `xai-grok-config-types`: Config type definitions.
- `xai-grok-auth`: Authentication (browser-based OAuth on first launch).
- `xai-grok-secrets`: Secrets management.

*Hooks & Plugins*:
- `xai-grok-hooks` (8.5K lines): Runtime hook system. File-based discovery from `~/.grok/hooks/` and project `.grok/hooks/`. Four event types: session_start, pre_tool_use, post_tool_use, session_end. JSON-defined, executed as child processes. pre_tool_use hooks can deny/allow.
- `xai-hooks-plugins-types`: Shared types for hooks and plugins.
- `xai-grok-plugin-marketplace`: Plugin marketplace integration.

*Other*:
- `xai-grok-sandbox` (4.5K lines): Sandboxing support.
- `xai-grok-telemetry`: Telemetry and logging.
- `xai-grok-update`: Self-update mechanism.
- `xai-grok-voice`: Voice input integration.
- `xai-grok-announcements`: Release announcements.
- `xai-grok-models`: Model definitions and metadata.
- `xai-grok-sampler`, `xai-grok-sampling-types`: Model sampling.
- `xai-prompt-queue` (176 lines): Prompt queuing.
- `xai-sqlite-journal`: SQLite journal integration.
- `xai-token-estimation`: Token counting.
- `xai-system-power`: System power management.
- `xai-crash-handler`: Crash reporting.
- `xai-fast-worktree`: Fast git worktree operations.
- `xai-file-utils`, `xai-fsnotify`, `xai-gix-status`, `xai-hunk-tracker`, `xai-tty-utils`: Utility crates.
- `mixpanel`: Mixpanel analytics integration.

### Key Architectural Patterns

**1. The Tool Trait & Type Erasure**

The `Tool` trait (`xai-tool-runtime/src/tool.rs`) is the central abstraction. Every tool implements:
- `id() -> ToolId`: stable identity for dispatch routing
- `description() -> ToolDescription`: model-facing description with JSON Schema args
- `execute(ctx, args) -> ToolStream<Output>`: streaming entry point
- `run(ctx, args) -> Result<Output, ToolError>`: blocking convenience hook

The streaming model is elegant: `ToolStream<T> = Pin<Box<dyn Stream<Item = ToolStreamItem<T>>>>` with a strict invariant — zero or more `Progress` items followed by exactly one `Terminal`. `terminal_only(result)` and `with_progress(progress_stream, terminal_future)` are the two supported constructors.

Type erasure happens via `ToolDyn` (auto-implemented for every `T: Tool`), which deserializes JSON args, calls `execute`, and re-serializes the output into `TypedToolOutput` (bundling JSON value, model-facing content blocks, and optional chat-completion response).

**2. ToolBridge — Central Registry**

`ToolBridge` (`xai-grok-tools/src/bridge.rs`) is the runtime's central tool registry. It owns:
- Tool registry (registered tools by ID)
- Session context (cwd, fs backend, terminal backend, notification handle)
- Resources (type-safe resource container with read_resource::<T>())
- Tool state (disabled tools, tool name overrides, completion tracking, retry config)
- Skill manager (discovered skills, announced names, pending baseline changes)
- AgentsMd tracker (AGENTS.md file watching)
- Gitignore filter

The bridge uses a builder pattern: `ToolBridge::get_builder()` → configure → `ToolBridge::finalize_builder()`. This two-phase initialization separates registration from activation.

**3. AgentBuilder — 10-Step Fluent Construction**

`AgentBuilder` (`xai-grok-agent/src/builder.rs`) constructs an `Agent` in 10 sequential steps:
1. Resolve definition (from file or programmatic)
2. Discover skills (filesystem + plugin registry)
3. Resolve preloaded skills
4. Build tool config (inject default tools, apply allowlist/denylist, resolve compat names)
5. Merge tool params (bash, ask_user_question, web_fetch overrides)
6. Resolve MCP tool access
7. Apply session-level tool clamping
8. Finalize ToolBridge
9. Read AGENTS.md files
10. Seed skill discovery + agents_md + gitignore into bridge

The allowlist resolution is particularly nuanced: compat names like "Read", "Bash", "Grep", "Edit" map to Grok's `ToolKind` enum, falling back to the full toolset if any entry is unrecognized (safety over restriction). MCP tools (`mcp__*` prefix) are always allowed. Subagent type directives (`Agent(type1, type2)`) are parsed via regex and constrain which subagent types can be spawned.

**4. Compaction — Three Strategies, Transport-Agnostic**

`xai-grok-compaction` (`crates/common/xai-grok-compaction/src/lib.rs`) defines three compaction styles:

- **code_compaction** (full-replace): Grok Build's approach. The entire conversation is summarized by a dedicated compaction model. The summary replaces the full history. Includes self-summarization prompt templates, summary quality validation (degeneracy detection, minimum seed chars), and failure classification (HTTP status → retry policy, stream event error → strategy selection).

- **intra_compaction** (tail-keep): Grok Chat's per-step pass. Keeps recent turns, summarizes older ones.

- **inter_compaction** (between-turn): Grok Chat's chunked, between-turn pass.

The crate abstracts over the host through trait seams: `CompactionItem`, `CompactionSampler`, `ItemTokenCounter`, and observer traits for metrics.

**5. Interjection System — Mid-Turn User Messages**

`xai-interjection-core` (`crates/common/xai-interjection-core/src/buffer.rs`) is a small but architecturally significant component. When a user sends a message while the agent is mid-turn, the message goes into an `InterjectionBuffer` (backed by `EventQueue`). At the next safe drain point, `drain_formatted()` wraps each entry as a synthetic user message (`<user_query>`) with the framing "The user sent a message while you were working." Entries are FIFO, one message per entry, never merged.

**6. ChatStateActor — Actor-Based Conversation State**

`xai-chat-state` uses the actor pattern: `ChatStateActor` owns conversation state in a dedicated tokio task. All access goes through `ChatStateHandle` (command + oneshot channel). This eliminates locks on the conversation vector — only one task ever touches it. Events (`ChatStateEvent`) are emitted on state changes for observers (TUI, logging, telemetry).

**7. Subagent Spawning — Resolution & Dispatch**

Subagent spawning involves two crates:
- `xai-grok-subagent-resolution`: Pure resolution logic — effective runtime config via explicit > role > persona > parent precedence. Persona instruction loading, resume identity validation.
- `xai-grok-shell`: Actual spawning — worktree creation, session initialization, model routing.

The task tool description is built dynamically from discovered subagents (built-in + user-defined from `.grok/agents/` + plugin agents). Built-in subagents (general-purpose, explore, plan) get hardcoded tool-name fragments; user-defined agents use their raw markdown description verbatim.

**8. Memory System — Markdown + SQLite + Vectors**

`xai-grok-memory` implements a three-tier memory architecture:
- **MEMORY.md files**: Markdown files in `~/.grok/memory/` (global and per-workspace). Human-readable, git-friendly.
- **Session logs**: `YYYY-MM-DD-{slug}-{sid8}.md` storing conversation digests.
- **Vector index**: SQLite with sqlite-vec extension for semantic search. Chunks are embedded in batches of 32, stored for MMR retrieval.

The `dream` module handles consolidation across sessions — extracting durable facts from session logs into MEMORY.md. The `watcher` module provides filesystem watching for live MEMORY.md changes. Feature-gated behind `--experimental-memory` / `GROK_MEMORY=1`.

**9. Codebase Graph — Tree-Sitter Indexing**

`xai-codebase-graph` builds a code graph using tree-sitter queries. Features:
- Parallel parsing via rayon
- Memory-mapped index files (fast loading)
- Channel-based incremental updates via `IndexManager` (spawned as a background task)
- Go-to-definition and go-to-references queries
- Scope graph analysis (`scope_graph` module)
- String interning for memory efficiency

**10. Config System — Multi-Layer Merge**

Config loading merges 6 layers (lowest to highest priority):
1. `/etc/grok/managed_config.toml`
2. `$GROK_HOME/managed_config.toml`
3. `$GROK_HOME/config.toml`
4. `$GROK_HOME/requirements.toml` (Ed25519-signed)
5. `/etc/grok/requirements.toml`
6. macOS MDM managed preferences

Each layer applies `[[version_overrides]]` before merging. Signed policy support enables enterprise-managed configuration.

**11. Hooks System — File-Based, Process-Backed**

Hooks are discovered from `~/.grok/hooks/` and `<git-worktree-root>/.grok/hooks/`. Defined as JSON files, executed as child processes. Four events: `session_start`, `pre_tool_use`, `post_tool_use`, `session_end`. `pre_tool_use` hooks can deny tool execution (blocking); all others are non-blocking. Fail-open by default.

**12. Mermaid Diagram Rendering — Vendored Rust Implementation**

The vendored `third_party/mermaid-to-svg` crate is a from-scratch Rust implementation of Mermaid diagram parsing and SVG rendering. Supports: flowchart, sequence, class, state, ER, Gantt, pie, journey, gitgraph, requirement, quadrant, timeline, mindmap, sankey, kanban, block, C4, radar, xychart, packet, info diagrams. Uses graphlib_rust (graph algorithms) and dagre_rust (hierarchical layout via network simplex) as vendored dependencies.

## Key Techniques

### Streaming Tool Execution with Progress

The `ToolStream<T>` design enforces a structural invariant: `[Progress(_)*, Terminal(Result<T, ToolError>)]`. This means tool consumers can stream progress updates (text chunks, content blocks, or custom payloads) to the UI while the tool executes, then receive exactly one terminal result. The `with_progress()` constructor chains a progress stream with a terminal future — the terminal is only awaited after the progress stream drains, preventing conflicts.

### Compat Tool-Name Mapping

The `claude_tool_kind()` function in `builder.rs` maps vendor tool names to Grok's `ToolKind` enum. This enables agents defined with Claude-compatible tool lists (e.g., `tools: Read, Bash, Grep, Edit`) to work correctly in Grok Build without modification. The mapping is centralized in `xai-grok-tools/src/tool_taxonomy.rs`. Unrecognized entries cause the builder to fall back to the full toolset rather than silently stripping tools — safety over restriction.

### Template Variable System

Tool descriptions and system prompts use `${{ template.variables }}` syntax. `TemplateRenderer` resolves these at finalize time. Example: `${{ tools.by_kind.task }}` resolves to the task tool's function name, `${{ params.task.subagent_type }}` resolves to the subagent_type parameter name. This decouples description authoring from concrete tool naming — descriptions reference semantic categories, not specific tool IDs.

### Dual Hosted-Tool + Function-Tool Architecture

Some tools (web search, X search) can be sent either as:
- **Hosted tools**: Native Responses API types executed server-side by the agentic sampler
- **Function tools**: Local tool implementations executed client-side

The choice is controlled by `backend_search` toggle, ANDed at request time with per-model `supports_backend_search`. Hosted tools skip the tool-call round-trip — the sampler integrates search results directly into the response stream.

### Session-Level Tool Clamping

Tools can be clamped at two levels:
1. **Agent definition**: `tools` (allowlist) and `disallowed_tools` (denylist)
2. **Session**: `session_tools_allowlist` and `session_tools_denylist`

These intersect — a tool must pass both the agent's allowlist/denylist AND the session's clamp to be available. An empty session allowlist means "no restriction" (unlike the agent allowlist where empty means "inherit all"). This is a subtle but important semantic difference.

### Pause/Resume with Worktree Checkpointing

`xai-grok-workspace/src/worktree.rs` and `xai-fast-worktree` implement git worktree-based isolation. Forked sessions (subagents) get their own worktree. The system prompt shows the original project path (via `prompt_working_directory`) while tool execution uses the worktree path — the model never sees the internal overlay path.

### Auto-Compaction Triggers

Compaction is triggered based on token usage exceeding a threshold percentage of the context window. Default threshold is 85% (from `DEFAULT_AUTO_COMPACT_THRESHOLD_PERCENT`). The check is: `total_tokens / context_window >= threshold_percent / 100`.

### Response API Integration

The crate uses `async-openai` with the "responses" feature for model interaction. Hosted tools are sent as `rs::Tool::WebSearch` and `rs::Tool::XSearch` — native Responses API types rather than function-call definitions.

## Design Decisions

### Monorepo Over Multi-Repo

All ~80 crates live in one workspace. This trades independent versioning for atomic cross-crate changes. The Cargo.toml is generated (treated as read-only) — per-crate Cargo.toml files are the canonical dependency declarations. This is unusual for a project this size and suggests the monorepo sync workflow is the primary development model.

### Closed Development, Open Source

External contributions are not accepted (`CONTRIBUTING.md` states this explicitly). The repository is a periodic snapshot from the internal monorepo. This is open-source as publication, not open-source as collaboration — the same model used by SQLite, Sentry, and Meta's Llama.

### TUI-First, Not API-First

The primary interface is the TUI. Headless mode, stdio, and ACP are secondary delivery modes. This is the opposite of most coding agents (Claude Code, Codex, Pi) which are CLI-first with optional TUI. The TUI investment shows in the 425K-line pager crate with its scrollback, modals, theming, and rendering subsystems.

### Rust All the Way Down

Unlike most coding agents (TypeScript for Claude Code, Python for OpenHands/DeerFlow, C# for OpenMonoAgent, TypeScript+Bun for Pi/omp), Grok Build is pure Rust from the TUI to the tool implementations. This gives it:
- No GC pauses during tool execution
- Single-binary distribution (with jemalloc)
- Zero-runtime-dependency deployment
- The cost of slower development iteration

### Dedicated Compaction Model

Rather than using the main conversation model for summarization, Grok Build can use a dedicated model (`compaction_model_name`). This is both an optimization (cheaper model for the bulk of compaction work) and a reliability measure (compaction doesn't compete with conversation for model availability).

### File-Based Configuration with Signed Policy

The config system supports Ed25519-signed requirements files, enabling enterprise-managed policy that survives tampering. This is unusual for a developer tool and suggests SpaceXAI's enterprise ambitions.

### Vendored Mermaid Stack

Rather than depending on the Node.js mermaid-cli (the standard approach), Grok Build vendors complete Rust implementations of the Mermaid parser, graph layout (dagre), and SVG renderer. This eliminates the Node.js dependency and makes diagram rendering fast and self-contained.

## Comparison Notes

Compared to other coding agents in the wiki:

- **vs. Claude Code**: Claude Code is closed-source (TypeScript), Grok Build is open-source (Rust). Both support subagents, skills, hooks, and ACP. Grok Build's TUI-first approach contrasts with Claude Code's CLI-first design. Both use similar compaction strategies (summarize-and-replace). Grok Build's tool protocol is more formally specified (JSON-RPC 2.0 with capability negotiation); Claude Code's is implicit.

- **vs. Oh My Pi (omp)**: omp (TypeScript + Rust N-API) and Grok Build (pure Rust) share some techniques — content-addressed editing (hashline vs. Write/Edit tools), dual memory architectures, and multi-provider support. But Grok Build is a product from a major AI lab with enterprise features (signed config, MDM, managed deployment); omp is a community project optimized for hackability.

- **vs. OpenMonoAgent**: Both use .NET-ecosystem tools. Grok Build is an order of magnitude larger and more feature-complete, but OpenMonoAgent's local-only focus (llama.cpp, zero API keys) is a fundamentally different deployment model.

- **vs. DeerFlow**: DeerFlow (Python, LangGraph, 175K lines) uses a middleware pipeline approach; Grok Build uses direct imperative Rust. DeerFlow's LangGraph middleware chain is more flexible for experimentation; Grok Build's compiled approach is more predictable in production.

- **vs. Zeroclaw**: Both are Rust agent runtimes. Zeroclaw is a framework for building agents; Grok Build is a complete product. Zeroclaw's trait-based design with 30+ channels has architectural similarities to Grok Build's Tool trait.

- **vs. Tau (τ)**: Tau is a Python educational coding agent with a three-layer architecture (brain/environment/face). Grok Build's architecture is similar at a high level (agent/shell/pager) but far more production-hardened.

## Tool System Deep Dive

### Tool Namespacing

Tools are identified by namespace-qualified IDs: `GrokBuild:read_file`, `GrokBuild:run_terminal_cmd`, `GrokBuildConcise:run_terminal_cmd`, `OpenCode:bash`, etc. Multiple implementations of the same logical tool can coexist under different namespaces.

### Tool Families

`ToolFamily` enables variant dispatch under a single `ToolId`. A family resolves a `ToolVariant` (Default or named) to a concrete `ArcTool`. Registries iterate variants at startup and cache results.

### Tool Capabilities

`ToolCapabilities` carries per-tool metadata: concurrency limits, scope (session/server), notification schemas, streaming capabilities. This feeds into the runtime's scheduling and the protocol's capability negotiation.

### MCP Integration

MCP tools are registered via `xai-grok-mcp`. The model accesses them through meta-tools: `search_tool` (find available MCP tools by description) and `use_tool` (invoke a specific MCP tool). MCP access is always preserved during tool allowlist filtering — compat allowlists like `[Read, Bash]` won't strip MCP meta-tools.

### Built-in Agent Types

Three built-in subagent types, defined in `xai-tool-types`:
- **general-purpose**: Full tool access — "Catch-all for any task"
- **explore**: Read-only search agent — read/grep/glob/list, no write/execute
- **plan**: Software architect — read/search/web_search, no write/execute

User-defined agents (from `.grok/agents/` or plugin registries) overlay or shadow these built-ins.

### Plan Mode

`enter_plan_mode` and `exit_plan_mode` tools are always present (even when not in the agent's tool list) because the TUI has a plan-mode keybind that needs them. This is a pragmatic coupling between the UI layer and tool availability.

## Security Architecture

### Permission Modes

Permissions are managed through the workspace crate's `permission` module with multiple modes: default, accept-edits, plan, yolo (bypass all). The `CapabilityMode` enum gates tool execution.

### Sandboxing

`xai-grok-sandbox` provides sandboxing support. `xai-grok-workspace/src/worktree.rs` implements git worktree isolation for subagent sessions.

### Folder Trust

`xai-grok-workspace/src/folder_trust.rs` and `trust.rs` implement trust-on-first-use for project directories. Similar to Claude Code's trust system.

### Credential Management

`xai-grok-secrets` manages API keys and credentials. `xai-grok-auth` handles OAuth-based browser authentication.

## Build System

- **Rust edition 2024**: Uses the latest Rust features
- **jemalloc**: Default allocator for release builds (via `tikv-jemallocator`)
- **Release profiles**: `release` (standard), `release-dist` (thin LTO, single CGU, hardened), `x-prod` (for latency-sensitive services), `release-dist-jemalloc` (alias for desktop)
- **Dev profile**: `panic = "abort"`, `codegen-units = 128`, `debug = "line-tables-only"` — optimized for fast iteration
- **proto codegen**: Uses `bin/protoc` via DotSlash for hermetic builds
- **Forbidden**: `unsafe_code` is forbidden in the tool protocol and runtime crates

## Observability

- **fastrace**: Distributed tracing (replaces tokio-tracing in the hot path)
- **OpenTelemetry**: OTLP export via gRPC and HTTP
- **Prometheus**: Process metrics, workspace metrics, permission metrics
- **tokio-console**: Tokio runtime inspection
- **pprof**: CPU profiling
- **Heap profiling**: Via jemalloc's profiling feature
- **Telemetry**: `xai-grok-telemetry` for unified logging with domain-specific targets (memory_log, etc.)
- **Crash handler**: `xai-crash-handler` for panic reporting
- **Mixpanel**: Product analytics
