# StarSling Skills — CI Audit Skills for Agents

A repository of four installable agent skills (`ci-speedup`, `ci-score`, `ci-secure`, `sling`) that turn coding agents into GitHub Actions auditors: measured CI speed diagnosis, a best-practice score, a closed-set security scan, and a control-plane CLI driver. What makes it notable is not the auditing itself but the engineering discipline around it — deterministic Python engines beneath the SKILL.md prose, a single sanctioned LLM step, machine-readable contracts for everything an agent might get wrong, and a CI gate whose own code treats its trust boundaries as a security threat model.

---

## Architecture

Four skill directories under `skills/`, each self-contained (`SKILL.md`, `scripts/`, `references/`, `tests/`, `evals/`), plus repo-level `tests/` guarding the install surface and privacy of examples:

- **`skills/ci-speedup/`** — the heavyweight. `scripts/collect_runs.py` (~34K lines) samples `gh` run history, resolves the required-check set via branch protection/rulesets, and routes the structural/critical-path track; `blocking_path.py` (~18K lines) renders the report as a measured wall-clock critical path ("long poles" drilled to root-cause steps); `scan.py` (~10K lines) runs static detectors parsed out of the catalog.
- **`skills/ci-secure/`** — `scan.py` (~8K lines) parses METADATA blocks from `references/security-patterns.md` and applies ten detector types (`regex`, `yaml-path`, `yaml-run-injection`, `yaml-job-correlated`, one network-gated `gh-impostor-sha`, `manual`, …) against workflow YAML. `scaffold/gate.py` is a stdlib-only CI check vendored into the audited repo.
- **`skills/ci-score/`** — the pure-function scorer: `references/ci-score-spec.json` (frozen v0.1.8) defines eleven pass/fail configuration facts and the formula; the scorer is "a pure function of (findings.json minus the stamp, this spec.json): no network, no clock, no LLM."
- **`skills/sling/`** — thin: `SKILL.md` routes requests between the `sling` CLI (reads), `gh` (state changes), and the three audit skills; `references/command-surface.json` is a machine-captured list of all 13 subcommands a test uses to prove the skill never routes to a subcommand that doesn't exist.

## Key techniques

- **No LLM in measurement.** Detection, ranking, and every measured number are deterministic; findings JSON + report are reproducible. The one LLM step is phase 4a "gap-fill" — when a drilled pole's log matches no catalog detector, the agent writes a *labelled* root-cause reading that never touches detection, ranking, or magnitudes. Irreducibly-semantic patterns (OPT13, OPT15) are demoted to a manual-review checklist: "omit rather than fake."
- **Catalog as executable spec.** Each of the ~99 `references/patterns/*.md` files carries a METADATA block (pattern id, detector type, affected files, fix strategy); the scanner parses it and each detector operationalizes the entry's *Anti-pattern + Detection heuristic*. Catalog entries with no registered detector are reported honestly in `catalog_patterns_without_detector`.
- **Two-axis measurement.** Developer wall-clock wait (the merge-gating critical path, the ranking axis) is kept strictly separate from runner-minutes (the bill); "wall-clock-neutral" runner-minute findings need a source-backed certificate to promote, everything else demotes to "Also noticed."
- **The gate trust split.** `gate.py` resolves the SCAN ROOT (attacker-writable on fork PRs — input only) separately from the ENGINE and RULE (resolved relative to the gate file, never the scan root), because "a gate that loaded its engine from the tree it was auditing would be running the code it is supposed to be judging." A check that could not run is reported as *did not run*, never a pass — crash-tolerant degraded shapes (empty facts list) are explicitly named and turned red.
- **Agent-UX engineering as first-class code.** The interaction contract (structured questions via `AskUserQuestion`/Codex `request_user_input`, fixed option order, verbatim fallbacks) is spelled to the keystroke, with live recorded failures ("sandboxed agent shells can't reach keyring creds"; "an experimental tool can silently fail to render — live Codex run 2026-10-29") baked into the text as regression lessons.
- **Offline replay testing.** ~100 recorded `gh` API response fixtures and synthetic e2e repos with `expected.json` verdicts make a network-dependent skill fully testable in CI.

## Design decisions

- **Diagnose, don't prescribe.** Findings ship a ready-to-paste agent prompt rather than a fix; the rationale is that detection plus run history is accurate while fixes need file intent, real logs, and load-bearing context. Structural fixes carry mandatory `risk`/`guardrail`/`rollout` fields — the render boundary *rejects* a structural finding missing them — and risk never demotes the rank.
- **Vendored, not fetched.** The ci-secure CI check is copied into the target repository so "the code that judges a pull request [is] readable and stable," shipping advisory-mode first (`--advisory` on by default; removing it makes the check blocking).
- **Explicit non-claims.** ci-score refuses to be a security audit; ci-secure refuses to be comprehensive or produce a score, restricting to a closed set of exploit chains with "a real incident behind each one" (`why-these-ten.md`). The refusal is the product.
- **Honest degradation everywhere**: skipped API checks recorded in `gh_checks`, uncovered poles get a labelled gap-fill rather than silence, partial checkouts are disclosed, and the score refuses outside a full checkout because partial views "silently inflate scores."

## Comparison notes

- Same distribution channel as [[Agent Skills Library (dzhng)]] (`npx skills add`), but the opposite build philosophy: dzhng's skills are mostly prose, while these treat the SKILL.md as a thin orchestrator over ~89K lines of tested deterministic Python — with the LLM admitted only where judgment is genuinely needed and fenced off from the numbers.
- [[Win Dev Skills]] makes the same "prose is Tier 3, tools are Tier 1" argument with in-repo ground-truth tools (a metadata indexer, a Roslyn analyzer); StarSling Skills is that doctrine applied to auditing, with evals and replay fixtures instead of build-time analyzers.
- [[Claude Code Skills System]] describes the platform mechanics skills ride on; this repo pushes the format's envelope — machine-readable spec JSON, command-surface tests, and evals per skill — and its Codex-compatible interaction contract shows the skill format being treated as harness-agnostic by design.
- Against [[harnesseval — Harness vs Prompt for AI Code Review]]'s finding that harnesses beat bare prompts, this is a data point from the other direction: determinism beats the harness for measurement, and the harness (the SKILL.md loop) is reserved for glue and the one gap-fill judgment call.
- The repo's own content marketing, [[CI Speedup — Linear's CI Optimizations as Agent Prompts]], is the pedagogical face of the same design philosophy: the same 90+ pattern catalog and critical-path thinking, repackaged as copyable agent prompts rather than a scanner.

---
*Sources: [[raw/skills-github-2]], [[summary/skills-github-2]]*
*Last updated: 2026-11-20*
