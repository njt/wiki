# Fleet Supervisor (sermakarevich)

Production-grade Python supervisor (~4,700 LoC) that claims tasks from a centralized beads queue and runs them in parallel through pluggable coder backends (Claude Code, agy, Codex CLI, OpenCode). Each task carries its own project working directory and optional coder/model override, so a single supervisor drives work across many projects from one machine. Ships with a FastAPI+React web UI, Telegram notifications and inbound task commands, and a bundled MCP server for human-in-the-loop questions. The supervisor itself is intentionally dumb — it manages task lifecycle (claim, spawn, reap, retry, block) without understanding task content; all intelligence lives in the agent's coder CLI and the shared protocol templates.

---

## Architecture

Fleet is a **single-process async supervisor with pluggable coder subprocesses**, structured as five concurrent asyncio loops plus a FastAPI web server.

### The five loops (`src/fleet/supervisor.py:80-93`)

The `Supervisor.run()` method starts five background asyncio tasks that constitute the entire runtime:

1. **Claim-and-spawn** — polls `bd ready` every 5s, checks concurrency cap and rate-limit gauge, atomically claims the highest-priority task via `bd update --claim`, spawns a `TaskRunner` subprocess in the task's `cwd`
2. **Reap** — `asyncio.wait(FIRST_COMPLETED)` on all in-flight tasks, reads the `TaskOutcomeRecord`, dispatches to the outcome handler
3. **Config poll** — re-reads `runtime.toml` every 5s via mtime comparison, applies changes without restart
4. **Status log** — emits a structured heartbeat every 30s with in-flight count, rate-limit usage, and per-task context tokens
5. **Kill poll** — watches for `.kill` sentinel files per task directory, terminates the matching runner

### Task runner (`src/fleet/runner.py:62-317`)

