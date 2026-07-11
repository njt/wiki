---
url: https://github.com/kunchenguid/no-mistakes
title: "no-mistakes: AI-Driven Git Gate for Clean PRs"
author: kunchenguid
date_fetched: 2026-07-11
date_published: 2025
---

# no-mistakes

A Go CLI tool (~46K LoC production, ~75K LoC tests) that installs a local git proxy ("gate") between your working repo and the remote. Push to `no-mistakes` instead of `origin`, and it spins up a disposable worktree, runs an AI-driven validation pipeline across multiple agent backends (Claude, Codex, OpenCode, RovoDev, Pi, Copilot, or ACP targets), and only forwards the branch to the configured push target after every check passes. Clean PRs by default.

## Architecture

### Repository structure

```
cmd/
  no-mistakes/      — CLI binary entry point, daemon bootstrap, background update check
  genskill/         — regenerates the committed /no-mistakes skill file from source of truth
  fakeagent/        — mock agent for e2e testing (Claude, Codex, OpenCode protocols)
  recordfixture/    — records real CLI sessions as e2e fixtures (spends API quota)
internal/
  agent/            — agent adapters: claude, codex, opencode, rovodev, pi, copilot, acpx (ACP)
  cli/              — cobra command tree: init, axi (headless agent-to-approval interface), TUI attach, stats
  config/           — global config (~/.no-mistakes/config.yaml) + per-repo (.no-mistakes.yaml) merging
  daemon/           — long-lived background daemon: event IPC, pipeline lifecycle, startup recovery
  db/               — SQLite (modernc.org/sqlite): repos, runs, steps, rounds, agent sessions, invocations
  gate/             — bare repo initialization, git remote wiring, post-receive hook installation
  git/              — git command wrappers, hook scripts, env setup, worktree isolation
  ipc/              — event type definitions for daemon↔CLI communication
  lifecycle/        — guard logic: skip pipeline when push only touches generated files
  paths/            — NM_HOME directory layout resolution
  pipeline/         — core executor: sequential step orchestration, fix loop, approval coordination, sessions
  pipeline/steps/   — individual step implementations: intent, rebase, review, test, document, lint, push, pr, ci
  scm/              — SCM adapters: github (gh CLI), gitlab (glab), azuredevops (az), bitbucket (API)
  tui/              — Bubble Tea TUI: findings display, approval actions, diff view, CI monitor, wizard
  types/            — shared types: RunStatus, StepName, StepStatus, ApprovalAction, AgentName, Findings
  skill/            — generated agent skill source of truth
  update/           — self-update: GitHub releases, archive extraction, daemon lifecycle, macOS quarantine
  wizard/           — Bubble Tea wizard: create branch, commit uncommitted changes, push through gate
  intent/           — user intent extraction: read local agent transcripts, summarize change intent
  telemetry/        — Umami analytics (configurable, optional host)
  buildinfo/        — linker-injected version/commit/date/telemetry endpoint info
```

### The Git Proxy Pattern

`no-mistakes init` creates a bare git repo in `~/.no-mistakes/repos/<id>.git` and installs a post-receive hook. It adds the `no-mistakes` remote pointing at that bare repo. When you `git push no-mistakes`, the post-receive hook notifies the daemon, which:

1. Creates a disposable git worktree from the bare repo
2. Runs the pipeline sequentially: intent → rebase → review → test → document → lint → push → PR → CI
3. Each step either passes, produces findings for review, auto-fixes safe issues, or parks for human approval

### Daemon Architecture

The daemon (`internal/daemon/`) is a long-lived background process that:
- Listens for push notifications via CLI `daemon notify-push`
- Manages pipeline executor lifecycles
- Runs per-platform (launchd on macOS, systemd on Linux, scheduled task on Windows)
- Recovers parked runs on startup (runs awaiting approval survive daemon restarts)
- Uses SQLite for durable state — runs, steps, rounds, agent sessions, invocations

### Pipeline Executor (`internal/pipeline/executor.go`)

The core execution loop:
- Runs steps sequentially in fixed order (9 steps)
- Supports fix loops: when a step finds issues, auto-fix findings with `action: "auto-fix"` up to the configured limit
- Parks at approval gates for `ask-user` findings
- Tracks execution duration separately from approval wait time
- Recovers approval gates across daemon restarts (durable parking)
- Each round (initial run, auto-fix attempt, user-fix attempt) is persisted as a StepRound in SQLite

