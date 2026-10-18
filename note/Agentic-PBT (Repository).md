# Agentic-PBT (Repository)

The code behind Anthropic's agentic property-based testing research ([[Finding Bugs with Property-Based Testing]]): a Claude Code slash command that mines properties from a target codebase, tests them with Hypothesis, and files rubric-triaged bug reports — plus a 507-line stdlib-only Python harness that ran it across 100+ Python packages in parallel sandboxes. The repo is the cleanest public example of a "prompt-as-the-product" agent: almost all of the engineering intelligence lives in one markdown file, and the surrounding code exists only to sandbox, parallelise, and resume.

---

This is the implementation companion to the [[Finding Bugs with Property-Based Testing]] note, which covers the paper's results (984 candidate reports, 86% validity on top-ranked items, merged fixes in NumPy and Hugging Face tokenizers). This note covers what the code actually looks like — and the answer is surprising: **~780 lines of Python total, of which the agent itself is 0**. The agent is `hypo.md`, a 300-line English document installed as a Claude Code command.

## Architecture

Three components, deliberately thin:

- **The agent** — `hypo.md`. A single slash command (`.claude/commands/hypo.md`) defining a seven-step todo-driven loop: analyze target → understand via introspection → propose properties → write Hypothesis tests → run and triage → report or conclude. Invoked as `/hypo numpy` or `/hypo statistics.median`; no argument means the whole working directory.
- **The runner** — `run.py` (507 lines). A `ThreadPoolExecutor` harness that per-package creates a `venv` (one shared `stdlib_env`, one per PyPI package, wheels-preferred installs), assigns each module to one of N worker directories, and invokes `claude -p "/hypo <module>"` headlessly with a fixed permission allowlist. Default 20 workers, 10 preinstall workers, 1-hour timeout.
- **The scorer** — `scoring.py` (274 lines). Post-hoc: reads every `bug_report*.md` under `results/`, sends each to the Claude API (not Claude Code) with a 15-point rubric, regex-parses dimension scores out of prose, writes a sorted CSV. Async with a semaphore; supports `--retry-failures` by re-reading its own CSV.

Output layout per package: `bug_reports/`, `logs/claude_call_<id>.json`, `aux_files/<id>/` (every file the agent wrote, preserved), and `call_mappings.jsonl` joining call IDs to modules and reports. Cost and wall-clock per call are extracted from the Claude Code JSON logs.

## Key techniques

- **Evidence-gated property mining.** The prompt's strongest instruction: *"Only test properties that the code is explicitly claiming to have"* — from docstrings, comments, or call sites — and exit cleanly ("No testable properties found") otherwise. The agent is told to follow import chains into private implementation modules (`numpy.linalg._linalg`) and to check *callers'* implicit assumptions, because an internal helper's undocumented preconditions are the most common source of false positives.
- **A taxonomy of property patterns** baked into the prompt: invariants, round-trips, inverse operations, multiple implementations (fast vs reference), idempotence/commutativity, confluence (compiler passes), metamorphic relations (`sin(π − x) = sin(x)`), and — for parsers — bare "single entry point doesn't crash on valid input."
- **Soundness over completeness.** When a strategy can't cover the full input domain, prefer sound-but-incomplete: *"Do not chase completeness: 90% is good enough."* This is a rare explicit anti-goal in agent prompts, and it's the right one for bug-finding, where an unsound test generates noise and an incomplete one merely misses bugs.
- **Triage as a three-step rubric inside the agent**, before anything is written: reproducibility (minimal standalone script), legitimacy (is the input realistic? do callers prevent it? is the property actually claimed?), impact. Failure loops back to strategy refinement via `assume()` and `filter()` — the prompt explicitly engineers against the classic agent failure of filing junk reports.
- **Permission-sandboxed fleet execution.** `call_claude()` invokes Claude Code with `--allowedTools "Bash(python:*)" "Bash(pytest:*)" "Edit(*.py)" "Edit(*.md)" "Write(*.py)" "Write(*.md)" Todo WebFetch` plus `--add-dir <venv>` so the agent can *read* package source but only *write* in its worker sandbox. Worker directories are cleaned between runs except for `.claude/`.
- **Resumability without a database.** `get_completed_modules()` replays `call_mappings.jsonl`, keeps the latest attempt per module, and checks its log for `type == "result"` with no error — so re-running the runner skips completed work. Append-only JSONL as a state machine.
- **Prompt-defined CLI contract for sub-agents.** `utils/check-issue-exists.md` is a second Claude Code command whose entire output contract is one line — `issue:<n>`, `pr:<n>`, or `None` — with instructions to wait out `gh` rate limits rather than give up. Structured output from an agent, enforced purely by prompt.

## Design decisions

- **Prompt over code.** No agent SDK, no framework, no orchestration library. The trade: zero code to drift out of date with the harness API (the command works with whatever Claude Code does), at the cost of any guarantees beyond what the permission allowlist and prompt enforce. The `marketplace.json` shows it's also packaged as a Claude Code plugin.
- **Sandboxing is advisory, not defensive.** The README is candid: "You should still be careful with the runner, because running arbitrary code is dangerous!" The allowlist blocks most tools but the agent runs `python`, which executes target-package code. This is containment against *accidents*, not against a malicious package — a deliberate cost cut, since testing 10,000 PyPI packages requires installing and running them regardless.
- **Model asymmetry: strongest model for both jobs.** Opus for bug-hunting *and* Opus 4.1 for scoring. The scoring rubric is opinionated about what makes a bug *worth reporting* — "maintainer defensibility" scores how hard it would be to dismiss the report, not how severe the bug is. That's an optimisation for upstream acceptance, not for correctness.
- **Transparency by artifact retention.** Every aux file, log, and failed report is kept (the paper directory even ships the *worse* initial rubric "for full transparency"). The reproducibility stack — frozen `paper/` results, hardcoded + random-sampled package selection scripts — is unusually thorough for an agent-evaluation paper.

## Comparison notes

- **[[Finding Bugs with Property-Based Testing]]** is the results-side companion note covering the same project's paper; this repo adds the harness engineering that paper abstracts away — notably that the "agent" is a 300-line prompt and the evaluation infrastructure is arguably more code than the agent.
- Unlike [[Agentic Testing]] (Slack's CLI-vs-MCP comparison, where tool transport dominated results), this project pins the tool interface — plain Claude Code with bash — and puts all variation into the prompt. It is evidence that a well-engineered single-file prompt can rival elaborate harnesses.
- It inverts the direction of [[Agentic Verification]] (Osmani): there, properties constrain *code the agent wrote*; here, properties mined from existing code are the *product* — the loop's output is a bug report, not a fix.
- The per-worker-sandbox, allowlisted-permissions fleet pattern is a minimal cousin of the sandbox platforms surveyed in [[Best Infrastructure Platforms for Coding Agents in 2026]] — here implemented with venv directories and `--allowedTools` strings instead of Firecracker VMs, which is either admirably frugal or fragile depending on your threat model.
- The rubric-scored output pipeline (agent produces reports, second model pass ranks them) parallels the two-stage generation-then-triage stack in [[A Serious AI Product]]'s verification-first demands, at research scale.

#tool #project #agents #testing #python

---
*Sources: [[raw/agentic-pbt]], [[summary/agentic-pbt]]*
*Last updated: 2026-10-18*