Spawns the coder CLI via `asyncio.create_subprocess_exec` with a 100 MB stdout buffer (up from Python's 64 KB default) to avoid `LimitOverrunError` on large MCP results. Reads stdout line-by-line as JSON events, normalizes through the coder's `normalize_event()`, appends to `events.jsonl`. Monitors context token accumulation against `context_pressure_threshold_pct` and SIGTERMs the agent when exceeded — the supervisor then re-queues the task for a fresh session. On subprocess exit, classifies into six outcomes: SUCCESS, FAILURE, RATE_LIMIT, CONTEXT_PRESSURE, BLOCKED_BY_AGENT, KILLED.

### Outcome handler (`src/fleet/supervisor.py:439-595`)

The most complex single method. SUCCESS checks whether the agent called `bd close` (the "close" protocol) — if not, increments a no-close counter (blocks after 12x). FAILURE increments a retry counter (blocks after 2x by default). RATE_LIMIT sets a `_paused_until` timestamp that gates the spawn loop. CONTEXT_PRESSURE releases the task for a fresh claim. BLOCKED_BY_AGENT is a no-op (the agent already set the status). KILLED blocks the task with a "manually interrupted" note.

### Coder abstraction (`src/fleet/coders/`)

Abstract `Coder` base class (`base.py`) defines four methods:
- `build_argv(task, task_dir)` → argv list for `create_subprocess_exec`
- `env(task, task_dir)` → env vars (`FLEET_TASK_ID`, `FLEET_TASK_DIR`, `FLEET_ARTIFACT_DIR`)
- `normalize_event(raw_line)` → `Event` dataclass (or None to drop)
- `write_runtime_config(project, task)` → inject config files before spawn

Four implementations:

| Coder | CLI invocation | Context limit | Event format |
|-------|---------------|---------------|--------------|
| `claude` | `claude -p --verbose --output-format stream-json` | 200K | `rate_limit_event`, `assistant`, `tool_use`, `tool_result`, `result` |
| `agy` | `agy -p --dangerously-skip-permissions` | 128K | Raw text lines (no structured JSON); JSON parsed opportunistically |
| `codex` | `codex exec --json --dangerously-bypass-approvals-and-sandbox` | 128K | `thread.started`, `item.started/updated/completed`, `turn.completed` |
| `opencode` | `opencode run --format json` | 128K | `step_start`, `text`, `tool_use`, `step_finish` |

The `opencode` coder is the most elaborate — it writes/refreshes `opencode.json` with an ollama-rtx provider entry, `ask-human` MCP server, `playwright` MCP, and `claude_code` MCP before each spawn (`coders/opencode.py:83-185`).

### Worktree isolation (`src/fleet/worktree.py`)

Opt-in via `FLEET_WORKTREE_ISOLATION=1`. Creates a git worktree per task on branch `fleet/<task_id>` (`worktree.py:26-50`). On agent exit with a clean commit, the supervisor merges the branch into main (`worktree.py:106-158`); conflicts block the task for manual resolution. Orphaned worktrees are swept on startup (`supervisor.py:103-125`).

### Web UI (`src/fleet/serve/app.py`)

FastAPI app factory with lifespan-managed background tasks: WebSocket file watcher for live event streaming, a 2-second Telegram question poller, and a Telegram inbound command listener. Seven route modules. React SPA frontend with five tabs (Dashboard, Task Detail, Chat, Analytics, Config), served via `StaticFiles` with SPA fallback.

### Daemon manager (`src/fleet/daemon.py:116-296`)

POSIX PID-file manager for both the supervisor and UI server. tracks running state via JSON PID files with SHA1 version fingerprints for stale-code detection. SIGTERM → grace window → SIGKILL escalation (`daemon.py:240-268`). `restart()` runs a `before_start` callback (e.g. `make ui-build`) before stopping the old daemon, so a failed build doesn't take the running service down (`daemon.py:284-295`).

### Agent protocol (`src/fleet/templates/`)

Every agent receives a composite prompt: header template (task ID, title, description, artifact paths) + `INSTRUCTION.md` (read PLAN_AND_STATUS.md and KNOWLEDGE.md on start, write progress updates, close with `bd close` when done, use `mcp__ask_human__ask_human_question` when blocked). For isolated tasks, `ISOLATED_PROTOCOL.md` is appended: commit to branch, don't close, exit 0.

### ask_human MCP broker (`src/fleet/ask_human/`)

Bundled MCP server on stdio. Agents call `ask_human_question`; the question is INSERTed into a shared SQLite DB. The agent blocks (polling) until a human answers via the web UI Chat tab or Telegram reply. Answering is an atomic `UPDATE WHERE status='pending'` — first responder wins. Shared DB means all frontends (web, Telegram, CLI) are thin clients.

---

## Key Techniques

**Rate gauge with auto-reset** (`src/fleet/rate_gauge.py:35-50`). `RateGauge.current_pct()` checks whether `now >= resetsAt + 5s` and zeros the gauge if so. Falls back to a 5-hour decay if no `resetsAt` was provided (edge case where the API didn't include one). Only Claude's `five_hour` rate limit type is tracked; weekly/overage limits are filtered out.

**Context-pressure enforcement** (`src/fleet/runner.py:191-221`). After every event with token usage, computes `peak_context_tokens / context_limit * 100`. Logs at each 10% bucket. When it exceeds `context_pressure_threshold_pct` (default 90%), writes a `.context_pressure` flag, SIGTERMs the agent, waits up to `SHUTDOWN_GRACE_SEC`, then SIGKILLs. The reaper detects the flag and classifies the outcome as CONTEXT_PRESSURE.

**Write-then-rename atomicity** — used for `task.json` (`queue.py:77-86`), `runtime.toml` (`config.py:94-104`), PID files (`daemon.py:163-173`), and `opencode.json` (`coders/opencode.py:183-185`). Every write goes to a `.tmp` sibling then `os.replace()` — no torn reads.

**Sentinel-file state machine** — `.kill` triggers manual termination (`supervisor.py:375-386`), `.pause` gates the spawn loop (`supervisor.py:139`), `.context_pressure` flags auto-termination (`runner.py:263-270`), `.failures` counts retries (`failures.py:19-24`), `.needs_validation` queues worktree merges (`failures.py:67-79`). All state is filesystem-inspectable with no database migration.

**Config hot-reload** (`src/fleet/config.py:55-65`). `reload_if_changed()` compares `os.stat().st_mtime` against a stored timestamp, returns (new_config, new_mtime) only on change. The supervisor applies the new config atomically by replacing `self.config`.

**Per-task coder/model freeze** (`src/fleet/supervisor.py:282-285`). On first claim, the resolved (coder, model) is written to `task.json` via `freeze_coder_model()`. This insulates in-flight tasks from config changes — if the user switches the default coder mid-run, existing tasks continue with what they started.

**PID-file stale detection** (`src/fleet/daemon.py:38-54`). `code_fingerprint()` SHA1s all `.py` files in the fleet package, stored in the PID file at start. `status()` compares stored vs. current and surfaces `stale=True` when they differ — the CLI prints a warning to restart.

**Pluggable coder registry** (`src/fleet/coders/__init__.py:7-12`). A dict mapping name strings to classes. `get_coder()` raises `ValueError` with available names on unknown input. Validation happens at config-set time and at task-claim time.

---

## Design Decisions

**Centralized queue, distributed work.** All tasks live in `~/.fleet/.beads`, but each task stores its own `cwd`. A single supervisor spawns agents across many projects. This is the opposite of per-project agent setups — coordination is centralized, work is distributed. The trade-off: fleet home becomes a single point of failure for task state.

**Dumb supervisor, smart agent.** The supervisor never inspects task content — only manages lifecycle. All intelligence is in the agent (via coder CLI + protocol templates). The spawn loop is ~45 lines; the outcome handler is ~160 lines. This is [[Smart Models Dumb Pipes]] applied to orchestration: the agent is the smart component, the supervisor is the dumb pipe.

**beads as the database.** Rather than building a custom queue, fleet delegates entirely to Gas Town's `bd` CLI. This avoids reinventing atomic claiming, dependency management, and status tracking — but introduces a Go binary dependency and, per [[Gas Town After 10,000 Hours of Claude Code]], pollutes git history with agent bookkeeping commits.

**File-based task state, not a database.** Per-task state is just files in `$FLEET_HOME/tasks/<id>/`: counters (`.failures`, `.noclose`), markers (`.kill`, `.context_pressure`, `.needs_validation`), metadata (`task.json`), events (`events.jsonl`), artifacts (`.md` files). No schema migrations, no ORM, inspectable with `cat` and `ls`. The trade-off: no querying across tasks without reading all files.

**MCP-based human-in-the-loop.** Claude Code `-p` mode filters `AskUserQuestion`, so fleet bundles a full MCP server instead of working around the limitation with file-based Q&A. The agent calls `ask_human_question` as a tool; it blocks until answered from any frontend. The answer is delivered directly (no re-invocation needed). This is more robust than [[Managing Agents via Kanban Boards]]' comment-polling approach because it uses a proper client-server protocol with atomic delivery.

**No task decomposition.** Unlike [[Cord]] (agents spawn subtrees dynamically) or [[Gas Town's Agent Patterns]] (the mayor decomposes work), fleet has no concept of subtasks. It's a flat priority queue with optional `depends_on` (via beads). This means the human (or an external agent) must decompose work before fleet sees it. The trade-off: simpler supervisor, more upfront work.

**Agent-close protocol, not supervisor detection.** The supervisor doesn't detect "done" — it relies on the agent calling `bd close` before exiting. If the agent exits 0 without closing, the supervisor re-queues the task. After 12 no-close cycles, the task is blocked. This pushes the "am I done?" judgment to the agent, where it belongs.

---

## Comparison Notes

**vs the Fleet of Agents tutorial** (same author): [[Fleet of Agents (sermakarevich)]] is the five-step tutorial that walks from a single agent to a parallel fleet. This repo is the production implementation of step 5 and beyond — it takes the bash loop pattern and adds production concerns: configurable concurrency, rate-limit awareness, context-pressure detection, automatic retry, worktree isolation, a web UI, Telegram integration, and MCP-based Q&A. The core insight (dumb loop, smart agent) is preserved.

**vs Gas Town:** Gas Town is a monorepo orchestrator with hierarchical decomposition. Fleet is a cross-project scheduler with flat task queues. Gas Town's mayor is intelligent; fleet's supervisor is dumb. Fleet uses Gas Town's beads for the queue — it's a user of Gas Town's infrastructure, not a competitor.

**vs Ralph/Huntley pattern:** Fleet is [[Ralph]] with production infrastructure. The bash loop becomes an async supervisor with five concurrent loops. The TODO.md becomes a beads SQLite database with atomic claiming. The Q&A file becomes an MCP server with multiple frontends. The pattern is identical; the implementation is industrial-grade.

**vs [[Maestro]]:** Maestro has role separation (PM/Architect/Coder); fleet has none. In Maestro, the Architect reviews but never writes code. In fleet, any agent can do any task — the role assignment is external (the human chooses the coder/model per task).

**vs [[Cord]]:** Cord gives agents primitives to build task trees dynamically. Fleet has no task tree — it's a priority queue. Cord agents coordinate with each other; fleet agents are independent workers that only interact through the queue.

---

#tool #orchestration #supervisor #claude-code #coding-agents #parallel #telegram #mcp #beads

---

*Source: [[summary/fleet-sermakarevich]]*
*Last updated: 2026-06-15*
