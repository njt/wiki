# Grok Build

SpaceXAI's terminal-based AI coding agent — a pure-Rust, ~80-crate monorepo (~1M+ lines) implementing a full TUI coding harness with subagent spawning, persistent memory, context compaction, hooks/plugins, MCP integration, and the Agent Client Protocol. Open-source as publication (no external contributions), synced periodically from the internal SpaceXAI monorepo. The largest production Rust coding agent in the open.

---

## Architecture

Grok Build is a **layered monorepo** in three tiers:

**Foundation** (`crates/common/`): A formal JSON-RPC 2.0 tool protocol with capability negotiation (`xai-tool-protocol`), a unified streaming `Tool` trait with type erasure (`xai-tool-runtime`), a transport-agnostic compaction engine with three strategies (`xai-grok-compaction`), a mid-turn interjection buffer (`xai-interjection-core`), and a circuit breaker for tool calls (`xai-circuit-breaker`).

**Product** (`crates/codegen/`): 50+ crates forming the application. The key subsystems:

- **Agent assembly** (`xai-grok-agent`, 21K lines): `AgentBuilder` fluent API with a 10-step build process — skill discovery, tool allowlist/denylist resolution with compat name mapping (Claude's `Read/Bash/Grep/Edit` → Grok `ToolKind`), subagent type classification via regex, and session-level tool clamping (intersection of agent and session allowlists). The `Agent` type is immutable after construction — mutations go through `ToolBridge`'s internal locks.

- **Shell runtime** (`xai-grok-shell`, 338K lines): Session management, model interaction, subagent spawning, ACP sessions, terminal integration, Claude import, plugin loading, and three entry points (leader/stdio/headless). The largest single crate.

- **TUI** (`xai-grok-pager`, 425K lines): ratatui-based full-screen terminal UI with scrollback, prompt, modals, rendering. The `xai-grok-pager-bin` crate is the composition root producing the `xai-grok-pager` binary (shipped as `grok`).

- **Tools** (`xai-grok-tools`, 112K lines): All tool implementations plus the `ToolBridge` central registry. Tools: `read_file`, `write`/`edit`, `run_terminal_cmd` (bash), `grep`, `glob`, `list_dir`, `web_search`, `web_fetch`, `task` (subagent spawning), `enter_plan_mode`/`exit_plan_mode`, `ask_user_question`, `memory_search`/`memory_get`, `image_gen`/`image_edit`, `video_gen`, LSP, deploy. Also includes compat tool implementations ported from openai/codex and sst/opencode.

- **Workspace** (`xai-grok-workspace`, 78K lines): Filesystem, VCS, execution, permissions, folder trust, file watching, session management, recovery, daemonization, worktree isolation, multi-session hub coordination.

- **Chat state** (`xai-chat-state`, 13K lines): Actor-based conversation state — `ChatStateActor` in a dedicated tokio task, commands via channel, events on state changes, persistence via `ChatPersistence` trait. No locks needed on the conversation vector.

- **Memory** (`xai-grok-memory`, 9.9K lines): Markdown-based persistent memory at `~/.grok/memory/` with per-workspace scoping (keyed by blake3 hash). SQLite vector index with sqlite-vec, MMR retrieval, query expansion, chunking, embedding batching, and a dream/consolidation pipeline that extracts durable facts from session logs.

- **Code intelligence** (`xai-codebase-graph`, 9.7K lines): Tree-sitter code graph with parallel indexing via rayon, memory-mapped cache files, and channel-based incremental updates via `IndexManager`. Go-to-definition and go-to-references queries.

- **Other subsystems**: MCP integration (`xai-grok-mcp`, 10.5K lines), hooks (`xai-grok-hooks`, 8.5K lines — file-based discovery, JSON-defined, child-process execution, four event types), config (`xai-grok-config`, 6.9K lines — six-layer merge with Ed25519-signed policy), ACP library (`xai-acp-lib`, 2.3K lines), sandboxing, auth, telemetry, updates, voice, plugin marketplace.

**Vendored** (`third_party/`): Complete Rust implementations of the Mermaid diagram stack — parser, graph layout (dagre), and SVG renderer — plus graph algorithm libraries. Eliminates the Node.js dependency that every other Mermaid integration carries.

## Key Techniques

### The Tool Streaming Contract

Every tool implements the `Tool` trait, whose `execute` method returns a `ToolStream<T>` with a structural invariant: zero or more `Progress` items (text chunks, content blocks, or custom payloads) followed by exactly one `Terminal` result. Two constructors — `terminal_only(result)` for blocking tools and `with_progress(stream, future)` for streaming tools — guarantee this shape at the type level. The `ToolDyn` blanket impl handles JSON deserialization of args and re-serialization of output, so tool authors never touch JSON manually.

### Compat Tool-Name Mapping

Agent definitions written for Claude Code (e.g., `tools: Read, Bash, Grep, Edit, Task, Skill, LSP`) resolve to Grok's `ToolKind` enum via a centralized mapping in `xai-grok-tools/src/tool_taxonomy.rs`. Unrecognized entries cause the builder to fall back to the full toolset rather than silently stripping tools — a deliberate safety-over-restriction choice. MCP tools (`mcp__*` prefix) are always preserved. Subagent type directives use regex pattern `Agent(type1, type2)` to constrain which subagent types can be spawned.

### Template Variable System

Tool descriptions use `${{ template.variables }}` syntax resolved at finalize time by `TemplateRenderer`. This decouples description authoring from concrete naming: `${{ tools.by_kind.task }}` resolves to the task tool's function name regardless of namespace, and `${{ params.task.subagent_type }}` adapts to the parameter name the model actually sees. The task tool description is dynamically built from discovered subagents.

### Dual Tool Architecture

Web search and X search can be sent as either **hosted tools** (native Responses API types executed server-side by the agentic sampler, skipping the tool-call round-trip) or **function tools** (local implementations executed client-side). The `backend_search` toggle is ANDed with per-model capability flags at request time, so the same agent definition works across models with different server-side search support.

### Three Compaction Strategies, One Engine

The `xai-grok-compaction` crate defines three styles sharing a common trait seam (`CompactionItem`, `CompactionSampler`, `ItemTokenCounter`):
- **Full-replace** (code_compaction): The entire conversation is summarized by a dedicated compaction model. Used by Grok Build. Includes degeneracy detection and failure classification.
- **Tail-keep** (intra_compaction): Recent turns preserved, older ones summarized. Used by Grok Chat's per-step pass.
- **Between-turn** (inter_compaction): Chunked summarization between conversation turns.

### Interjection Buffer

When a user sends a message while the agent is mid-turn, it goes into an `InterjectionBuffer` (`EventQueue<PendingInterjection>`). At the next safe drain point, `drain_formatted()` wraps each entry as a synthetic user message with the framing "The user sent a message while you were working." Entries are FIFO, one per message, never merged. This is a small crate (320 lines) but solves a real problem that most agent frameworks handle ad hoc.

### Actor-Based Conversation State

`ChatStateActor` owns the full conversation in a dedicated tokio task. All access goes through `ChatStateHandle` (command + oneshot). This eliminates locks on the conversation vector — only one task ever touches it. Events are emitted on state changes for observers (TUI, logging, telemetry). The pattern is the same one used by `xai-hunk-tracker`.

## Design Decisions

**Rust all the way down.** Unlike every other major coding agent — Claude Code (TypeScript), Codex (TypeScript), Pi/omp (TypeScript+Bun+Rust N-API), OpenHands/DeerFlow (Python), OpenMonoAgent (C#) — Grok Build is pure Rust from TUI to tool implementations. This trades slower development iteration for: no GC pauses during tool execution, single-binary distribution with jemalloc, and zero runtime dependencies.

**TUI-first, not CLI-first.** The primary interface is the 425K-line pager, with headless mode, stdio, and ACP as secondary delivery modes. This is the inverse of most coding agents, which are CLI-first with optional TUI. The TUI investment includes scrollback, modals, theming, and rendering subsystems.

**Open source as publication, not collaboration.** External contributions are not accepted. The repo is a periodic snapshot from the internal monorepo, recorded by `SOURCE_REV`. This is the same model as SQLite, Sentry, and Meta's Llama — the code is public but the development process is not.

**Generated workspace root.** The root `Cargo.toml` (workspace members, dependency versions, lints, profiles) is generated and treated as read-only. Per-crate `Cargo.toml` files are the canonical dependency declarations. This is unusual at this scale and reflects the monorepo sync workflow.

**Dedicated compaction model.** Rather than using the main conversation model for summarization, Grok Build can route compaction to a separate model. This avoids the failure mode where a degraded conversation model also degrades its own compaction.

**Enterprise policy support.** The config system supports Ed25519-signed requirements files and macOS MDM managed preferences — unusual features for a developer CLI tool that indicate SpaceXAI's enterprise ambitions.

**Vendored Mermaid stack.** Instead of shelling out to Node.js mermaid-cli (the standard approach), Grok Build contains complete Rust implementations of the Mermaid parser, dagre layout engine, and SVG renderer. Zero Node.js dependency.

**Fail-open tool allowlists.** If a `tools:` allowlist contains entries that can't be mapped to any known tool, Grok Build falls back to the full toolset rather than leaving the agent crippled. MCP access is always preserved. The philosophy: an overly restrictive allowlist is more dangerous than an overly permissive one.

## Comparison Notes

Grok Build is the **most architecturally ambitious** open-source coding agent. Compared to peers:

- **vs. Claude Code** (closed-source, TypeScript): Both support subagents, skills, hooks, MCP, and ACP. Grok Build's tool protocol is formally specified (JSON-RPC 2.0 with capability negotiation); Claude Code's is implicit. Grok Build is TUI-first; Claude Code is CLI-first. Both use summarize-and-replace compaction. Claude Code's CLAUDE.md + skills + hooks ecosystem is more mature; Grok Build's `AgentDefinition` markdown format is more structured.

- **vs. Oh My Pi (omp)** (community, TypeScript+Rust): Both share content-addressed editing, dual memory architectures, and multi-provider support. omp is optimized for hackability (MIT, accepts contributions); Grok Build is optimized for production deployment (enterprise policy, MDM, signed config, managed rollout).

- **vs. Codex CLI** (closed-source, TypeScript): Codex is CLI-first with the OpenAI ecosystem; Grok Build is TUI-first with the xAI ecosystem. Codex's skills/hooks model influenced the broader coding-agent space; Grok Build's tool protocol and compaction engine are more formally architected.

- **vs. Zeroclaw** (community, Rust): Both use Rust's trait system for tool definition. Zeroclaw is a framework for building agents; Grok Build is a complete product. Zeroclaw's 30+ channel architecture has conceptual similarities to Grok Build's `ToolBridge` + `ToolDispatch` pattern.

- **vs. DeerFlow** (ByteDance, Python): DeerFlow's LangGraph middleware pipeline favors composability; Grok Build's direct imperative Rust favors predictability. 175K Python lines vs. 1M+ Rust lines — different scales, different philosophies.

- **vs. OpenMonoAgent** (community, C#): Both provide terminal-native experiences. OpenMonoAgent targets local-only, zero-API-key deployment; Grok Build targets the xAI cloud ecosystem with local fallback. Grok Build is roughly 10× the codebase size.

- **vs. [[Magnitude]]** (magnitudedev, TypeScript+Rust): Both pair a Rust native inference layer with a serious agent harness. Magnitude is local-first — embedded llama.cpp (no Ollama), hardware-profiled model recommendation, speculative decoding — and event-sourced (projections over a durable event log) rather than actor-based, splitting TypeScript/Effect for the agent from Rust only for inference. Grok Build is one pure-Rust monolith targeting the xAI cloud; Magnitude owns the whole stack on the user's machine.

- **vs. [[Shelley]]** (Bold Software, Go): Grok Build is TUI-first and pure Rust; Shelley is web-first and Go. Both compile to a single binary and both treat the harness as the product, but Shelley bets on a phone-reachable SSE UI and a single pi-ported compaction strategy, while Grok Build ships a three-strategy compaction engine and a formal JSON-RPC tool protocol.

What distinguishes Grok Build architecturally is the **formal protocol layer** (tool registration, capability negotiation, JSON-RPC envelope) and the **three-strategy compaction engine** shared across product lines — both suggest an architecture designed for a platform, not a single product.

---
*Sources: [[raw/grok-build]]*
*Last updated: 2026-07-18*
*Tags: #tool #project #agents #coding-agent #rust #TUI #terminal #MCP*
