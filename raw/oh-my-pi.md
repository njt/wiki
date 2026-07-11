---
url: https://github.com/can1357/oh-my-pi
title: oh-my-pi (omp) — A Coding Agent with the IDE Wired In
author: Can Bölük (fork of Mario Zechner's pi-mono)
date_fetched: 2026-07-11
date_published: 2025
---

# Oh My Pi (omp)

Fork of [Pi](https://github.com/badlogic/pi-mono) by Mario Zechner, rewritten as a coding-first surface. Open-source MIT-licensed terminal-based coding agent harness. TypeScript (Bun runtime) for agent logic, Rust (N-API addon) for performance-critical operations. ~345K lines of TypeScript, ~55K lines of Rust.

Homepage: https://omp.sh
npm: `@oh-my-pi/pi-coding-agent`
Version: 16.4.4

## Architecture Overview

### Monorepo Structure

```
oh-my-pi/
  packages/
    coding-agent/   — main CLI + SDK (345K LoC)
    agent/          — agent runtime, compaction, tool execution (13K LoC)
    ai/             — multi-provider LLM client (88K LoC)
    tui/            — terminal UI with differential rendering (23K LoC)
    mnemopi/        — local SQLite+vector memory engine (19K LoC)
    hashline/       — content-hash-anchored patch language (5.7K LoC)
    catalog/        — model catalog, bundled models.json, provider descriptors
    natives/        — N-API bindings to Rust layer
    utils/          — shared utilities
    stats/          — local observability dashboard
    wire/           — shared collab protocol types
    collab-web/     — browser guest client for live sessions
    snapcompact/    — bitmap-frame context compression
    swarm-extension/— swarm orchestration extension
  crates/
    pi-natives/     — core Rust N-API addon (17K LoC)
    pi-shell/       — embedded shell/PTY (brush-shell, vendored)
    pi-ast/         — tree-sitter code summarizer and AST utilities
    pi-iso/         — task isolation backend (APFS, btrfs, zfs, overlayfs, projfs)
    pi-walker/      — workspace walker
    vendor/brush-*  — vendored bash interpreter
```

### Two-Layer Architecture

**TypeScript layer** (Bun runtime): Agent orchestration, session management, TUI rendering, provider negotiation, tool dispatch, slash command handling, extension loading, memory pipeline, compaction strategies.

**Rust native layer** (N-API `cdylib`): All performance-critical operations run in-process — no fork/exec on the hot path. grep (ripgrep), shell (brush/bash), AST (ast-grep + 50+ tree-sitter grammars), text processing, syntax highlighting, PTY, clipboard, image decode/SIXEL encoding, BPE token counting, workspace scanning, process-tree management.

The native addon ships per-platform: `linux-x64`, `linux-arm64`, `darwin-x64`, `darwin-arm64`, `win32-x64`. x64 gets AVX2-variant detection at load time.

### Session Model

Append-only JSONL files with tree-shaped branching. Each line is a `SessionEntry` union type. `parentId`/`leafId` pointers support `/tree` navigation — branching doesn't mutate history. Header carries version, cwd, model info, and currency metrics. Blobs are content-addressed (SHA-256) in a separate store. Sessions live at `~/.omp/agent/sessions/<dir-encoded>/<timestamp>_<sessionId>.jsonl`.

### Compaction System

Two strategies for keeping sessions usable without losing context:

1. **Context-full summarization** — LLM summarizes old history into a `CompactionEntry`; only entries from `firstKeptEntryId` forward + the summary are sent to the model.

2. **Snapcompact** — Dense bitmap-frame representation of prior history. Archive strategy for when you need the record but not the tokens.

Automatic triggers: context overflow recovery, incomplete-output recovery, post-turn threshold maintenance, mid-turn threshold maintenance, and idle maintenance. Each recovery path has different rules about whether handoff strategy is allowed.

### Entry Points

Four modes, one engine:
- **Interactive TUI** (`omp`) — Default. Differential rendering, tool cards, ask picker, keyboard shortcuts.
- **One-shot** (`omp -p`) — Single prompt, JSON/text output, exits.
- **RPC** (`omp --mode rpc`) — NDJSON over stdio for embedders. `--mode rpc-ui` adds tool cards/dialogs.
- **ACP** (`omp acp`) — Agent Client Protocol over JSON-RPC. Editor-integrated: reads/writes through editor save paths, destructive tools gated by `session/request_permission`.

Plus a Node SDK (`@oh-my-pi/pi-coding-agent`) exposing `ModelRegistry`, `SessionManager`, `createAgentSession`, `discoverAuthStorage`.

## Key Techniques

### 1. Hashline: Content-Hash-Anchored Editing

Instead of line numbers or string matching, hashline patches reference lines by content hash. The model writes `[file.ts#abc123]` then `SWAP 42:56` or `DEL 87:92` with content bodies. Stale anchors cause rejection before corruption. This eliminates the whitespace battles and string-not-found loops that plague str_replace-based editing.

Implementation: `packages/hashline/` — tokenizer → parser (state machine) → patcher (applier). The parser validates range ordering, detects apply_patch contamination, and rejects bare literal values that look like YAML bodies. Recovery path: if an anchor doesn't match, the applier tries fuzzy matching before rejecting.

### 2. Time-Traveling Stream Rules (TTSR)

Regex matches on the model's output stream abort mid-token, inject a rule as a system reminder, and retry from the same point. Rules sit dormant until the model goes off-script — no context tax per turn. Injections survive compaction. Implementation lives in `packages/coding-agent/` with the `ttsr-injection-lifecycle.md` doc.

### 3. Dual Memory Architecture

Two independent memory systems:

**Hindsight (markdown pipeline)**: Phase 1 — per-session extraction (LLM reads session history, produces raw memory block + synopsis). Phase 2 — consolidation (second LLM pass synthesizes across sessions, produces `MEMORY.md`, `memory_summary.md`, and `skills/` playbooks). Summary injected at session start as system prompt guidance.

**Mnemopi (SQLite + vector)**: Local SQLite database with vector embeddings (`BAAI/bge-base-en-v1.5` 768d or `multilingual-e5-large` 1024d). Beam store with Weibull MMR scoring for recall. Polyphonic recall (4-voice: vector, graph, fact, temporal) with reciprocal rank fusion. Scoping: global, per-project, or per-project-tagged. Episodic graph, entity extraction, typed memory, consolidator. LLM-dependent paths use configured pi-ai models, with a no-LLM fallback.

### 4. In-Process Native Operations

Rust crates compiled as N-API addon, loaded into the Bun process. Key modules:
- **grep** (`crates/pi-natives/src/grep.rs`): ripgrep-backed. PCRE2 and Rust regex engines. Parallel file search with rayon. Context lines, count, files-with-matches modes. 4MB file cap, 128KB small-file read threshold.
- **shell** (`crates/pi-shell/`): Vendored brush-shell for embedded bash execution with persistent sessions. Custom builtins, timeout/abort, output minimizer (reduces verbose command output before it reaches the LLM).
- **summary** (`crates/pi-natives/src/summary.rs`): tree-sitter structural source summaries with elision controls across 50+ language grammars.
- **keys** (`crates/pi-natives/src/keys.rs`): Kitty keyboard protocol with xterm fallback, PHF perfect-hash lookup for ~1500 lines of key parsing.

### 5. Role-Based Model Routing

Five named roles: `default`, `smol` (cheap subagent fan-out), `slow` (deep reasoning), `plan` (plan mode), `commit` (changelogs). Each role can be bound to a different provider/model. Per-role fallback chains under `retry.fallbackChains` — when primary throws 429s, next entry takes over with session-affine round-robin credential rotation and per-credential backoff.

### 6. Provider Abstraction with Dialect Layer

`packages/ai/src/dialect/` — per-provider adapter classes implementing the `Dialect` interface. Handles tool schema coercion, thinking block demotion, XML rendering vs. native structured output, harmony leak detection (GPT-5), fenced thinking (DeepSeek), inband tool history encoding. 40+ providers registered in `packages/ai/src/registry/` with OAuth flow support (device-code, PKCE, callback-server).

### 7. Compaction Entry Model and Context Reconstruction

Compaction produces `CompactionEntry` (summary, shortSummary, firstKeptEntryId, tokensBefore). Context rebuild: latest compaction → one `compactionSummary` message; kept entries from `firstKeptEntryId` forward → regular messages; later entries appended. Branch summaries (`BranchSummaryEntry`) capture abandoned branch context during `/tree` navigation.

### 8. Subagent Orchestration with Type Safety

`task` tool fans out subagents in parallel with optional worktree isolation. Each subagent runs its own tool surface. Results are schema-validated typed objects — no prose to parse. The `irc` tool provides short prose communication between live agents in the same process. Parent reads `agent://<id>/findings.0.path` to pull structured fields out of subagent output.

### 9. Internal URL Schemes

Twelve internal schemes resolve transparently inside every FS-shaped tool: `pr://`, `issue://`, `agent://`, `skill://`, `rule://`, `memory://`, `conflict://`, and others. `read pr://1428` returns structured diff data. `search` walks a diff like a directory. This collapses API-specific tool calls into the single `read`/`search` surface the model already knows.

### 10. Advisor/Watchdog Pattern

A second model reads every turn the main agent takes, injecting notes inline: a quiet aside, a concern, or a hard blocker. Runs on its own context and model, so it catches what the doer rushed past. The main agent sees the note and course-corrects, or tells the user why it won't. This is architecturally the inverse of the Thrifty pattern (where cheap does, expensive reviews): here the expensive model typically drives and a second model advises.

## Design Decisions

### Optimized for: Terminal-first real coding work
The TUI is the default surface. Tool calls render as cards, edits preview before landing, ambiguity routes through structured option pickers. This design choice means the agent feels like a tool, not a chat app — it stays out of the way until you need it.

### Optimized for: Harness correctness over model intelligence
The core thesis: "the harness matters more than the model." Hashline eliminates edit-format failures. In-process native tools eliminate fork/exec errors and missing-binary failures. TTSR catches model mistakes mid-stream without paying context tax. The README's benchmark table quantifies this: Grok Code Fast 1 goes from 6.7% to 68.3% pass rate just by fixing the edit format.

### Trade-off: Monorepo complexity vs. tight integration
14 npm packages + 6 Rust crates in a single repo. This makes the build system complex (bun workspaces + cargo workspace + N-API build) but enables the tight coupling that makes features like in-process ripgrep, internal URL schemes, and cross-package type sharing possible.

### Trade-off: Rust native layer vs. portability
The ~55K lines of Rust are a significant maintenance burden and require platform-specific builds. But they eliminate the "missing binary" problem (rg, bash, find not installed) and the fork/exec round-trip cost. On Windows, this is especially valuable — no WSL bridge needed.

### Trade-off: Configuration inheritance vs. format lock-in
omp reads 8 competitor config formats natively (Cursor MDC, Cline .clinerules, Codex AGENTS.md, Copilot applyTo, etc.). This is generous for adoption but creates a maintenance burden as each upstream format evolves.

### Trade-off: Batteries-included vs. minimalism
Pi's original philosophy was minimalism. omp adds 32 built-in tools, 40+ providers, memory, subagents, browser automation, collaboration, ACP, and more. This makes it the "most capable agent surface that ships" but also the most complex — the tension noted in the existing [[Pi Coding Agent]] wiki page.

### Weakness: Tool sprawl
32 built-in tools is a lot for any model to navigate. The `search_tool_bm25` tool-discovery mechanism mitigates this by hiding tools behind a BM25 index until needed, but tool selection is still a nontrivial part of the agent's job.

### Weakness: Compaction complexity
The compaction system has six trigger paths with different rules about handoff strategy, context promotion, and retry behavior. This is correct but hard to reason about. The `compaction.md` doc is thorough but runs 200+ lines to explain the various paths.

## Comparison Notes

**Vs. Claude Code (Anthropic)**: Claude Code uses str_replace for editing; omp uses hashline (content-hash anchors). Claude Code shells out to bash/rg/git; omp links them in-process. Both have subagents, but omp's return typed objects while Claude Code's return prose. Claude Code has native skill support; omp inherits skills from 8 competitor formats plus its own.

**Vs. Codex (OpenAI)**: Codex uses apply_patch with a different patch format. omp has hashline and explicit rejection of apply_patch contamination. Both have sandboxing, but omp's isolation is filesystem-level (pi-iso crate, APFS clones/reflinks) rather than container-level.

**Vs. Pi (original)**: Pi was a minimal agent surface. omp adds sessions, subagents, slash commands, extensions, memory, compaction, browser, collaboration, ACP, and 30+ more tools. The philosophical tension between minimalism and platform ambitions is captured in [[Pi Coding Agent]].

**Vs. MiMo Code**: Both target long-horizon tasks with independent writer subagents and checkpointing, but MiMo uses code-based orchestration (Dynamic Workflow) while omp uses prompt-based orchestration with typed tool results.

**Vs. Aider**: Aider uses a unified diff format; omp uses hashline anchors + ast_edit structural rewrites. Aider's edit format has well-known failure modes (whitespace, context matching) that hashline was designed to eliminate.

**Vs. OpenMonoAgent / CodeAlta**: Both are terminal coding agents in C#/.NET with local LLM support. omp is TypeScript+Rust with a much larger provider surface (40+ vs. <10) and more built-in tools (32 vs. ~20).

## Repository Stats

- Total files: 5,229
- TypeScript: ~345K lines (coding-agent), 88K (ai), 23K (tui), 19K (mnemopi), 13K (agent)
- Rust: ~17K lines (pi-natives core), plus pi-shell, pi-ast, pi-iso
- 100+ documentation files in `docs/`
- 32 built-in tools, 40+ providers, 14 LSP ops, 28 DAP ops
- 5 platform targets for native addon
- 70+ tree-sitter grammars bundled for AST/summary
