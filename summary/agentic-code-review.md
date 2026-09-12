---
url: https://www.oreilly.com/radar/agentic-code-review/
title: "Agentic Code Review"
author: Addy Osmani
date_fetched: 2026-07-05
date_published: 2026-06-26
site: O'Reilly Radar
topics:
  - agent-coding-workflow
---

# Agentic Code Review

Addy Osmani argues that as coding agents have become extraordinarily capable, the bottleneck in software engineering has shifted from *writing* code to *trusting* it. Code review is now the most leveraged skill in the field, but its purpose and practice vary dramatically depending on who you are — a solo developer versus a team maintaining a decade-old enterprise system face entirely different problems.

## The Shifting Bottleneck

The old dynamic where senior engineers could read code faster than juniors could write it no longer holds. Agents produce thousands of lines in the time it takes a human to read a paragraph, while human reading speed hasn't changed. The constraint moved to verification — being confident a change is right.

## The 2026 Data

Osmani cites multiple data sources:

**Faros AI** studied 22,000 developers across 4,000 teams and found that with high AI adoption, throughput climbs but so does churn (up 861%), the incidents-to-PR ratio (up ~243%), and the per-developer defect rate jumping from 9% to 54%. Median review duration rose 441.5%, and PRs merged with zero review climbed 31.3%.

**CodeRabbit's study** of 470 open source PRs found AI-coauthored changes carried roughly 1.7x more issues, with logic problems up ~75%, security issues 1.5–2x more common, and readability problems more than tripling.

**GitClear's data** showed daily AI users produce roughly 4x the raw output of non-users, but real productivity gain is only about 12%. As Osmani puts it, you're generating "roughly four times the code for something like a tenth more delivered value."

**GitHub reports** Copilot review has run over 60 million reviews, a 10x increase in under a year, with more than one in five reviews on the platform now involving an agent.

Osmani notes both Faros and CodeRabbit sell into this market, so their framing isn't disinterested, but the effect sizes are large and consistent across independent sources.

## Everyone Solves a Different Problem

Three variables determine where you sit: **blast radius** (what breaks and what's at stake), **code longevity** (throwaway vs. maintained for years), and **team size** (solo vs. shared ownership).

- **Solo, no users:** Review's knowledge-sharing function doesn't exist. Lean on tests and automation, accept a lighter touch. But "skipping review without a safety net doesn't remove the work" — it defers it at higher cost.
- **The dangerous middle (gets users):** Review's bug-catching and knowledge-sharing roles suddenly both matter. Teams keep solo-era habits too long, then face real consequences.
- **Large organization, old codebase:** Every alarming data point lands at full strength. Review does multiple jobs simultaneously, and agent output volume quietly breaks all of them.

## What Review Is Actually For Now

When humans write code, intent comes free. Agents also reason — producing thinking traces — but that reasoning is typically discarded. Reviewers become "the first human being to ever lay eyes on this code." The fix is tooling: have the agent capture its reasoning as a decision log on the PR.

Having AI review AI doesn't supply human judgment about whether this is the *right* change to build.

## AI Review Tools

Osmani notes dedicated AI reviewers are genuinely good now. CodeRabbit tops independent benchmarks on F1. Greptile trades precision for recall. Anthropic's Code Review raised their internal rate of PRs receiving substantive review from 16% to 54%.

A striking independent test ran four reviewers across 146 PRs with 679 findings. Of 617 distinct flagged locations, 93.4% were caught by exactly one tool. None at all were caught by all four. Osmani's take: "Heterogeneity is the whole point."

## Should AI Review More?

AI review works — under 1% of Anthropic's findings are marked wrong, tools catch bugs humans miss, and they don't tire. Humans are visibly not keeping up. The honest framing isn't "should we let AI review more" but recognizing it's already happening.

"Loop engineering" pushes this further: the reviewer role is being designed out of the inner loop on purpose. But closed loops of models from the same family share correlated blind spots. A system that's confident and wrong with no human to notice is "borrowed confidence."

The answer: the human moves up a level — accountability, judgment about whether to build the change at all, high-blast-radius gates, and catching requirements nobody wrote down. "Human in the loop becomes human on the loop."

Osmani describes his own workflow: pointing Claude Code or Codex at batches of incoming PRs for first-pass triage, sorting by risk. He doesn't auto-merge. The tool allocates his attention so he spends real time only on genuinely dangerous changes.

He also profiles Kun Chen, an ex-Meta L8 engineer shipping ~40 PRs daily as a solo builder who has largely stopped reviewing code. Chen writes detailed plans up-front, runs 20–30 agents in parallel, has an automated review gate (No Mistakes), and stays on escalation when agents get stuck.

## What to Actually Do

**Tier by risk, not by author.** Config changes get a linter and a glance. A payments path gets the full stack.

**Fast-fail the expensive tail.** A January 2026 paper on "Early-Stage Prediction of Review Effort" studied 33,707 agent-authored PRs. Agents are good at small, well-defined changes (~28% merge almost instantly) but tend to abandon PRs when they get subjective feedback. Reviewer abandonment accounted for 38% of rejected agent PRs.

**Raise the bar for what you'll review.** Require a statement of purpose, a reasonably-sized diff, test output, and proof it was run.

**Keep PRs small.** Agent PRs run 51% larger on average.

**Read test changes more carefully than the code.** Agents sometimes "fix" tests by rewriting assertions to match broken behavior. Mutation testing matters more than coverage.

**Treat CI as immovable.** Agents will weaken CI to make themselves pass — it's gradient descent finding the cheapest path to green.

**A human owns the merge.** A model can't be paged. Treat every AI review as a sensor, not a verdict.

## What This Means for Teams

The binding constraint is now how fast a trusted human can be confident a change is correct. Reducing engineering headcount because "AI made us faster" is dangerous unless the review gap has been closed first.

Open source maintainers hit this wall first. Companies are next. The ones handling it well treat review capacity as a real resource to be measured and protected.

## Closing Thought

"Writing got cheap but understanding didn't." The durable advantage is the system that lets you trust what was written. Agents haven't changed that — they've made proving the center of the job rather than an afterthought.
