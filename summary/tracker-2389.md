---
url: https://github.com/2389-research/tracker
title: "Tracker: Pipeline orchestration engine for multi-agent LLM workflows"
author: 2389 Research
date_fetched: 2026-05-31
date_published: 2026-04-01 (earliest tracked release)
topics:
  - agent-orchestration
---

# Tracker — Full Architectural Analysis

## Project Overview

Tracker is a Go-based pipeline orchestration engine for multi-agent LLM workflows, built by [2389 Research](https://2389.ai). It defines workflows in `.dip` files (Dippin language), executes them with parallel agents, and provides a TUI dashboard for monitoring. The project is ~67,000 lines of Go across three layers: LLM client (~2K lines in `llm/`), agent session (~4K lines in `agent/`), and pipeline engine (~15K lines in `pipeline/` + handlers). Tests are ~32K lines.

## Architecture Deep Dive

### Three-Layer Stack

```
Layer 3: Pipeline Engine — Graph execution, edge routing, checkpoints, TUI
Layer 2: Agent Session — Turn loop, tool execution, context compaction, event streaming
Layer 1: LLM Client — Provider adapters (Anthropic, OpenAI, Gemini), token tracking
```

This three-layer architecture was independently rediscovered by multiple teams (StrongDM, Dan Shapiro's Trycycle, 2389 Research) — suggesting it solves a fundamental problem in agent orchestration.

### Layer 3: Pipeline Engine (`pipeline/`)

The engine is a **graph traversal loop** with handler dispatch, edge selection, retry management, and checkpoint save/resume.

**Core loop** (`pipeline/engine.go:202-257`):
```go
for {
    lr := e.processNode(ctx, s, currentNodeID, resumeVisited)
    switch lr.action {
    case loopReturn: return lr.result, lr.err
    case loopBreak: goto done
    case loopContinue: currentNodeID = lr.nextNodeID; continue
    }
}
```

Each iteration dispatches a handler for the current node, collects the `Outcome`, applies context updates, scopes dirty keys into per-node namespaces, drains steering channel updates, and then selects the next edge based on conditions.

**Edge selection** (`pipeline/engine_edges.go`): Evaluates conditions using a custom expression evaluator (`pipeline/condition.go`). Supports `=`, `!=`, `contains`, `not contains`, `startswith`, `endswith`, `in`, `not in`, `&&`, `||`, `not`. The evaluator tokenizes on `strings.Split("||")` and `strings.Split("&&")` — a deliberate simplification that means parentheses are NOT supported (the dippin adapter rejects parenthesized expressions at parse time with `ErrParenthesizedParsedCondition`).

**Strict failure mode**: If a node returns `OutcomeFail` and has only unconditional outgoing edges, the engine halts the pipeline. This prevents the "silent continuation on failure" bug that plagued earlier agent orchestration systems.

**Budget guard**: `BudgetGuard` checks aggregate token count, cost, and wall time after each node. Child engines (subgraphs, manager_loops) inherit the parent's guard + baseline usage via `ChildRunContext` stashed in `ctx`. This closes the budget-bypass bug where `--max-tokens`/`--max-cost` had no effect on nested subgraphs (#183).

**Checkpoint system**: Serializes `{runID, currentNode, completedNodes, context snapshot, retry counts, edge selections}` to `checkpoint.json`. On resume, skips completed nodes and reconstructs edge selections from the checkpoint. Uses fidelity compaction to reduce context size on resume (`pipeline/engine_run.go:306-328`).

### Layer 2: Agent Session (`agent/`)

**Turn loop** (`agent/session.go:174-219`): A structured agentic loop — LLM call → tool execution → inject results → repeat. Key features:

- **Loop detection**: Computes a signature from tool call names and arguments. If the same signature repeats `LoopDetectionThreshold` times (default 4), the session aborts with `LoopDetected=true`.
- **Reflection on error**: Injects a structured reflection prompt (`agent/session.go:152-157`) after tool failures to help the LLM reason about what went wrong. Capped at 3 consecutive reflection turns.
- **Verify-after-edit**: After turns that include file writes, runs a verification command (auto-detected from project structure: `go test`, `cargo test`, `npm test`, `pytest`). On failure, injects a repair prompt and runs an extra LLM turn (outside the main `MaxTurns` budget). `MaxVerifyRetries` caps repair cycles.
- **Empty response handling**: Retries up to 2 times on zero-output-token, zero-tool-call responses before failing.
- **Turn-budget checkpoints**: Injects messages at configured fractions of `MaxTurns` (e.g., "60% of turns used — wrap up").

**Context compaction** (`agent/compaction.go`): Replaces old tool results with short summaries to prevent context window exhaustion. Tool-specific summarizers:
- `read_file`/`read`: reports line count, suggests re-reading
- `grep_search`/`grep`: reports match count
- `bash`/`execute_command`: extracts the command (first 80 chars) and scans the last 10 output lines for pass/fail signals
- Everything else: reports character count

Protected turns (default 5 most recent) are never compacted.

**Memory system** (`agent/memory.go`): Episodic — captures per-session tool attempts as `EpisodeEntry` records (tool name, args, success/fail, output summary). At session end, produces an `EpisodeSummary` string. Prior episode summaries are injected into retry sessions via `PriorEpisodeSummaries` so the agent avoids repeating known-failing approaches. Bounded by `maxEpisodeSummaryCount=8` and `maxEpisodeSummaryTotalRunes=4000`.

**Tool safety** (`pipeline/handlers/tool_safety.go`): Multi-layer defense:
1. Built-in denylist (glob patterns matching dangerous commands)
2. CLI allowlist + graph attr allowlist (union, deduplicated)
3. Per-node output limits (default 64KB per stream, global ceiling 10MB)
4. `tool_access: none` — zero-tool mode (defends against multi-tool-call vector)
5. Fail-closed for typos in `tool_access`

### Dippin Language Adapter (`pipeline/dippin_adapter.go`)

Converts the Dippin IR (from `github.com/2389-research/dippin-lang`) into Tracker's `Graph` model. The IR provides typed node configs (`ir.AgentConfig`, `ir.ToolConfig`, `ir.ManagerLoopConfig`, etc.) which the adapter flattens into `map[string]string` node attrs.

Node kind to shape/handler mapping (`pipeline/dippin_adapter.go:143-152`):
- `NodeAgent → box → codergen`
- `NodeHuman → hexagon → wait.human`
- `NodeTool → parallelogram → tool`
- `NodeParallel → component → parallel`
- `NodeFanIn → tripleoctagon → parallel.fan_in`
- `NodeSubgraph → tab → subgraph`
- `NodeConditional → diamond → conditional`
- `NodeManagerLoop → house → stack.manager_loop`

Edge conditions are serialized from parsed IR trees back to flat strings. Parenthesized conditions are rejected at adapter time because the engine evaluator doesn't support them.

### Handler System

Each node shape maps to a handler implementing `pipeline.Handler`:
```go
type Handler interface {
    Name() string
    Execute(ctx context.Context, node *Node, pctx *PipelineContext) (Outcome, error)
}
```

**Handler registry** (`pipeline/handlers/registry.go`): Factory pattern with functional options. `NewDefaultRegistry` wires up all handlers with dependency injection (LLM client for codergen, execution environment for tools, interviewer for human gates). `NewRegistryFactory` creates scoped registries for subgraph execution with namespaced event handlers.

**Key handlers**:
- **codergen** (`pipeline/handlers/codergen.go`): Creates an LLM session from node config, runs the agentic loop, handles `auto_status` parsing and `response_format` / `response_schema` for structured output
- **tool** (`pipeline/handlers/tool.go`): Executes shell commands with tail-buffer output capture, `marker_grep` regex extraction, `_TRACKER_ROUTE=` sentinel scanning
- **human** (`pipeline/handlers/human.go`): Five gate modes (choice, freeform, hybrid, yes_no, interview). ~920 lines of TUI code.
- **parallel** (`pipeline/handlers/parallel.go`): Fan-out to concurrent branches with isolated `PipelineContext` snapshots, wait for all to complete, merge results via fan_in
- **subgraph** (`pipeline/handlers/subgraph.go`): Recursive engine execution with context passthrough
- **manager_loop** (`pipeline/handlers/manager_loop.go`): Async child pipeline with poll loop, stop/steer condition evaluation, context injection

### PipelineContext (`pipeline/context.go`)

A thread-safe key-value store with two novel features:

1. **Dirty tracking**: Every `Set`/`Merge` marks keys as dirty. After each node completes, `ScopeToNode(nodeID)` copies dirty keys into `"node.<nodeID>.<key>"` entries, then clears the dirty set. This means downstream nodes can read a specific upstream node's output (`ctx.node.MyAgent.last_response`) without relying on the global last-writer-wins value.

2. **Internal namespace**: Separate from user-visible values, used for engine bookkeeping (retry counters, artifact dir). Not exposed to prompt expansion or edge conditions.

### Backend System

Three agent backends implementing the same interface:
- **native**: Direct API calls to Anthropic/OpenAI/Gemini via the `llm` package
- **claude-code**: Shells out to the Claude Code CLI, parsing NDJSON output
- **ACP**: Agent Client Protocol — connects to ACP-compliant servers over gRPC

Per-node `backend` attribute allows mixing backends within a single pipeline.

## Key Techniques

### Tail-buffer output capture (v0.27.0)

The tool handler uses a ring buffer (`agent/exec/tail_buffer.go`) that keeps the trailing N bytes of stdout/stderr rather than the head. This fixes the bug where routing markers (`printf 'tests-pass'`) emitted at the end of command output were silently dropped when output exceeded the 64KB cap and the head was kept. The tail buffer means routing markers survive truncation by construction.

### Dirty-set-based per-node scoping

Rather than prefixing every context write with a node ID (which would break backward compatibility with bare `${ctx.outcome}` references), the engine tracks which keys a node modified and copies them into scoped namespaces after execution. This is a clean separation between "what the node wrote" and "what the node's namespace contains."

### Tool output truncation as structured events

When stdout/stderr exceeds the per-stream cap, the engine emits `EventToolOutputTruncated` events with exact byte counts (captured, dropped, limit). `tracker diagnose` correlates these with routing misses to surface probable cause: "your routing marker may have been dropped from the output head."

### Fidelity degradation chain

Six fidelity levels form a total order (`full → summary:high → summary:medium → summary:low → compact → truncate`) with `DegradeFidelity` stepping down one level. Used during checkpoint resume to compact context when the pipeline restarts. The degradation is one-way (you can't go back to `full` without a fresh run).

### Sentinel-protected activity log integrity

Live activity writes go to `$XDG_STATE_HOME/tracker/runs/<id>/` with mode `0o600` and a `\x1f\x1e` sentinel prefix. `tracker diagnose` validates every line has this sentinel and surfaces `SuggestionAuditLogInjection` when it doesn't. At run-end, a sentinel-stripped snapshot is mirrored to the legacy path for external tooling. This is detection-only (not cryptographic authentication) but catches accidental corruption and casual tampering.

### Multi-source allowlist/denylist union

Tool command allowlists merge CLI flags (`--tool-allowlist`) with graph attributes (`tool_commands_allow`) via order-preserving dedup. Denylist additions merge CLI flags (`--tool-denylist-add`) with graph attrs (`tool_denylist_add`). The built-in denylist is always evaluated first regardless of these extensions — you can add blocks but never remove them (except via the nuclear `--bypass-denylist` flag).

## Design Decisions

### Optimized for: Correctness and auditability over raw speed

- Every edge routing decision is logged as a structured JSON event
- The engine fails loudly on unknown outcome statuses rather than silently continuing
- Conditional edges without explicit failure handling cause pipeline termination
- Parenthesized condition expressions are rejected at parse time rather than silently mis-evaluated

### Optimized for: Pipeline author experience over runtime flexibility

- The DAG is static (defined in `.dip` files) rather than dynamically constructed at runtime. This is a deliberate trade-off vs. Cord's dynamic task decomposition — Tracker's static graph can be validated, simulated, and linted before execution.
- Conditional routing provides branching but the graph topology doesn't change mid-run.
- `marker_grep` and `_TRACKER_ROUTE=` provide structured routing channels that fail loudly on missing markers rather than silently falling through unconditional edges.

### Optimized for: Multi-backend portability over deep integration

- The three backends (native, claude-code, ACP) share a common handler interface but each has different capabilities. Tool safety enforcement varies: native fully enforces; claude-code does best-effort via `--disallowedTools`; ACP refuses session creation for `tool_access: none`.
- Cost estimation differs: native uses actual token counts from API responses; ACP estimates from rune counts; claude-code parses NDJSON envelope fields.

### Sacrificed: Dynamic task decomposition

- Unlike Cord (which lets agents spawn/fork subtasks at runtime), Tracker requires the workflow author to define the full DAG upfront. This is better for validation and reproducibility but worse for tasks where the decomposition isn't known in advance.
- The `manager_loop` handler partially addresses this by allowing iterative supervision, but the child pipeline structure is still static.

### Sacrificed: Distributed execution

- Tracker runs on a single machine. Parallel branches execute concurrently but share the same process. There's no built-in distributed execution model (compare to Loomkin's Erlang/OTP-based actor system).
- Worktree isolation provides filesystem separation for parallel agents but not process-level isolation.

## Core Abstractions

1. **Graph + Node + Edge** (`pipeline/graph.go`): The universal data model. Everything the engine processes is a graph. The Dippin IR, DOT files, and programmatic construction all converge on this model.

2. **PipelineContext** (`pipeline/context.go`): The shared state bus. Every handler reads from and writes to this context. The dirty-set mechanism enables per-node scoping without breaking backward compatibility. Internal namespace separates engine state from prompt-visible state.

3. **Outcome** (`pipeline/handler.go`): The handler return type. Four fields (Status, ContextUpdates, PreferredLabel, SuggestedNextNodes) control the engine's next action. Extensions (ChildUsage, Truncations, MissingMarker, MissingRoute) carry structured diagnostic data that the engine emits as typed events.

## Codebase Stats

- ~67,000 total lines of Go (production + test)
- ~32,000 lines of tests
- `pipeline/` package: ~15,000 lines (engine, adapter, handlers, context, fidelity, etc.)
- `agent/` package: ~4,000 lines (session, config, compaction, memory, localization)
- `cmd/tracker/`: ~4,000 lines (CLI commands, TUI, flags, configuration)
- Single dependency on `dippin-lang` (same org) for parsing `.dip` files
- Go 1.25.5