### Agent Abstraction (`internal/agent/agent.go`)

A uniform interface across six native agents plus ACP targets:
- `Agent.Run(ctx, RunOpts)` — single invocation with prompt, working dir, optional JSON schema, streaming callbacks
- `RunOpts.Session` — durable session resume (Claude and Codex support it; other agents run cold)
- `RunOpts.OnLifecycle` — lifecycle events for status display
- `RunOpts.OnAttempt` — per-attempt instrumentation for retry/fallback tracking
- Structured output parsing: JSON schema validation, code fence extraction, bare JSON object detection

### Fallback Agent (`internal/agent/fallback.go`)

When multiple agents are configured (e.g., `agent: [codex, claude]`), `NewFallback` wraps them in a chain:
- Tries each in order
- Only falls through on "agent unavailable" errors (process start failure, crash)
- Stops on the first success; surfaces the last error on total failure
- Session-aware: routes resume attempts to the provider that minted the session

### Session Management (`internal/pipeline/sessions.go`)

For the review step, durable agent sessions are maintained per run:
- Two roles: `reviewer` (all full reviews in a run) and `fixer` (all fix turns)
- Roles never share sessions — the fixer never inherits the reviewer's context
- Sessions are persisted to SQLite, survive daemon process boundaries
- Resume failure drops the dead session ID, re-runs in a fresh session (never skips a turn)
- Only Claude and Codex support sessions; other agents run cold

### Findings Model (`internal/types/findings.go`)

Every step produces structured JSON findings:
```json
{
  "findings": [{"id": "...", "severity": "error|warning|info", "file": "...", "line": N, "description": "...", "action": "no-op|auto-fix|ask-user"}],
  "summary": "...",
  "tested": ["..."],
  "testing_summary": "...",
  "risk_level": "...",
  "risk_rationale": "..."
}
```

Finding lifecycle:
- `auto-fix`: applied automatically within the configured attempt limit per step
- `ask-user`: parked for human attention (approve/fix/skip/abort)
- `no-op`: informational, doesn't block pipeline
- Deterministic IDs assigned during normalization: `<step>-N`
- User can add their own findings during fix review
- User instructions can be attached to specific findings
- Findings have fingerprint-based deduplication for merge/remove operations

### Config Merging and Trust (`internal/config/config.go`)

Two config layers, merged with a trust model:
- Global: `~/.no-mistakes/config.yaml` — agent preference, timeouts, auto-fix limits, intent settings
- Repo: `.no-mistakes.yaml` in repo root — commands, ignore patterns, auto-fix, document policy
- `EffectiveRepoConfig` enforces trust: commands and agent selection come from the trusted default-branch copy, not the pushed branch, unless `allow_repo_commands: true`
- This blocks the supply-chain vector: a contributor's pushed branch cannot inject shell commands or select an agent

### Pipeline Steps

1. **Intent** — extracts user intent from local agent transcripts (optional, configurable)
2. **Rebase** — rebases branch onto current default branch tip; auto-fixes conflicts
3. **Review** — AI code review against the diff; finds bugs, security issues, doc gaps
4. **Test** — runs repo's test command, gathers evidence artifacts (screenshots, logs)
5. **Document** — combined document+lint housekeeping pass; checks docs are complete
6. **Lint** — consumes the lint half of the housekeeping pass; runs repo's lint command
7. **Push** — force-pushes to the configured target (origin or fork)
8. **PR** — creates PR/MR via SCM adapter (GitHub, GitLab, Azure DevOps, Bitbucket)
9. **CI** — monitors CI checks, auto-rebases on base branch advancement, auto-fixes CI failures

### TUI Architecture (`internal/tui/`)

Bubble Tea terminal interface:
- Layout: findings list panel + diff view + action bar + log box
- Three modes: compact, wide, and wizard
- Approval actions: approve (a), fix (f), skip (s), abort (ctrl+c)
- Diff view for fix review — shows what the agent changed
- CI monitoring with live status display
- Terminal title updates with run status

### Git Hook Pipeline (`internal/git/hook.go`)

