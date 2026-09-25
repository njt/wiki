# Trying the Software Factory Pattern (Larson)

Will Larson's field report on adopting the software factory pattern at Imprint: a `/linear-project-loop` agent skill that audits a Linear project's goal scaffolding (RFC, Datadog/Snowflake metrics), reviews and updates issues, works non-blocked tasks, and reruns itself — plus the 2026 adoption arc that had to be built first to make any of it possible.

---

## What it says

Larson opens with the 2026 condition he finds most interesting: effective patterns now emerge faster than anyone can adopt them. His year at Imprint reads as a forced-march sequence — every engineer on Claude Code daily (January), everyone else on Claude Code or Cowork (March), a shift from repo-level to workspace-level development with ~10 independent checkouts so agents can ship cross-repo PRs across frontend, backend, infrastructure and data monorepos (April), a company-wide Jira→Linear migration so agents have a task system with "higher visibility and less permission complexity" (June), and finally the "Agent Fleet" orchestrated harness along the lines of Stripe's Minions (July).

Only then does the factory pattern arrive: "looping on a broad goal, and then relying on the harness to drive progress towards that goal." The implementation is unglamorous — a single agent skill that checks whether the project's goal definition actually exists (Notion RFC, measurable dashboards), creates what's missing, syncs issues against observed reality, works whatever isn't blocked, and restarts the audit when the project description goes stale. Larson credits the term's AI-context origin to Justin McCarthy's February 2026 "Software Factories And The Agentic Moment," while flagging that attribution is messy.

## Key quotes

> "new, effective patterns emerge faster than I can adopt them. I'll find a handful, get back to work, and realize a month later that I'd missed four or five more."

The honest version of "keeping up" in 2026 — not a skills gap but a throughput gap. Most adoption essays skip this; Larson makes it the frame.

> "I was already asking agents to iterate on specific Linear projects, but they didn't have the ability to evaluate if they were going in the right direction, or if it was missing necessary tasks. Now it does."

The sharpest insight in the piece: the factory pattern's value isn't autonomy, it's *externalized state*. Larson was "accidentally hoarding parts of the state for myself regarding the goals of the project" — the RFC and the metric dashboards had to exist in agent-readable systems before the loop could run at all. This connects directly to spec-as-control-surface arguments elsewhere in the wiki.

> "running the factory in a less frequent post-release mode would catch it immediately."

A genuinely underrated use case. Post-release monitoring is the lowest-glamour, highest-abandonment part of engineering; delegating it to an occasional agent loop that knows the project's goals is a better fit than dashboards nobody checks.

> "this factory pattern depends on having Datadog MCP and Snowflake access available to manage goal-tracking, but it also depends on Linear being the single source of state for the company's work, and an orchestrated harness that can perform work independently from your laptop."

The compounding observation. Nothing here works in isolation; each piece is worthless without the others. That's the real cost of the "keeping up with this many migrations is a fascinating industry moment" problem — adoption isn't additive, it's multiplicative.

## Themes

- #pattern — the factory loop: goal audit → metric review → issue sync → task work → re-audit when the description stales
- #concept — externalized state: agents can only be trusted with direction if goals and measures live where they can read them
- #tool — Linear as the agent-legible single source of work state, Datadog/Snowflake MCP as the goal-measurement layer
- #project — Imprint's Agent Fleet harness, in the Stripe Minions lineage

## Analysis

The most credible thing about this piece is its modesty. Where most factory writing is either a blueprint (Lloyd's [Cloud Software Factories]) or a victory lap (1,700 merged PRs in six months), Larson ships a single skill and is honest that it runs locally on his laptop for now. That makes it the most reproducible factory write-up in the corpus: the barrier to entry here is one skill file and a well-kept Linear project, not a control plane.

The piece also quietly argues something about *sequencing* that most factory advocacy skips. Look at the order: tool adoption, workspace topology, task-system migration, orchestrated harness — and only then the factory loop. Each earlier step exists because the later one exposed a missing substrate. The June Jira→Linear migration is the tell: agents can't operate a permission-complex system designed for human ceremony. If the factory pattern is real, the journey is not "adopt a factory" but "make your organization legible to agents," which is a much harder and less marketable claim.

It's also worth noting what's absent: no metrics, no merge rates, no token costs, no failure stories. This is a one-month-old experiment by a CTO on his own projects. The post-release monitoring use case is the strongest claim precisely because it needs the least autonomy — a loop that pings you about drift is low-risk in a way that a loop that merges PRs is not.

## Relations

This source strengthens [[Crawl, Walk, Run to the Software Factory]] — Warp's crawl/walk/run roadmap describes the adoption ladder in the abstract, and Larson's January→July arc is a concrete, org-scale instance of it, complete with the intermediate platform migrations (workspaces, Linear) the roadmap implies but doesn't name.

It nuances [[What Matters After the Software Factory Works]] — that six-month memoir covers what matters after the factory is scaled and merging thousands of PRs; Larson shows the opposite end, what matters to get one running at all, and the answer (externalized goals and metrics) turns out to be the same substrate from the other direction.

It complements [[Minions — Stripe's One-Shot Coding Agents]] — Larson explicitly models his Agent Fleet harness on Stripe's Minions, and this piece is a rare direct account of what it takes to absorb one-off-task harnesses into goal-driven factory loops.

---
*Sources: [[raw/software-factory-experiment]], [[summary/software-factory-experiment]]*
*Last updated: 2026-09-25*
