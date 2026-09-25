# Yak — Agent Platform for Papercuts, PR Review, and Preview Servers

Yak (github.com/Geocodio/yak, MIT, by Geocodio) is a production agent platform built on a deliberately boring stack: Laravel 13, Inertia 3, MariaDB, two Docker containers, and Incus system containers on a dedicated Hetzner server. It runs three workflows — autonomous papercut fixes (Slack/Linear/Sentry → sandboxed Claude Code → PR), line-level PR review with risk-gated approval, and per-branch OAuth-gated preview deployments — all sharing one sandbox fleet, one GitHub App, one dashboard. It matters because it is a real shop's whole agent harness, open-sourced, with a safety model and cost model spelled out in code rather than aspiration.

---

## Architecture

- **Workflows over substrate.** `docs/architecture.md` draws it explicitly: three user-facing workflows sit on a shared substrate (repos ⟷ GitHub App ⟷ ingress ⟷ auth, Incus+ZFS sandboxes, Laravel queues, dashboard/artifacts/costs). New workflows plug in without reshaping the base.
- **Channel drivers.** Three contracts in `app/Contracts/` — `InputDriver` (webhook → normalized task), `CIDriver` (build webhook → pass/fail), `NotificationDriver` (status updates). Channels activate by credential presence at boot; the same channel can fill multiple roles (Slack is input + notification). Routing back is by `source` column: respond where you were asked.
- **Task pipeline.** `app/Jobs/RunYakJob.php` creates the branch, runs Claude in the sandbox, and *Yak* (not Claude) pushes and later creates the PR via `CreatePullRequestJob`. CI runs asynchronously off the queue; a webhook (`ProcessCIResultJob`) triggers the next step.
- **Sandbox layer.** `app/Services/IncusSandboxManager.php` (1,148 lines) clones a ZFS CoW snapshot `yak-tpl-{repo}/ready` per task in ~2s. Each container gets its own Docker daemon (`security.nesting=true`), network namespace, and filesystem — no shared sockets, no DinD hacks. `SetupYakJob` builds the snapshot once per repo (clone, npm/composer install, compose up, snapshot).
- **State machine.** `app/Enums/TaskStatus.php` uses the `artisan-build/fat-enums` package: transitions declared as PHP attributes on enum cases (`#[CanTransitionTo([...])]`), enforced at the model — assigning an illegal status throws `InvalidStateTransition`. The rules live in the type, not in job code.

## Key techniques

- **Two-tier AI.** Routing/analysis (repo detection, Slack parsing, Sentry triage) runs on Haiku/Sonnet via the Anthropic API; implementation always runs Opus via Claude Code CLI. The rationale: better first-attempt results mean fewer retries and less total work than a cheap-model-then-escalate ladder.
- **Session resume as the biggest cost lever.** Retries and Slack clarification replies use `claude -p --resume $session_id` stored on the task row, so Claude keeps the codebase context from the first attempt. But the retry *prompt* is still self-contained — the sandbox is destroyed after push, so `YakPromptBuilder::retryPrompt()` pre-assembles the original task, previous attempt summary, and CI failure output for a fresh session.
- **Queue split as an anti-wedge.** `yak-claude` (4 workers, 600s timeout) holds only Claude jobs; webhook handling, PR creation, and notifications live on `default` (3 workers, 30s) so a 10-minute Opus run can never block CI-result processing.
- **Streaming runner hardening.** `app/Agents/SandboxedAgentRunner.php` (838 lines) wraps `incus exec` of `claude -p` with line-by-line stream parsing (`StreamEventHandler`), a 1200s idle timeout, a 15s post-result grace period (backgrounded `php artisan serve &` inheriting stdout can wedge the exec pipe indefinitely), capped malformed-line warnings, and a resume prompt for truncated streams.
- **Video pipeline with a lint-before-shoot gate.** The agent writes `script.json` (shots, captions, pauses); `yak-browser script` dry-runs it headlessly before capture burns time; `yak-browser shoot` captures per-beat clips; `RenderWalkthroughJob` renders a Remotion composition and passes it through `RenderQaCheck`, a frame-sampling gate for blank/garbled frames. Artifact *roles* (`script`, `manifest`, `shot`, `cut`, `thumbnail`…) are the pipeline's vocabulary, queried rather than filenames.
- **Review with a risk policy.** `ReviewRiskScorer.php` and `ReviewApprovalPolicy.php` implement shadow-mode risk-based approval: scores and confidence thresholds ship uncalibrated, run in shadow first, compare against human reviews, and never confer merge authority. Reviews are incremental on re-request (only commits since last review) and deliberately don't re-trigger on every push.
- **Findings double-filtering.** Path excludes are applied both before the prompt is built *and* to the parsed findings — a model that hallucinated a finding in `vendor/` is silently dropped.

## Design decisions

- **The safety boundary is the sandbox, not permission dialogs.** `--dangerously-skip-permissions` is always on; the walls are Incus namespace isolation on a dedicated server with no production access. This is the honest trade for unattended operation, and Yak says so plainly.
- **No merge authority, ever.** PRs are created, humans merge; the GitHub App must not be in the branch protection bypass list. Even the opt-in risk-based approval grants no merge power. "If you want to automate merging, don't use Yak."
- **Bounded everything.** Max two attempts, $5 per-task budget flag, $50 daily routing budget enforced by `EnsureDailyBudget` middleware, 200-LOC scope flag (`yak-large-change`), 15-minute hibernation and 30-day eviction for previews.
- **Boring stack, single server.** Not Kubernetes, not horizontally scaled; throughput scales with RAM (~4–8GB per sandbox). The docs' "What Yak Is Not" section is a model of scope discipline.
- **Cost accounting is a first-class table.** `daily_costs` keyed by date, `ai_usages`, and per-task `cost_usd`/`duration_ms`/`num_turns` — the platform watches its own spend.

## Comparison notes

- Compared to [[A Deep Dive on Agent Sandboxes]], Yak is a concrete instance of the sandbox-as-boundary pattern, choosing Incus system containers over gVisor/Firecracker-style microVMs so that repo Docker Compose environments work natively — heavier isolation than user separation, lighter than a VM per task, and ZFS CoW makes per-task clones nearly free.
- It differs from [[Agentic Code Review]] in shipping the full review loop, not just the reviewer: incremental re-reviews on GitHub's native re-request button, reaction polling as feedback, a findings dashboard, and a shadow-mode approval policy that treats its own confidence scores as uncalibrated heuristics.
- Against [[The Case Against Building Your Own Agent Platform]], Yak is the counterexample that argues *for* owning the platform when the substrate (sandboxes, channels, costs) is the differentiated part and the workflows on top are thin — though it is a two-engineer shop's tool, which is exactly that essay's favored condition.
- Its session-resume economics give a concrete implementation of [[Coding Agents Continuity Not Memory]]: continuity is a session ID plus a self-contained retry prompt, not a memory architecture, because the sandbox is disposable but the conversation is not.

---
*Sources: [[raw/yak]], [[summary/yak]]*
*Last updated: 2026-09-25*