The post-receive hook is a shell script that:
- Resolves the bare repo directory absolutely (defense against `pwd` collapsing to ".")
- Passes each ref update to `no-mistakes daemon notify-push` with ref/old/new SHAs
- Respects `GIT_PUSH_OPTION_*` for git push options
- Never blocks the push — failures are logged to stderr and `notify-push.log`
- Shows ASCII banner on success: "Pipeline started. Run no-mistakes to review."

### Intent Extraction (`internal/intent/`)

When pushing a branch, `no-mistakes` can read recent transcripts from the user's local coding agent (Claude Code, Codex, OpenCode, Rovo Dev, Pi, Copilot CLI), identify the session that produced the change, and summarize the user's intent. This intent is fed into review, test, document, lint, and PR prompts so agents have context beyond just the diff.

### Self-Update (`internal/update/`)

Downloads GitHub release archives, extracts to a versioned directory, and replaces the running binary atomically via `os.Rename` (same-filesystem, always atomic on Unix). macOS quarantine attributes are cleared. A background daemon checks for updates periodically.

### Testing Strategy

- Unit tests: `go test -race ./...` (excludes e2e)
- E2E tests: behind `e2e` build tag, drives real binary against fake agents through full push→pipeline→push journey
- Recorded fixtures: `cmd/recordfixture/` captures real Claude/Codex/OpenCode CLI output as e2e fixtures (spends API quota)
- Fake agents: `cmd/fakeagent/` implements JSONL wire protocols for Claude, Codex, and OpenCode for deterministic testing
- Workflow tests: top-level `workflow_*_test.go` files test CI, release, docs, and no-mistakes-required workflows

### Key Dependencies

- `github.com/charmbracelet/bubbletea` + `bubbles` + `lipgloss` — TUI framework
- `github.com/spf13/cobra` — CLI command tree
- `gopkg.in/yaml.v3` — config parsing
- `modernc.org/sqlite` — embedded SQLite (no CGo)
- `github.com/toon-format/toon-go` — TOON protocol for `axi` (agent-to-approval) interface
- `golang.org/x/sys` — platform-specific syscalls

## Key Techniques

### 1. Git Worktree Isolation

The pipeline runs in a disposable worktree created from the bare repo. This is the central design insight: your working directory stays untouched while AI agents review, test, lint, fix, and push. The worktree is also defended against shared config poisoning — `IsolateHooksPath` pins `core.hookspath` per-worktree so tools like husky can't disable the gate's post-receive hook from inside a linked worktree.

### 2. Structured Output with Multi-Format Parsing

Rather than relying on JSON schema alone, the agent output parser (`parseStructuredTextOutput`) implements a three-tier extraction strategy:
1. Direct JSON parse with schema validation
2. Code-fence extraction: finds ` ```json ... ``` ` blocks (handles fences glued to preceding text, a pattern real models produce)
3. Last bare JSON object: scans text for balanced `{...}` objects, returns the last valid one
4. Fallback: surfaces the original parse error with a 200-char output snippet for debugging

This matters because real model output is messier than the API contract suggests — models emit reasoning prose before JSON, embed JSON in fences, or produce multiple JSON objects.

### 3. Finding Fingerprint Deduplication

Findings are deduplicated using structural fingerprinting (`findingKey` and `findingFingerprint`):
- `findingKey`: strips ID, action, source, user instructions — matches by content
- `findingFingerprint`: further strips line number — matches by description+file+severity
- Used in merge, remove, retain, and exclude operations
- Also counts fingerprints to handle the case where the same finding appears multiple times legitimately

### 4. Trust Boundary via Config Filesystem

`EffectiveRepoConfig` enforces that code-executing fields (`commands`, `agent`) in `.no-mistakes.yaml` are only honored from the trusted default-branch copy. The pushed branch's copy controls only non-executing fields (`ignore_patterns`, `auto_fix` limits). This is a defense-in-depth supply-chain control: a contributor cannot inject shell commands by adding them to `.no-mistakes.yaml` on their feature branch.

### 5. Durable Approval Parking

When a step needs human approval, the executor:
1. Sets a mutex-guarded `waiting` flag and `waitingStep` name
2. Persists `StepStatusAwaitingApproval` and the execution duration to SQLite
3. Emits an event that the TUI/axi interface receives
4. Blocks on a channel waiting for `Respond()`
5. On daemon restart: `Resume()` reconstructs the approval gate from SQLite and re-attaches

This means runs survive daemon crashes, restarts, and upgrades — the human never loses their approval context.

