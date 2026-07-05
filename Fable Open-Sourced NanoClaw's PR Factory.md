# Fable Open-Sourced NanoClaw's PR Factory

Gavriel Cohen's overnight $800 ultracode session that took a 405-commits-stale fork of NanoClaw to open-sourceable, guided by nothing more than a set of customization guidelines. The most detailed public account of what unattended frontier-model agent workflows can actually produce — and a masterclass in how good specs (not good prompts) drive agent judgment.

---

## The Setup

Cohen pointed Claude Fable 5 at a private NanoClaw fork — the "PR Factory" that creates a Slack thread run by an agent for every pull request — and at NanoClaw's customization guidelines. He told it to reconcile the two with an ultracode workflow, and went to sleep.

The fork was 405 commits behind upstream, 8,000 lines of delta across 78 files, and stale beyond repair.

## What $800 Bought

Five unattended hours produced ~80% of the path to publishable. Six reader agents mapped the delta into five capabilities and 41 integration points with core, each classified and assigned a test archetype. The workflow:

- **Mutation-verified every guard test.** Each reach-in was deleted, the test was required to fail, then restored and required green. ~90 mutations across isolated worktrees.
- **Found a latent upstream bug.** The command-gate denial path opened the outbound DB read-only, so denials were never delivered. Sent to core.
- **Diagnosed an architectural gap.** Multi-bot support reached into twelve core files. Cohen asked why. The analysis decomposed the mess: 45% intrinsic (bot identity is a coordinate in six keyspaces), 30% retrofit tax, 15% separable policy, 10% avoidable choices. It also refuted Cohen's proposed alternative design by tracing it to a silent role-check failure.
- **Shipped a PR train.** Three bug fixes, three hook/registry PRs, one instance substrate (#2733). One PR rejected (an optimization with too much core surface) — Fable closed it, stripped dependencies, and moved on.
- **Self-reviewed adversarially.** About a third of raw review findings were refuted by verifier agents before reaching humans.

The remaining 20% took two more days: packaging as a recipe (a meta-skill that applies five component skills), then mechanically proving apply/idempotency/removal from a fresh upstream clone.

## The Contract That Made It Work

The critical enabler wasn't the model — it was the customization guidelines Cohen pointed it at:

> "The guidelines gave the agent a clear, checkable definition of done, and a decision rule for every piece of code it encountered."

The guidelines encode four rules:
1. **Mostly add.** Skills add files, append to barrels, add dependencies. Reach-ins are one or two lines.
2. **Test every integration point.** A guard test that goes red if wiring is deleted or drifts. When upstream moves something, you get a failing test that's your upgrade TODO.
3. **Removal ships with the skill.** A REMOVE.md that reverses everything apply did.
4. **Your fork is defined by a recipe.** One file listing skills in apply order. Rebuildable from scratch.

And the maintainer's side: when many skills reach into the same spot, add a proper hook so reach-ins become clean appends.

This is the specification-as-durable-artifact pattern: the guidelines outlived the fork, outlived the model that processed them, and produced staff-engineer-level judgment without being prompted to.

## The Human Contribution: Twelve Calls

> "Around a dozen calls. Every one of them was a decision: taste, scope, risk appetite. None of them was labor."

Cohen picked what went to core vs. the bundle, kept private tuning out, chose where recipes live, rejected one PR, ordered two features ripped out instead of fixed, declared single-repo scope, and said "yes, push."

This is the [[Optimizing for Decision Points]] thesis made concrete: the workflow surfaced taste-sensitive decisions, and the human made them. Everything else was agent labor.

## Critical Analysis

**This is the most important agentic-development dispatch of 2026 so far.** Not because of the $800 number — that's a headline — but because it demonstrates the complete loop from stale fork to upstreamable contribution with the human operating exclusively at the decision layer.

**The customization guidelines are doing the heavy lifting, not the model.** This is [[Specifications as the Product]] in action: the spec is the durable artifact, the code is disposable. Fable 5 went dark hours after finishing (export controls), but the guidelines-produced code stands on its own. Cohen could point any future model at the same guidelines and get comparable results.

**The mutation-testing self-correction is the sleeper detail.** One verification batch mislabeled its indices and silently skipped a guard — and the orchestrator caught the mismatch. This is the [[Guardrails and Feedback Loops]] pattern at the workflow level: deterministic checks on probabilistic output. The system caught its own error without human intervention.

**The "last 20%" is doing real epistemological work.** Going from "our fork is clean" to "strangers can run this" required mechanical proof: apply, idempotency, removal, all verified byte-identical from a fresh clone by an agent forbidden from seeing the reference implementation. That's not polish — it's the difference between "works on my machine" and "actually open source."

**The export-control coda is the ghost at the feast.** Fable 5 went dark on June 12, the same day Cohen pushed the final revision. The PR sat open and mergeable while the model was unavailable. The post itself was likely written with Fable after its return. The regulatory layer is now part of the engineering stack, whether we want it to be or not.

**The skills standard Cohen references is Anthropic's Agent Skills format.** Combined with the NanoClaw customization guidelines, it forms a two-layer contract: how to structure a skill (Anthropic's standard), and what a skill must do to be mergeable (NanoClaw's rules). This is [[Steering Claude Code]] in production — deterministic enforcement through structure, not probabilistic pleading through prompts.

## What's Missing

- **No cost breakdown for the follow-up workflows.** The $800 is the overnight session; the total bill is higher. Was it $2K? $5K? Worth knowing for teams budgeting similar efforts.
- **No discussion of what happens when upstream moves.** The guard tests are supposed to catch drift, but Cohen's fork was already 405 commits stale — the guidelines prevent recurrence, but they don't erase the merge-debt that already exists.
- **The "channel instances" architectural change (#2733) is undersold.** It's a genuine contribution to NanoClaw's core architecture, not just a cleanup. "Every NanoClaw multi-bot setup now gets this for free" — that's the open-source value proposition made concrete by an agent.

---

*Sources: [[raw/fable-nanoclaw-pr-factory]]*
*Last updated: 2026-07-05*
