# LongHorizon-Harness

A Python harness (~20K lines of source) that turns existing coding-agent CLIs into long-running computer-use systems by wrapping them in a plan → act → verify → checkpoint-or-recover loop. The key move: it never replaces the agent, it engineers the durable execution loop *around* it. Three roles — Manager, Executor, Auditor — split the loop into focused responsibilities, and only auditor-verified results become trusted task state. Reported to take WeaveBench from 51.8 → 80.7, OSWorld 2.0 from 2.8 → 8.3 (3×), and Terminal-Bench 2.1 from 69.7 → 77.2 with 24% fewer tokens, all on the same Qwen 3.7-Plus backbone.

---

## Architecture

**A single asyncio loop with a pluggable role/backend spine.** The whole system hangs off two `Protocol`s in `src/lh_harness/`:

- `AgentAdapter` (`adapters/base.py`, 17 lines) — `run_episode(prompt, env, budget, live_trajectory_path) -> EpisodeResult`. Every backend is a `CommandAgentAdapter` that shells out to a CLI and parses its stream.
- `Environment` (`environment/base.py`, 21 lines) — `exec`/`screenshot`/`upload`/`download`. `LocalEnvironment` runs agents in their own process group (`start_new_session` + killpg) with streaming stdout tee for the live dashboard.

**The core loop is `manager.py` (2,480 lines).** Role bindings are resolved once at startup (`manager.py:224`), so each of the five roles (manager, gui/cli executor, gui/cli auditor, plus final_response) can bind a different agent/model. Each round: build the manager prompt → run the manager → parse its `Next: gui|cli|done|blocked|ask` route → run the bound executor → run the auditor → record the round. The route enum is `RoleNextStep` in `types.py:43`.

**A supervisor/worker split.** `supervisor/service.py` is deliberately "a small supervisor around the existing CLI" — the worker is the normal `lh-harness run` process, and the supervisor owns its lifecycle and a command boundary. Operator commands (stop, restart, inject instruction) travel over an **append-only JSONL control bus** (`supervisor/control_bus.py`), where a receipt is the only terminal authority — so a browser disconnect or API restart never drops an operator action. The FastAPI server (`webapi/server.py`, 1,317 lines) and a Vite/React frontend hang off it.

**Everything is durable by default.** Rounds append to `rounds.jsonl`; events to `events.jsonl`; the `run()` wrapper in `manager.py:128` is a crash boundary that *always* writes a terminal report even on `BaseException`. Resume (`manager.py:269`) reopens the ledger and rebuilds the manager's full planning context by replaying recorded rounds.

## Key Techniques

**A three-line control header instead of JSON.** The auditor's report is natural language, but its first three lines are a machine-readable protocol — `Status: complete|incomplete|blocked`, `Integrity: clean|suspect|violation`, `Contract audit: aligned|unknown|needs_revision|invalid` — parsed by regex in `auditor_agent.py`. If the header is missing, a **format-repair episode** re-prompts the same auditor to re-emit the report ("formatting repair, not a new audit") before the harness will accept it. The report stays human-readable *and* becomes the trusted intermediate state; the parser is only a transient stop-condition check (`manager.py:1756`).

**Completion is a harness decision, never the executor's self-report.** The final report carries `completion_authority: "manager_with_role_auditors"`. The manager can only emit `Next: done` if the *latest* auditor report is complete/clean/aligned (`_latest_auditor_is_clean_complete`, `manager.py:1522`); otherwise the harness synthesizes an invalid-completion report and feeds it back as a repair signal. An executor saying "I'm done" is a claim; an auditor confirming files/logs/tests is the only accepted proof.

**The auditor is enforced read-only by filesystem snapshot, not by trusting the prompt.** `ClaudeCodeAdapter.run_episode` (`adapters/claude_code.py:137`) snapshots the workspace before the auditor runs and diffs it after. Any auditor write downgrades the audit to `blocked/violation`, restores the snapshot, and invalidates any claim built on auditor-created files. This is the concrete "verifier must not grade its own homework" guarantee.

**Fresh context per episode — the harness is the memory.** Claude Code is launched with `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` and `CLAUDE_CODE_SKIP_PROMPT_HISTORY=1`; every role starts cold. Cross-round context flows only through the manager-maintained `task_state`/`task_contract` strings and the auditor reports the manager references by round id (`extract_related_report_refs` → `format_related_auditor_reports`). The executor *never* sees raw trajectories or prior prompts — only "audited state and the stable contract" as its semantic boundary. Character budgets are explicit (`role_verified_context_chars=60k`, `role_history_chars=100k`), and `_clip_preserve` keeps head+tail on truncation.