### 6. Agent Session Durability

The review loop maintains two durable sessions per run:
- Reviewer session: reused across all full reviews (initial + fix-review cycles)
- Fixer session: reused across all fix turns (agent applies fixes to working tree)
- Sessions are provider-specific: stored with the provider name, only resumed by matching adapters
- If resume fails, the session ID is dropped and a fresh session starts (never skips a turn)
- This is critical for cost and coherence: the reviewer accumulates context about the codebase across fix cycles

### 7. Auto-Fix with Configurable Limits

Each step type has a configurable auto-fix attempt limit (default: review=0, lint=3, test=3, document=3, ci=3, rebase=3). The review step defaults to 0 because auto-applying review fixes is the riskiest category — the user must explicitly enable it. Auto-fix only targets findings with `action: "auto-fix"`; `ask-user` findings always park.

### 8. Combined Document+Lint Pass

The document step performs both documentation review AND lint assessment in a single agent invocation. It stashes the lint findings in an in-memory `RunShared` object. The lint step consumes them via `TakeHousekeepingLint()` — a one-shot take that clears the value, so a fix round cannot consume stale findings. This avoids a second cold agent startup for lint.

### 9. CLI-to-Daemon IPC via Unix Socket

The CLI communicates with the daemon through a Unix domain socket (or named pipe on Windows). Events are typed: `EventRunStarted`, `EventStepCompleted`, `EventLogChunk`, etc. The TUI subscribes to the daemon's event stream and renders reactively. Events carry findings JSON, diffs, durations, and error messages.

### 10. ACP (Agent Communication Protocol) Support

Beyond the six natively-supported agents, `no-mistakes` supports `acp:<target>` agents via `acpx`, allowing any agent that speaks the ACP wire protocol to serve as the pipeline agent. The config accepts `acp_registry_overrides` to map target names to custom commands.

## Design Decisions

### Optimized for correct gating, not speed

The pipeline runs sequentially, not in parallel. Review, test, document, and lint could theoretically run concurrently against the same diff, but sequential execution means each step sees the fixed state from previous steps and the approval model is simpler. The trade-off is latency — a full pipeline run takes multiple agent invocations sequentially.

### Human-in-the-loop by default

Auto-fix is opt-in for the riskiest category (review=0). The findings model defaults unknown actions to `ask-user` (not `auto-fix`), closing a fail-open hole where unclassified findings would silently pass. The human decides what to fix, skip, or approve. This is the right call for a gate that's supposed to prevent mistakes.

### Agent-agnostic by design

Every agent backend implements the same `Agent` interface. Fallback chains let you configure `agent: [codex, claude]` and have the pipeline survive a single-agent outage. The ACP bridge extends this to any future agent. The cost is that prompt engineering must target the lowest common denominator — prompts that work well for Claude might need different wording for Codex or OpenCode.

### SQLite as the sole state store

All durable state (repos, runs, steps, rounds, agent sessions, invocations) lives in SQLite via `modernc.org/sqlite` (no CGo required). This is a pragmatic choice: zero setup, single file, works everywhere. The schema is versioned and initialized on first use. For a single-user desktop tool, this is the right call over Postgres or file-based state.

### Filesystem as config boundary

Repos ship `.no-mistakes.yaml` alongside their code. Global config lives in `~/.no-mistakes/config.yaml`. The trust boundary (default-branch copy overrides pushed-branch copy for executing fields) is enforced at merge time in `EffectiveRepoConfig()`. This is cleaner than requiring server-side config or a web dashboard.

### No daemon dependency for the CLI

The CLI can run without the daemon for `init` and `stats`. For pipeline operations, it connects to the daemon or starts one. The daemon is a background process managed via platform-native service managers (launchd, systemd, schtasks). This split means the CLI is fast for quick operations, while the daemon owns long-running pipeline execution.

### Worktree isolation, not containerization

Unlike tools that use Docker or VMs for isolation, `no-mistakes` uses git worktrees. This is simpler and faster (no image pull, no virtualization overhead) but provides weaker isolation — the pipeline steps execute on the host with the agent's permissions. The trade-off is acceptable for a developer-side gate (not a CI server processing untrusted code).

## Comparison Notes

### vs. CI/CD pipelines (GitHub Actions, Jenkins)

