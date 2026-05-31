# Tracker

Pipeline orchestration engine for multi-agent LLM workflows from 2389 Research. Define pipelines in `.dip` files (Dippin language), execute with parallel agents across three backends (native API, Claude Code CLI, ACP), and watch progress in a TUI dashboard. The reference implementation of the three-layer architecture described in [[The Dark Factory is a DOT File]] — and the most battle-tested open-source pipeline engine for coding agents, with ~67K lines of Go in production use on real CI pipelines and SWE-bench evaluations.

---

## Architecture

Tracker is a three-layer monolith:

```
Pipeline Engine → Agent Session → LLM Client
     (graph)         (turn loop)    (providers)
```

**Pipeline engine** (`pipeline/engine.go`): Graph traversal loop — dispatch handler for current node, collect `Outcome`, apply context updates, select next edge via condition evaluation, manage retries and checkpoints. The loop supports three actions: `loopContinue` (advance), `loopBreak` (success), `loopReturn` (error/budget-exhausted). Strict failure mode: a failed node with only unconditional edges halts the pipeline.

**Agent session** (`agent/session.go`): Turn-based LLM loop. Signature-based loop detection (4 repeats → abort). Reflection prompts on tool errors (capped at 3 consecutive). Verify-after-edit loop (auto-hunt for test command, inject repair prompt on failure, bounded retries). Empty response retry. Turn-budget checkpoints (inject "wrap up" messages at configured fractions of MaxTurns).

**Handler registry** (`pipeline/handlers/registry.go`): 10 handlers (codergen, tool, human, parallel, fan_in, conditional, subgraph, manager_loop, start, exit). Factory pattern with functional options — test stubs can replace any handler. Scoped registries created for subgraph execution with namespaced event handlers.

**PipelineContext** (`pipeline/context.go`): Thread-safe KV store with dirty tracking. After each node, dirty keys are copied into `node.<id>.<key>` namespaces — downstream nodes can reference specific upstream outputs without relying on last-writer-wins globals. Internal namespace for engine bookkeeping (retry counters).

**Dippin adapter** (`pipeline/dippin_adapter.go`): Converts Dippin IR to Tracker's Graph. Eight node kinds map to DOT shapes and handlers. Parenthesized edge conditions rejected at adapter time — the engine evaluator tokenizes on `strings.Split("||")` and `strings.Split("&&")`, so `(a || b) && c` would produce garbage tokens.

## Key Techniques

**Tail-buffer output capture** (`agent/exec/tail_buffer.go`): Tool stdout/stderr uses a ring buffer keeping the trailing 64KB, not the head. Routing markers (`printf 'tests-pass'`) survive truncation by construction — the tail is where markers live.

**Sentinel-protected activity log** (`pipeline/events_jsonl.go`): Live writes go to `$XDG_STATE_HOME` with `\x1f\x1e` sentinel prefix and `0o600` permissions. `tracker diagnose` detects injection. Detection-only (not cryptographic), but beats plain grep-able JSONL.

**Fidelity degradation chain**: `full → summary:high → summary:medium → summary:low → compact → truncate`. Six levels, one-way degradation. Applied during checkpoint resume to compact context. The `summary:medium` level keeps only outcome, last_response, human_response, and routing hints.

**Context compaction** (`agent/compaction.go`): Tool-specific summarizers replace old results. `read_file` reports line counts; `bash` extracts the command and scans last 10 lines for pass/fail; `grep` reports match count. Five most recent turns are never compacted.

**Budget guard propagation** (`pipeline/engine_run.go:451-461`): Child engines (subgraph, manager_loop) inherit parent's `BudgetGuard` + baseline usage via `ChildRunContext` on ctx. Prevents the "subgraph sandbox" where `--max-tokens` is silently non-binding for nested nodes (#183).

**Episodic memory** (`agent/memory.go`): Per-session tool attempt log. Summaries injected into retry sessions so the agent avoids known-failing approaches. Bounded: 8 episodes max, 4000 runes total.

**Multi-source tool safety**: CLI allowlist + graph attr allowlist (union, deduplicated). Built-in denylist always evaluated first. `tool_access: none` gives zero-tool mode (fail-closed for typos). Three levels of enforcement varying by backend.

## Design Decisions

**Static DAG over dynamic decomposition**: Unlike [[Cord]] (which lets agents spawn/fork subtasks at runtime), Tracker requires the full graph defined upfront. This enables validation, simulation, and linting before execution — but sacrifices the ability to discover task structure at runtime. The `manager_loop` handler partially addresses this with iterative supervision of a static child pipeline.

**Correctness over speed**: Unknown outcome statuses fail the pipeline. Conditional edges without explicit failure handling cause termination. Parenthesized conditions rejected at parse time. Every routing decision logged as a structured event. This is not the fastest engine — it's the one you can debug after a $100 run goes wrong.

**Multi-backend over deep integration**: Three backends share a common interface but diverge on tool safety enforcement and cost estimation. Native is most capable; claude-code and ACP are best-effort. The trade-off is that pipeline authors need to know backend capabilities.

**Filesystem as sandbox boundary**: Parallel agents use `git worktree` isolation — each gets a separate filesystem view. No container-level isolation (compare [[yolo-cage]], [[Stockyard]]). Worktree isolation is fast but shares the same kernel; defense in depth relies on the tool safety system.

**Pipeline spec as product**: The `.dip` file is the durable artifact. Tracker is the disposable runner. This matches the thesis in [[Specifications as the Product]] and [[The Dark Factory is a DOT File]] — and is the architectural bet that drove the Dippin language development.

## Comparison Notes

- **vs [[The Dark Factory is a DOT File]]**: Tracker IS the reference implementation. The three-layer architecture described there (LLM client, agent loop, pipeline engine) is Tracker's actual code structure.
- **vs [[Cord]]**: Cord has dynamic task decomposition; Tracker has static DAG validation. Cord is ~500 lines of Python prototyping a protocol; Tracker is ~67K lines of production Go.
- **vs [[workgraph]]**: Both treat the workflow as the durable artifact. Workgraph uses dynamic claims/handoffs; Tracker uses pre-defined graphs with conditional routing. Workgraph stores state as JSONL on disk; Tracker uses a checkpoint JSON file + sentinel-protected JSONL audit log.
- **vs [[Swamp Club]]**: Both execute DAGs. Swamp Club adds Zod typing + encryption; Tracker is more generic and backend-agnostic. Swamp Club targets agent-to-agent cooperation; Tracker targets human-authored workflow definitions.
- **vs [[speedrift-ecosystem]]**: Speedrift is a higher-level control plane that observes multiple repos and detects drift. It could orchestrate Tracker runs as workers.
- **vs [[The Claude Code Playbook]] / [[A Guide to Claude Code 2.0]]**: Tracker's Claude Code backend wraps the same CLI these guides describe. The key difference: Tracker adds graph-level orchestration and audit trails on top.

Tags: #tool #project #orchestration #agents #pipeline #multi-agent #dippin

---

*Source: [[raw/tracker-2389]]*
*Last updated: 2026-05-31*
