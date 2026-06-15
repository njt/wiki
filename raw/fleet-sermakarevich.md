---
url: https://github.com/sermakarevich/fleet
title: fleet — Python supervisor for running coding agents in parallel
author: sermakarevich
date_fetched: 2026-06-15
date_published: unknown
---

# fleet — Python supervisor for running coding agents in parallel

A production-grade Python supervisor (`fleet`) that claims tasks from a centralized [beads](https://github.com/gastownhall/beads) (`bd`) queue and runs them in parallel through a coder CLI (`claude`, `agy`, `codex`, or `opencode`) in a headless loop. Each task remembers its project working directory and optional per-task coder/model override, so a single supervisor can drive work across many projects and agent backends from one machine. Ships with a FastAPI web UI (`fleet serve`), Telegram integration for notifications and inbound task creation, and a bundled `ask_human` MCP server for human-in-the-loop questions.

Total: ~4,700 LoC Python core (excluding tests/UI), ~500 LoC React TypeScript UI, 53 test files.

## Architecture

fleet is a **centralized supervisor with pluggable coder backends**, structured as an async Python event loop managing a pool of agent subprocesses.

### Supervisor loop (`supervisor.py`, 658 lines)

The `Supervisor` class is the heart of fleet. It runs five concurrent asyncio tasks:

1. **`_claim_and_spawn_loop`** — polls `bd ready` every 5 seconds, atomically claims the highest-priority task via `bd update --claim`, spawns a TaskRunner subprocess in the task's working directory
2. **`_reap_loop`** — waits for any child process to exit, reads the outcome, and handles it: success (with/without close), failure (retry up to 2x), context pressure (re-queue), rate limit (pause fleet), killed (block), blocked-by-agent
3. **`_config_poll_loop`** — re-reads `runtime.toml` every 5 seconds, applying config changes without restart
4. **`_status_log_loop`** — emits a heartbeat every 30s with in-flight count, rate-limit usage, context tokens per task
5. **`_kill_poll_loop`** — watches for `.kill` sentinel files per task directory, terminates the matching runner

The spawn decision (`supervisor_spawn.py`, 58 lines) gates on three conditions: max concurrent cap, rate-limit threshold (90% by default), and a `.pause` file at the fleet root for manual pausing. Non-Claude coders skip the rate-limit check.

### Task runner (`runner.py`, 354 lines)

`TaskRunner` spawns the coder CLI as an `asyncio.subprocess.Process`, reads stdout line-by-line as JSON events, normalizes them via each coder's `normalize_event()` method, and appends them to `events.jsonl`. It tracks:

- **Context pressure**: when `input_tokens` exceeds `context_pressure_threshold_pct` (default 90%) of the coder's context limit, it SIGTERMs the agent and sets a `.context_pressure` flag — the supervisor re-queues the task for a fresh session
- **Rate limit rejection**: when the coder emits a rate-limit rejection event, the runner releases the task and signals the supervisor to pause spawning
- **Outcome classification**: after subprocess exit, classifies into one of six `TaskOutcome` values (SUCCESS, FAILURE, RATE_LIMIT, CONTEXT_PRESSURE, BLOCKED_BY_AGENT, KILLED)

### Queue (`queue.py`, 331 lines)

Abstract `Queue` ABC with a single `BeadsQueue` implementation that wraps the `bd` CLI (Go binary from Gas Town's beads project). All task state lives in `$FLEET_HOME/tasks/<id>/task.json` as JSON metadata (cwd, coder, model, status). Operations delegate to `bd` subprocess calls with JSON envelope parsing. Key design: write-then-rename for atomic `task.json` updates.

### Coder abstraction (`coders/base.py`, 37 lines)

The `Coder` ABC defines four methods:
- `build_argv(task, task_dir)` — construct the CLI invocation for the agent subprocess
- `env(task, task_dir)` — environment variables (`FLEET_TASK_ID`, `FLEET_TASK_DIR`, `FLEET_ARTIFACT_DIR`)
- `normalize_event(raw_line)` — parse one stdout line into a normalized `Event` dataclass
- `write_runtime_config(project, task)` — inject coder-specific config files before spawn

Four built-in coders:
| Coder | CLI | Context Limit | Default Model | Notable |
|-------|-----|---------------|---------------|---------|
| `claude` | `claude -p --verbose --output-format stream-json` | 200K | sonnet | Injects PreCompact/PreToolUse hooks into `.claude/settings.json` |
| `agy` | `agy -p --dangerously-skip-permissions` | 128K | GPT-OSS 120B | Model from `~/.gemini/antigravity-cli/settings.json`; no model flag |
| `codex` | `codex exec --json --dangerously-bypass-approvals-and-sandbox` | 128K | o4-mini | Passes `--cd <cwd>` for per-task working directory |
| `opencode` | `opencode run --format json` | 128K | gpt-oss:20b | Writes `opencode.json` with ollama-rtx provider; SSH tunnel to remote GPU |

### Worktree isolation (`worktree.py`, 158 lines)

Opt-in via `FLEET_WORKTREE_ISOLATION=1`. Creates a git worktree per task on branch `fleet/<task_id>`, checked out from main. The agent works in isolation; on success, the supervisor merges the branch into main (with conflict detection). On conflict, the task is blocked for manual resolution. Orphaned worktrees are swept on startup.

### Web UI (`serve/app.py`, 207 lines)

FastAPI application with React SPA frontend, served via `StaticFiles` with SPA fallback. Lifespan hooks start a file watcher for WebSocket event streaming, a Telegram question poller (every 2s), and a Telegram inbound listener for `/new_task` commands. Seven route modules (tasks, beads, supervisor, config, analytics, search, chat).

### Daemon manager (`daemon.py`, 300 lines)

POSIX PID-file daemon manager for long-lived services (supervisor and UI server). Tracks running state via JSON PID files with version fingerprints (SHA1 of all `.py` files) for stale detection. Supports `start`/`stop`/`restart`/`status` with SIGTERM-grace-then-SIGKILL shutdown.

### ask_human MCP broker

Bundled MCP server that agents call via `mcp__ask_human__ask_human_question`. Questions are written to a shared SQLite database; the agent blocks (polling) until a human answers via the web UI chat tab or by replying to the Telegram notification. Answering is an atomic UPDATE WHERE status='pending' — first responder wins.

### Templates and agent protocol

The agent receives a composite prompt: a header template (task ID, title, description, artifact paths) + `INSTRUCTION.md` (protocol: read PLAN_AND_STATUS.md and KNOWLEDGE.md on start, write progress updates, close with `bd close` when done, use `ask_human_question` when blocked). For isolated tasks, `ISOLATED_PROTOCOL.md` is appended: commit to the branch, don't close the task, exit 0.

## Key Design Decisions

### Centralized queue, distributed work

All tasks live in a single `~/.fleet/.beads` beads database, but each task records its own `cwd` — the project directory the agent should work in. A single supervisor spawns agents across multiple projects. This is the opposite of per-project agent setups: coordination is centralized, work is distributed.

### Dumb supervisor, smart agent

The supervisor never understands task content — it only manages task lifecycle (claim, spawn, reap, retry, block). All intelligence is in the agent (via the coder CLI and the agent protocol templates). The supervisor's spawn loop is ~45 lines; the outcome handler is the most complex part at ~160 lines.

### Atomic claiming via beads

The race condition of parallel supervisors is solved by beads' `bd update --claim` which atomically assigns a task to a claimer. Multiple `./loop.sh` instances or multiple fleet supervisors would race without this — fleet itself runs one supervisor but relies on the same atomicity.

### Config hot-reload

`runtime.toml` is polled every 5 seconds; changes to `max_concurrent`, `coder`, `model`, `context_pressure_threshold_pct` take effect without restart. The `config set` command writes atomically via mkstemp + os.replace.

### Per-task coder/model freeze

When a task is first claimed, the resolved (coder, model) pair is frozen into `task.json`. This prevents config changes mid-task from affecting retries or context-pressure reclaims. Overrides set at create time survive config changes.

### Filesystem as agent memory

Per-task state lives under `$FLEET_HOME/tasks/<id>/`: `task.json` (metadata), `events.jsonl` (structured event stream), `log.jsonl` (supervisor log), `.failures` (retry counter), `.noclose` (no-close counter), `.worktree` (worktree path marker), `artifacts/` (PLAN_AND_STATUS.md, KNOWLEDGE.md, other agent outputs). The filesystem is the persistence layer — no database migration, no schema, inspectable with `cat`.

### Human-in-the-loop via MCP, not tool approval

Claude Code in `-p` mode filters out `AskUserQuestion`. Instead of working around this with prompts, fleet bundles a full MCP server (`ask_human`) that agents call as a tool. The question blocks until answered from any frontend (web UI or Telegram). This is more robust than file-based Q&A because it's a proper client-server protocol with atomic answer delivery.

## Comparison to Related Systems

**vs Gas Town** (Steve Yegge's system): Gas Town is a monorepo orchestrator with a "mayor" agent that decomposes work and assigns to workers, all within one repo. Fleet is a cross-project supervisor with no intelligent decomposition — it relies on humans (or other agents) to create well-scoped tasks in beads.

**vs Ralph/Huntley's bash loop**: Fleet is the productionized version of the pattern the tutorial describes. Where the tutorial uses a 15-line `loop.sh`, fleet adds: configurable concurrency, rate-limit awareness, context-pressure detection, automatic retry, worktree isolation, a web UI, Telegram integration, and MCP-based human-in-the-loop. Fleet is 4,700 lines of Python vs. a bash one-liner — the complexity comes from production concerns, not from the core loop pattern.

**vs Cord**: Cord gives agents five primitives (spawn, fork, ask, complete, read_tree) to build task trees dynamically. Fleet has no task tree — it's a flat priority queue. Tasks can have `depends_on` relationships (via beads), but agents don't decompose or spawn subtasks. Fleet is a scheduler, not a workflow engine.

**vs Maestro**: Maestro has role-based teams (PM, Architect, Coder) with the Architect reviewing but never writing code. Fleet has no roles — any agent can do any task. The role separation is external: the human creating the task decides which coder/model to use.

**vs Dororthy/klaw.sh**: These are session managers for parallel agents with tmux/kubectl metaphors. Fleet is a production scheduler with lifecycle management. Dorothy shows you your agents; fleet decides when to spawn them.

## Key Techniques

1. **Async subprocess management** — `asyncio.create_subprocess_exec` with stdout line-by-line parsing, 100MB buffer limit (up from 64KB default) for large MCP tool results, SIGTERM → grace → SIGKILL escalation
2. **Rate gauge with auto-reset** — `RateGauge` tracks API usage percentage from stream events, auto-resets after the `resetsAt` timestamp + 5s grace, falls back to 5-hour decay if no reset time provided
3. **Write-then-rename atomicity** — used for `task.json`, `runtime.toml`, PID files, `opencode.json` to prevent torn reads from concurrent access
4. **Event normalization layer** — each coder's `normalize_event()` maps vendor-specific JSON formats (Claude's `stream-json`, Codex's `item.started/item.completed`, OpenCode's `step_start/step_finish`) into a uniform `Event` dataclass
5. **Config injection per spawn** — `write_runtime_config()` is called before each subprocess spawn, allowing coders to inject hook scripts (Claude), provider configs (OpenCode), or MCP server registrations without manual setup
6. **Sentinel-file signaling** — `.kill` for manual termination, `.pause` for fleet-wide pause, `.context_pressure` for auto-termination tracking, `.failures` for retry counting, `.needs_validation` for worktree merge queue — all file-based, no in-memory state that can be lost
7. **PID-file version fingerprinting** — SHA1 of all `.py` files stored in PID file, compared on status check to detect "stale code" and warn the user to restart

## Telemetry

- Structlog for JSON-line logging to `$FLEET_HOME/logging/fleet-<date>.jsonl`
- Per-task `events.jsonl` with normalized agent events
- `fleet log [N]` for supervisor log tail
- `fleet tail <id> -f` for live per-task event streaming
- Web UI analytics tab with token usage and throughput charts
- Telegram notifications for blocked agent questions

## Dependencies

- **Python ≥ 3.11**, `uv` for package management
- **beads (`bd`)** — Go binary from [gastownhall/beads](https://github.com/gastownhall/beads) for SQLite-backed task queue with atomic claiming
- **Claude Code / agy / codex / opencode** — at least one coder CLI on PATH
- **git** — for beads database and worktree isolation
- **FastAPI + Uvicorn + websockets** — web UI server
- **Typer + Rich** — CLI with colored task tables
- **structlog** — structured logging
- **watchfiles** — file change detection for WebSocket events
- **mcp ≥ 1.0.0** — Model Context Protocol SDK for ask_human server
- **React + TypeScript + Vite** — web UI frontend
