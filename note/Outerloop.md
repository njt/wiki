# Outerloop — Autoresearch Agents That Improve Your Benchmark

Outerloop (outerloop-science/outerloop) is an open-source kernel from NYU's Agentic Learning AI Lab that turns a benchmark-bearing research repo into a self-improving loop: agents propose changes, the kernel measures them against the base tree at the same seed, judge agents review before any PR exists, and every attempt — including failures — ends in a written report. What makes it notable is not the agent but the governance: ~22k lines of kernel code (44k lines of Python total) exist mostly to make the agent untrusted — scope checks, credential separation, budget arithmetic, mechanical verdict merging — while the interesting research decisions stay with the model.

---

## Architecture

The shape is a **kernel-and-roles** system, not a monolithic agent:

- **The contract** (`src/outerloop/contract.py`, 668 lines) is the source of authority. A target repo opts in with `.outerloop.yaml`: benchmarks (command + metric + direction), budgets, allowed write paths, roadmap. The loader parses contracts as *untrusted input* — it rejects YAML aliases, duplicate keys, and documents over 64 KB — and hard-codes invariants no YAML can override: `.github/`, the contract file itself, and the roadmap are always forbidden writes, and outerloop is never a valid target of itself.
- **Roles are data** (`src/outerloop/roles.py`): each role (author, review judges, verifier, planner, steward) is a `RoleSpec` — instructions, tool allowlist, session budget, and for judges an output schema. Adding a role is a new factory plus a result-policy, no kernel change. Crucially, "roles differ by prompt, not by tool posture": judges get Read/Grep/Bash plus web tools, the author gets the editing set.
- **The harness seam** (`src/outerloop/harness.py`, 1926 lines): every LLM touchpoint goes through one `Harness.run` — brief text in, `SessionResult` out. Claude Code, Codex, and hermes-agent are wired; provider quirks live in adapters, and context policy lives in `brief` shared by all backends so comparisons stay honest. Sessions run in a scrubbed environment: explicit allowlist, HOME redirected per-run, exactly one API key.
- **The tick** (`src/outerloop/tick.py`, 3994 lines) is the scheduler: a self-resubmitting Slurm `sbatch` chain (scrontab is disabled on their cluster), each tick acquiring a lease by **compare-and-swap commit to a `state` git branch** — a non-fast-forward push means another tick is alive, so exit. Task queue, leases, pause sentinel, and run ledger all live on that one unprotected branch. The bot already has write there; git IS the distributed state store. A watchdog is alert-only: restart is a human action, the lease makes it safe.
- **The climb** (`src/outerloop/attempt.py`, 6539 lines; `orchestrator.py`, 3011 lines): one invocation = one run = at most one PR. Clone → attempt → veto-checked commit → push → PR → report.
- **Containment** (`src/outerloop/containment/`): on macOS without a container, sessions run under a Seatbelt profile (write only workspace + scratch, read only toolchain paths); the README is honest that on Linux without a container a session can read anything you can — including key files — so the defense there is the credential split, not filesystem fencing.

## Key techniques

- **The syscall pattern.** Every agent talks to the kernel through exactly one tool (`syscall_cli.py`, installed as `.outerloop/syscall`); a syscall is typed and dispatched by type. The author's `sleep` syscall — write the ABI file and end the session — IS the park: the kernel submits real GPU launches as jailed jobs on a sealed snapshot, then wakes the *same session* with results delivered as data. The `.outerloop/` channel is kernel-excluded from the diff via `.git/info/exclude` so requests never pollute the candidate or the scope check. Readers validate rather than trust, because the channel is agent-controlled once dropped.
- **Measurement as a pure function** (`src/outerloop/measure.py`): the session can't be re-run on wake (it already made its edits), but measure-and-decide after the candidate is committed is a pure function of committed trees + contract — so it's cacheable by identity and re-runs idempotently after process death. `MeasurementPending` as a park-and-resume exception across Slurm jobs is the trick that makes the whole loop resumable.
- **Mechanical verdict merging** (`src/outerloop/panel.py`): judge lenses (integrity `verify`, code `review`) run as agent sessions, and their verdicts are merged *mechanically* — any blocking finding wakes the author; the kernel never adjudicates judgment. "Silence is never endorsement," and every lens outcome including "could not run" lands in the transcript the PR carries. No output schema for the author on purpose: the artifact is the diff plus a free-text report, judged by measurement, not parsing.
- **Scope is enforced, not sandboxed.** The agent's whole workspace is writable, so there's no per-path sandbox — instead the kernel rejects a candidate whose changes fall outside the contract's allowed paths (comparing the full candidate tree against its merge-base, so base-branch edits don't count as author edits).
- **Budgets in arithmetic, warnings in prompts** (`src/outerloop/syscall.py`): launches metered by `depth_k`, sleeps by `sleep_k`, enforced arithmetically in the kernel; the prompt carries only warnings. Deployment is pull-at-tick-start ordered so a bad merge crashes the tick but not the chain.

## Design decisions

- **Untrusted-agent posture throughout.** The contract parser hardens YAML; guidance text is read from git objects at the contract ref, never the working tree, with symlinks refused; PR text, diffs, job output are rendered as fenced data. The README candidly admits injection hardening "lowers the risk… without guaranteeing it."
- **Credential separation is the real security model.** Sessions see one capped API key; the bot PAT is read orchestrator-side after the session ends; the judge's session job holds a read-scoped token while the write token lives in a separate post job with no session next to it. The architecture doc's honesty about Linux's weak spot ("read denial would be theater anyway") is refreshing.
- **Keep the ruler out of the pen.** The contract guidance explicitly instructs putting eval code outside `scope.allowed` so a doctored benchmark can't survive into a measured candidate — an attack surface most autoresearch demos simply ignore.
- **All connections outbound.** Recovery needing inbound access is a human with a runbook; 2FA problems are dissolved by direction, not solved.
- **Humans keep the merge button** by default; `merge: auto` is an explicit opt-in.

## Comparison notes

- Where [[A New Era for Software Testing]] treats agentic QA as a compensation for lower-quality AI code, Outerloop goes further: the eval IS the loop's authority, and judges exist to stop the agent gaming it.
- It's the institutional-scale sibling of [[Automating Myself Out of Development]] — both move from interactive sessions to a background loop with human review as the bottleneck, but Outerloop adds budgets, leases, and measurement discipline for the multi-week GPU case.
- Against [[The Harness Is the Company]]'s claim that owning the outer loop is the moat, Outerloop is a concrete instance: a business whose outer loop *is* a separate product, and one that deliberately refuses to be the target of itself.
- Its measured-candidate-before-PR discipline contrasts with [[FrontierCode]], which benchmarks whether PRs would be accepted — Outerloop instead makes acceptance a property of the measurement and review pipeline, not a prediction problem.

---
*Sources: [[raw/outerloop]], [[summary/outerloop]]*
*Last updated: 2026-11-02*
