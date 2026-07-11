# no-mistakes

A local git proxy that gates pushes through an AI-driven validation pipeline — review, test, document, lint, push, PR, and CI monitoring — forwarding your branch to the remote only after every check passes. Agent-agnostic (Claude, Codex, OpenCode, RovoDev, Pi, Copilot, or any ACP target), runs in an isolated git worktree without disrupting your work, and produces clean PRs by default.

---

## Architecture

`no-mistakes` is a Go CLI (~46K LoC production code) with a daemon-backed pipeline executor. Three components work together:

**The git proxy gate** (`internal/gate/`): `no-mistakes init` creates a bare git repo in `~/.no-mistakes/repos/<id>.git` with a post-receive hook, then adds a `no-mistakes` remote pointing at it. `git push no-mistakes` triggers the hook, which notifies the daemon. The gate never rejects a push — failures are logged to stderr and a notification log.

**The daemon** (`internal/daemon/`): a long-lived background process that receives push notifications, spins up disposable git worktrees, and runs the pipeline executor. Uses SQLite (`modernc.org/sqlite`, no CGo) for all durable state. Communicates with the CLI/TUI via Unix socket IPC. Managed by platform-native service managers (launchd, systemd, Windows scheduled tasks). On startup, recovers any runs that were parked at approval gates when the daemon last stopped.

**The TUI** (`internal/tui/`): a Bubble Tea terminal interface showing pipeline progress, findings, diffs, and approval actions. Three entry points trigger the same pipeline: `git push no-mistakes`, `no-mistakes` (TUI with optional wizard), and `/no-mistakes` (agent skill for headless operation via the `axi` TOON interface).

## Pipeline

Nine sequential steps in fixed order, defined in `internal/types/types.go`:

1. **Intent** — extracts user intent from local agent transcripts (configurable, default on)
2. **Rebase** — rebases onto current default branch tip; auto-fixes conflicts (3 attempts)
3. **Review** — AI code review for bugs, security issues, doc gaps; auto-fix disabled by default
4. **Test** — runs repo's test command; gathers evidence artifacts (screenshots, logs)
5. **Document** — combined document+lint housekeeping: docs review plus lint assessment in one agent call
6. **Lint** — consumes lint findings from document step; runs repo's deterministic lint command
7. **Push** — force-pushes to configured target (origin or fork)
8. **PR** — creates PR/MR via GitHub, GitLab, Azure DevOps, or Bitbucket adapters
9. **CI** — monitors CI checks, auto-rebases on base branch movement, auto-fixes CI failures

Each step produces structured JSON findings (`internal/types/findings.go`) with severity, file location, description, and action: `auto-fix` (applied automatically within limit), `ask-user` (parks for human decision), or `no-op` (informational). Missing/unset action defaults to `ask-user` — a deliberate fail-closed design.

## Key Techniques

**Git worktree isolation**: the pipeline runs in a disposable worktree from the bare repo. Your working directory stays untouched. The hooks path is isolated per-worktree so tools like husky can't disable the gate's post-receive hook.

**Structured output with multi-format extraction** (`internal/agent/agent.go`): three-tier parsing — direct JSON parse, code-fence extraction (handles fences glued to preceding text), and last-bare-JSON-object scan. This handles real model output which is messier than the API contract.

**Finding fingerprint deduplication** (`internal/pipeline/findings.go`): findings are matched by structural hash (description+file+severity, optionally ignoring line number). Prevents duplicate findings from accumulating across fix→review cycles.

**Config trust boundary** (`internal/config/config.go` `EffectiveRepoConfig`): code-executing fields (commands, agent) in `.no-mistakes.yaml` are only honored from the trusted default-branch copy. A contributor's pushed branch cannot inject shell commands or select an agent, unless the maintainer explicitly opts in with `allow_repo_commands: true`.

**Durable approval parking** (`internal/pipeline/executor.go` `Resume`): when a step parks for approval, the state is persisted to SQLite with step status, findings JSON, and duration. On daemon restart, `Resume()` reconstructs the gate and re-attaches — runs survive crashes, restarts, and upgrades.

**Session durability** (`internal/pipeline/sessions.go`): the review loop maintains two separate agent sessions per run — reviewer (all full reviews) and fixer (all fix turns). Sessions are provider-specific and persisted to SQLite. Resume failure drops the dead session and starts fresh; never skips a turn.

**Agent fallback chains** (`internal/agent/fallback.go`): when `agent: [codex, claude]` is configured, the pipeline tries agents in order, falling through only on process-start failures. Session-aware routing ensures resume attempts reach the provider that minted the session.

**Combined document+lint pass** (`internal/pipeline/shared.go` `RunShared`): the document step runs both documentation review and lint assessment in one agent invocation, stashing lint findings in memory. The lint step consumes them with a one-shot `Take` that clears the value, so fix rounds never consume stale findings.

## Design Decisions

**Sequential over parallel**: review, test, lint could run concurrently against the same diff. Sequential means each step sees the fixed state from previous steps and the approval model is simpler. Trade-off: latency.

**Human-in-the-loop by default**: auto-fix is off for review (riskiest category), on (3 attempts) for mechanical steps (lint, test, rebase). Unclassified findings default to `ask-user` — fail closed, not fail open.

**Agent-agnostic by design**: every agent backend implements the same `Agent` interface with structured output parsing, session support detection, and lifecycle callbacks. ACP bridge extends this to future agents. Trade-off: prompts must target lowest common denominator.

**SQLite as sole state store**: zero setup, single file, works everywhere. Schema is versioned. Right call for a single-user desktop tool.

**Worktree isolation, not containerization**: simpler and faster than Docker/VMs, but provides weaker isolation. Acceptable for a developer-side gate (not a CI server processing untrusted code).

**Filesystem as config boundary**: repos ship `.no-mistakes.yaml`; global config lives at `~/.no-mistakes/config.yaml`. No web dashboard or server-side config needed.

## Comparison Notes

Unlike CI/CD pipelines which validate *after* the push, `no-mistakes` gates *before* — the branch only reaches the remote after every check passes. The CI step then monitors the post-push pipeline as a complementary babysitter.

Unlike [[Bram]] (hash-verified worklist lifecycle) or [[Fleet Supervisor (sermakarevich)]] (parallel agent orchestration), `no-mistakes` is specifically a *push gate* triggered by `git push`, not a general-purpose agent harness.

Unlike [[OpenCodeReview]] (Alibaba's hybrid AI review), which focuses on code review comments, `no-mistakes` is a full 9-step pipeline including test execution, documentation, PR creation, and CI monitoring — and it gates the push itself.

Unlike [[Orchestrating AI Code Review at Scale]] (Cloudflare's 7-agent + judge system), which runs server-side with parallelism, `no-mistakes` uses a single agent per step running locally. Different trade-off: Cloudflare gets parallel review dimensions; `no-mistakes` gets privacy and zero infrastructure.

The auto-fix feedback loop (find→fix→re-review→approve) is a concrete implementation of the pattern described in [[Guardrails and Feedback Loops]]. The finding fingerprint deduplication ensures the same issue isn't reported twice across cycles.

#tool #project #agents #git #code-review

---
*Sources: [[raw/no-mistakes]]*
*Last updated: 2026-07-11*
