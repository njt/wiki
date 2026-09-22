# Code Standards That Survive the Next Agent

Adam Bertram (Telerik/Progress) argues against standardizing which AI coding tool a team approves and for standardizing what every change must prove before merge: AGENTS.md for guidance an agent must interpret, CI checks for rules a machine can block on, and pull-request evidence for the judgments only humans can make. The diagnosis underneath is that AI broke the informal proxies reviewers priced in — polish, diff shape, test presence — so review has to run on receipts.

---

## What it argues

The setup is two PRs reaching the same service, one written with Cursor and one with Claude Code, both formatted, summarized, and green — and neither demonstrably following the architecture rules, reusing approved helpers, or proving the behavior it changed. Banning all tools but one looks like the easy fix, but "the approved tool will change." The durable object is the receipt, not the toolchain.

The diagnosis is the most interesting part. Reviewers never actually read polish as polish; they read it as a *proxy* for something else. Bertram names the three: "Formatting was a proxy for effort, a focused diff was a proxy for scope control and tests were a proxy for the developer understanding the change." AI-generated code keeps the signals and deletes what they signaled — an agent produces formatted, confident code that still calls the wrong API or duplicates an existing helper. The proxies didn't fail; they stopped carrying information.

Two failure modes follow, on different clocks: debugging overhead and incidents now (Stack Overflow 2025: 84% using or planning to use AI, 46% distrusting the output, 45% reporting more debugging time; Veracode spring 2026: >95% syntax correctness, ~55% passing security testing), and maintainability debt quietly later — GitClear's 623M-change dataset shows moved code collapsing from 21% of changed lines (2022) to 3.8% (2026) while copy-paste rose to 15.7%. "Five copies of a pricing rule look productive until the rule changes and a developer updates only three."

The prescription is one control split three ways by enforceability:

- **AGENTS.md** holds what an agent must *interpret* — required commands, module boundaries, approved helpers, protected files. Fleet-level template owned by the platform team (security rules, dependency policy, required evidence); per-repo additions owned by the service team; a named owner who removes outdated rules, not just adds new ones.
- **CI** holds what a machine can *enforce* — formatting, strict type checks, tests, changed-lines coverage, dependency allowlists, security scans. One extra safeguard: "Require every meaningful new test to fail against the pre-change code," so an agent can't raise coverage without proving behavior. Post-merge, the CI evidence doubles as the timestamped audit trail NIST's SSDF expects.
- **The PR template** holds *evidence* for the judgments CI can't encode. "An AI-written description saying all tests pass is only a claim. When the execution evidence is missing, return the pull request before reviewing the diff."

The process-weight objection is answered with the same division: each control lives where it's cheapest, so reviewers are left with exactly two questions — does the change fit the architecture, and was the ticket the right change to make? Rollout advice is deliberately small: one high-traffic repository, three repeated review comments converted into CI checks, two sprints before expanding. The AGENTS.md format itself is now stewarded by the Linux Foundation's Agentic AI Foundation, which Bertram reads as the instruction layer becoming a durable standard every tool must survive.

## Key quotes

> "Don't try to control which AI coding tool developers use. Control what every code change is expected to demonstrate before it can be merged."

The central reframe, and the reason the piece is more than generic process advice. Tool standardization is a losing fight against developer preference and vendor churn; evidence standardization is a property of the review process, which survives every tool swap. "The tool can change. The receipt cannot."

> "Polish is no longer a useful indicator of quality. AI can make bad decisions look polished."

The cleanest statement of the broken-proxies thesis. What died isn't code quality but the *correlation* reviewers used to price risk — and that's a sharper diagnosis than "AI code is bad," because it explains why experienced reviewers are suddenly disoriented rather than merely annoyed.

> "A sentence in AGENTS.md can be misunderstood or ignored. A required CI check can block the merge."

The placement rule in one line: sort every standard by whether a machine can enforce it, then put it in the only bucket that matches. Most teams muddle exactly this — prose instructions carrying machine-checkable rules, or judgment calls hard-coded into CI — and this one-line test sorts them.

> "Require every meaningful new test to fail against the pre-change code."

