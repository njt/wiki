# RADAR — Meta's Risk-Aware Diff Auto Review

Meta's production system for automating review and landing of low-risk diffs: 535K+ diffs reviewed, 331K+ landed at 25K/day, with a revert rate 1/3 and a production-incident rate 1/50 of non-RADAR diffs — and the argument that agent-generated code volume has made stratified review automation mandatory, not optional.

---

## What it is

RADAR (Risk Aware Diff Auto Review) is Meta's answer to a bottleneck its own AI tooling created. Over one year, significant lines of code per human-landed diff grew 105.9% and diffs per developer rose 51%, with 80%+ of the growth attributed to agentic AI — while timely review declined and some large groups accumulated thousands of pending reviews. The paper's framing is blunt: "a workflow that relies on humans reading most diffs does not scale."

RADAR is a funnel, not a model. Each diff is classified by authorship and automation source, then routed to a distinct path:

- **Deterministic codemods** → Blanket AutoAccept: the transformation is vetted at the codemod level, so diffs bypass per-diff AI review entirely.
- **AI-generated codemods and RACER runbooks** → the ACE pipeline: static safety checks (no OSS, no SOX-scoped code, no blocklisted paths) → Diff Risk Score threshold → LLM-based Automated Code Review requiring ≥8/10 confidence and zero risk signals.
- **Human-authored diffs** → RADAR Verification (ship with deferred review) and RADAR Approval (waive even the deferred review).

The Diff Risk Score — an ML model predicting production incidents, already powering ~20 risk features at Meta — is the calibration lever. Thresholds are percentiles (P5 = only the safest 5% qualify), configured per organization, per source type, and per RACER runbook. Runbook granularity is the paper's distinctive design claim: each runbook is judged on 60-day risk history (zero PIs, low reverts, minimum volume), daily landing caps (10–2,000 diffs/day), and a denylist. Two runbooks with identical transformations get different treatment if one has a revert history.

## Key quotes

> "a workflow that relies on humans reading most diffs does not scale, and we need mechanisms that preserve rigor while shifting scarce human attention toward changes where judgment and accountability are most beneficial."

The problem statement in one sentence — automation targeted not at review quality but at *reviewer triage*. This is the inversion: risk models historically surfaced risky diffs to reviewers; RADAR uses them to certify diffs that need no reviewer at all.

> "All PIs were manually reviewed by domain experts and none were deemed to have been detected by a human reviewer."

Buried in a footnote and quietly the most consequential claim in the paper. If no production incident in the cohort was caught by a human reviewer anyway, the human review being "replaced" was, for these diffs, pure ceremony — the strongest empirical support yet for [[The End of Code Review]]'s rubber-stamp thesis, from inside the largest codebase in the world.

> "the trade-off between review efficiency and the knowledge diffusion that review provides."

The authors name the cost every other review-automation write-up omits: code review is knowledge transfer, and each automated review deletes a channel for it. They claim the balance is currently favorable because RADAR-reviewed diffs are "qualitatively different from the typical human-reviewed diff" — routine changes nobody learned from anyway — but concede this may shift as coverage expands.

> "Because diffs are not randomly assigned to RADAR, unobserved confounding can influence observed changes."

The threats-to-validity section is unusually honest: observational designs, no full model specification, no diff-level data released, and the caveat that as RADAR absorbs low-risk volume it changes queue dynamics for everything else, so some measured gains include second-order effects.

## Themes

#pattern — risk-stratified funnels: layered eligibility (authorship → source → history → per-diff AI review) where each layer is cheaper than the next, and the cheap layers gate access to the expensive one. Deterministic vetting where possible, probabilistic where necessary, human where cheapest — the same three-tier rule as [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]], deployed at 25K diffs/day.

#concept — calibration as an operational lever: the P25→P50 relaxation raised the approve rate to 60.31% with no measurable safety degradation, suggesting conservative thresholds can be ratcheted open as confidence accrues.

#tool — the RADAR/ACR/DRS/RACER stack, with RACER as the agentic diff generator (3,000 diffs/week, 59% landing without human revisions) whose runbooks feed the pipeline.

## Analysis

This is the most important production evidence in the review-automation conversation, and it is more interesting as an engineering artifact than as an evaluation. The numbers are strong but observational — no randomization, unreported model specs, and a difference-in-differences analysis whose assumptions the authors decline to fully specify. The 330% time-to-close reduction is a median comparison of self-selected populations: diffs that opt into (or are eligible for) automated review are plausibly the trivial ones that humans would also have rubber-stamped quickly. The genuinely striking number is the PI rate — 1/50th of non-RADAR diffs — but the footnote undercuts the obvious reading: with zero incidents caught by human reviewers in the comparison population, the true baseline for "what human review catches here" may be near zero, making RADAR's safety advantage partly a statement about review's null contribution on this class of diffs.

The design lesson outruns the statistics. RADAR's real contribution is the eligibility model: differentiated trust per automation source, per-runbook risk histories with earned daily limits, and organizational risk appetite as explicit configuration. That is a template any large org can copy, and it answers the question [[Orchestrating AI Code Review at Scale]] (Cloudflare's seven-specialist review system) leaves open — how to *triage* what enters review agents at all rather than how to review better once inside. It also complicates [[The End of Code Review]]: Meta did not end review; it built the stratification machinery that ends review for one slice of the diff population while (implicitly) concentrating human attention on the rest. And it sharpens [[Agentic Code Review]]'s bottleneck thesis with the first at-scale numbers: the agents broke review, and now the same infrastructure — LLMs — is being applied to unbreak it, tier by tier.

The knowledge-transfer caveat deserves more weight than the paper gives it. If review is also how engineers learn a codebase, systematically removing it for the low-risk tail removes it precisely for the diffs juniors touch most. Meta can afford that trade; organizations copying the funnel template may not notice what they lost until their senior bench thins.

---
*Sources: [[raw/2605-30208]], [[summary/2605-30208]]*
*Last updated: 2026-10-11*