**"Evidence, not progress."** A rejected or failed result is still recorded — executor output, harness feedback, and the failure classifier's message all persist into the round record, so the next manager turn starts from the original goal + last verified checkpoint + the failure evidence. Timeouts are treated as *recoverable* (the next manager inspects the real workspace) rather than as provider failures.

**Defensive filesystem engineering throughout.** Because the agent runs arbitrary commands, every harness-owned read/write walks paths with `O_NOFOLLOW` component-by-component to defeat symlink swaps, enforces `st_nlink == 1` (no hard links), and bounds every read/write (`_MAX_SAVED_TRAJECTORY_BYTES = 16 MiB`, event log caps, tail-only reads). `./.lh-harness/` is off-limits to the agent so its own logs are never mistaken for task content.

## Design Decisions

**Correctness and verifiability over speed.** Every round costs at least two model episodes (manager + executor + auditor, occasionally + format-repair + final-response). That's the obvious sacrifice — and the headline result that Terminal-Bench still ends up 24% *cheaper* in tokens than the single-shot agent suggests the verification loop buys back its cost by stopping wasted work early. GUI tasks remain expensive (OSWorld uses many screenshot-bearing episodes).

**Natural-language state over structured state.** `task_state`, `task_contract`, and auditor reports are all prose re-fed into prompts, not typed objects. This maximizes flexibility and debuggability — but it means the harness's understanding of progress is regex-mediated and can be gamed by a malformed report, which is exactly why format-repair and the completion guard exist. A stronger-typed state machine would be more reliable and less inspectable; they chose inspectability.

**Wrap the CLI, don't reimplement the agent loop.** Driving `claude --print --output-format stream-json`, `codex`, `opencode run`, and `dsh` means inheriting all backend capability (and their quirks) for free — but also inheriting `--dangerously-skip-permissions`, `codex-computer-use`'s manual macOS grants, and DeepSeek's positional task leaking into the child process arg list (documented in the README). Computer-use itself is delegated to npm MCP plugins (`codex-computer-use`, `open-computer-use`, `clawdcursor`) loaded per-run from `~/.lh-harness/`, never touching user-scope MCP registries.

**Per-role model assignment as a cost lever.** `[run.roles.*]` with an inheritance chain (`gui_executor → executor → [run]`) lets you pay for a strong Manager/Auditor and a cheap Executor. This is the same multi-dimensional budgeting pattern [[Building an Advanced Agentic Harness]] describes as a production primitive, but baked into the config schema.

**Agent availability is tri-state, not boolean.** `agent_registry.py` distinguishes `usable` / `found_but_broken` / `missing`, because `shutil.which` reports a Windows Store zero-byte `codex.exe` alias as "present" while it fails every real call. `lh-harness doctor` verifies each CLI by actually running `<binary> --version`.

## Comparison Notes

**vs. [[Cua — Computer Use Agent Platform]]** — Cua is an SDK/platform for *building* computer-use agents (macOS drivers, VM orchestration, per-model agent loops). LongHorizon-Harness builds no agent at all: it wraps CLIs the user already has and supplies only the verification loop and state management. Cua is the substrate; this is the loop around a substrate it didn't build. The trade: Cua controls the screen at the driver level; LH-Harness gets screen control secondhand via MCP plugins.

**vs. [[LLM-as-a-Verifier]]** — both make verification a first-class scaling axis, but by opposite mechanisms. LLM-as-a-Verifier is a *probabilistic* verifier (logit-expectation scoring) that needs logit access and generalizes across domains. LH-Harness's verifier is *behavioral* — a second agent with read-only filesystem enforcement and a control-header contract — so it works with closed, CLI-wrapped models, at the cost of needing a full agent episode (and the ground-truth access that requires) per check.

**vs. [[Slate]]** — Slate argues the long-horizon bottleneck is context management and proposes threads-as-processes with episodes-as-compressed-return-values. LongHorizon-Harness is a production realization of the same shape: fresh-context episodes whose "return value" is a natural-language auditor report that gets selectively re-injected. Slate generalizes the routing; LH-Harness fixes it to a three-role plan/act/verify loop and adds filesystem-grounded verification Slate leaves unbenchmarked.

**vs. [[Loop Engineering]]** — Osmani's term names the meta-skill of designing systems that prompt agents; LongHorizon-Harness *uses the term itself* for a concrete, fixed implementation: a deterministic three-role loop with checkpoint/recovery, shipped as `pip install lh-harness`. Where Osmani's taxonomy is open-ended (automations + worktrees + skills + connectors + sub-agents), this is one such designed system, specialized to computer use, with the sub-agent separation hard-coded as Manager/Executor/Auditor.

---

*Sources: [[raw/longhorizon-harness]], [[summary/longhorizon-harness]]*
*Last updated: 2026-08-21*