The single sharpest concrete control in the piece: poor-man's mutation testing as a merge gate. An agent can raise the coverage number without exercising the change; a red run on the pre-change code is evidence the test bites. Bertram is honest about the residual — reviewers still confirm the failure is *for the intended reason* — which keeps this a reviewer aid rather than a full automation.

> "An AI-written description saying all tests pass is only a claim."

A quiet echo of Kenton Varda's moratorium on AI-written change descriptions, resolved differently: Bertram doesn't ban the descriptions, he demotes them — a description is a hypothesis until attached execution evidence makes it reviewable. Return the PR before reading the diff.

## Key themes

#concept — broken proxies: review heuristics as statistical signals that AI nullified
#pattern — the placement rule: interpretive → AGENTS.md, enforceable → CI, judgmental → PR evidence
#tool — AGENTS.md as governed infrastructure, not documentation
#project — Progress Forge, the vendor product this essay is a funnel for

## Opinionated take

Declare the conflict of interest first: this is a Telerik (Progress) blog post whose final third is an early-access pitch for Progress Forge, "designed for exactly that shift." An essay about evidence-before-merge ends with a product ask that requires no evidence — read the prescription on its merits and treat the product claims as marketing. That said, the core argument stands on cited data and it's a good one.

The real contribution is the sociological observation, not the three-pillar checklist (which is close to industry consensus by 2026). Reviewer intuition was always a Bayesian pricing of weak signals, and agentic code decoupled every signal from the thing it signaled. "The tool can change. The receipt cannot" is the right invariant to organize around, and it explains why tool bans and model upgrades both fail as policy: neither touches the evidence channel.

The placement rule is the durable takeaway, and it's falsifiable in daily use — every team fight about "should this be in CLAUDE.md or a hook?" is the same question, and "can a machine enforce it?" answers most of them.

Where it's weakest: the two questions left for human reviewers ("does it fit the architecture, was the ticket the right change to make") are waved through as if they were the light part, when they're the whole hard part — the article presumes review capacity that [[The End of Code Review]] argues is already indefensible at scale. And the pre-change-failure requirement, while excellent, has unmentioned costs: CI must run tests against the parent commit, and environmental false failures will train people to override the gate. Finally, "named owner who removes outdated rules" assumes the maintenance discipline that organizations famously fail at for every other doc; nothing here explains why AGENTS.md would escape that fate — though treating the file as runtime configuration, as [[Benchmarking AGENTS.md Changes]] does, is the stronger answer.

This piece is the skeleton; [[Engineering Standards Enforcement at Cloudflare]] is the organ. Bertram's three buckets map cleanly onto Cloudflare's Codex (interpretive RFCs → enforced MUSTs → reviewer agents), which makes the essay a useful on-ramp rather than a competitor.

## Related pages

- [[Engineering Standards Enforcement at Cloudflare]] — strengthens this source's skeleton with the fully built version: Cloudflare's Codex shows where Bertram's AGENTS.md/CI/PR-evidence triad lands when standards become machine-readable RFCs with a SHOULD→MUST lifecycle and reviewer agents blocking 16,000 merges.
- [[Benchmarking AGENTS.md Changes]] — nuances the guidance layer: Bertram assigns AGENTS.md a named owner who prunes stale rules, while Stet's iteration data shows even well-tended instruction files need empirical validation, since improvements can mask specific-task regressions (AGENTS.md inversion).
- [[Architectural Guardrails for AI-Generated Code]] — converges on the same thesis from the governance side: both argue that prose alone can't hold architectural intent and that enforcement belongs in deterministic CI with traceable verdicts; Bertram adds the PR-evidence bucket for exactly the judgments O'Reilly's essay routes into injected decision context.
- [[AI-Written Change Descriptions]] — complicates Varda's moratorium: where Varda bans AI-written descriptions outright, Bertram keeps them but demotes them to claims that only attached execution evidence makes reviewable — a reconciliation worth testing in practice.

---
*Sources: [[raw/ai-coding-standards-repeatable-workflows]], [[summary/ai-coding-standards-repeatable-workflows]]*
*Last updated: 2026-09-22*