Traditional CI validates after the push. `no-mistakes` validates *before* the push — it's a pre-push gate, not a post-push check. CI catches problems after they're public; `no-mistakes` catches them before they leave your machine. The two are complementary: `no-mistakes` can babysit CI as its final step.

### vs. Code review tools (Gerrit, Phabricator, GitHub PR review)

Traditional code review is human-only. `no-mistakes` adds an AI reviewer as the first pass, then escalates findings to the human. Unlike Gerrit's rigid approval workflow, `no-mistakes` has a flexible fix loop where the AI can auto-fix safe issues and the human can approve/fix/skip the rest.

### vs. Agent harnesses (Bram, Fleet Supervisor)

While tools like [[Bram]] provide a worklist lifecycle with hash-verified gates, and [[Fleet Supervisor (sermakarevich)]] orchestrates parallel coding agents, `no-mistakes` is specifically a *push gate* — it triggers on `git push` and runs a fixed pipeline, not a general-purpose agent harness. It's complementary: you could use Bram or Fleet Supervisor to build the code, then `no-mistakes` to gate the push.

### vs. Linting tools (ESLint, ruff, golangci-lint)

Traditional linters are deterministic rule engines. `no-mistakes`'s lint step can *also* run the repo's deterministic lint command, but its primary lint check is AI-driven — an agent reviews the diff for style, convention, and documentation issues. This catches things rules can't (e.g., "this function needs a doc comment explaining the new edge case"). The trade-off is nondeterminism.

### vs. [[Guardrails and Feedback Loops]]

The wiki's guardrails synthesis argues "linters beat prompts." `no-mistakes` actually combines both: deterministic lint commands + AI-driven review. The auto-fix feedback loop (find→fix→re-review→approve) is a concrete implementation of the self-tightening feedback loop concept. The finding fingerprint deduplication ensures the same issue isn't reported twice across fix cycles.

### vs. [[Orchestrating AI Code Review at Scale]]

Cloudflare's system uses 7 specialized agents + a coordinator judge running server-side. `no-mistakes` uses a single agent per step running locally. The architecture is simpler but makes different trade-offs: Cloudflare gets parallel review dimensions; `no-mistakes` gets privacy (code never leaves your machine except to the agent's API) and simplicity (single binary, no infrastructure).

### vs. [[The Dark Factory is a DOT File]]

`no-mistakes`'s pipeline is a DOT-shaped DAG (though currently linear). The pipeline definition is code in `internal/pipeline/steps/`, not a config file — but the concept is the same: the pipeline structure IS the valuable artifact. The `auto_fix` limits and `commands` overrides in the YAML config are the parameterization layer.

### vs. [[cco]] (sandboxing)

`cco` sandboxes Claude Code at the OS level (Seatbelt/bubblewrap/Docker). `no-mistakes` could theoretically benefit from this — running its pipeline agent inside a sandbox — but currently trusts the agent's own safety mechanisms. The `--dangerously-skip-permissions` flag passed to Claude is the pragmatic choice for an unattended pipeline.

### vs. [[OpenCodeReview]]

Alibaba's OpenCodeReview shares the hybrid deterministic+agent architecture (tree-sitter analysis + LLM review). `no-mistakes` differs in that it's a full pipeline (not just review), it gates the push itself (not just comments on PRs), and its dual-threshold document+lint combined pass is a clever optimization OpenCodeReview doesn't attempt.

## Innovation Points

1. **The git proxy pattern**: using a git remote + post-receive hook as the trigger mechanism is elegant. No new CLI, no new workflow — just `git push no-mistakes` instead of `git push origin`. Every developer already knows how to push.

2. **Durable approval parking**: runs survive daemon restarts. The approval gate is reconstructed from SQLite, not lost on process death. This is the kind of reliability engineering that separates a toy from a tool.

3. **Findings fingerprint deduplication**: a small but important detail. Without it, fix→review cycles would accumulate duplicate findings, cluttering the TUI and wasting the human's attention.

4. **Config trust boundary**: `EffectiveRepoConfig` with per-field trust is a supply-chain security control implemented at the config layer, not the network layer. It's the right scope for a local-first tool.

5. **Combined document+lint pass**: running both in one agent invocation avoids a full second cold startup. The findings are tagged with `category` to route them to the correct gate. Small optimization with real latency impact.

---

*Ingested 2026-07-11. Full source at https://github.com/kunchenguid/no-mistakes*
