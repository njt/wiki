# Cognitive Debt

When velocity exceeds comprehension. AI-assisted development has decoupled code production from code understanding -- "code has become cheaper to produce than to perceive." The organizational assumption that reviewed code is understood code no longer holds. The result: high output combined with low confidence, invisible to every metric that matters, until something breaks.

---

## Key Quotes

> "Code has become cheaper to produce than to perceive."

> "They are debugging a black box written by a black box."

> "The system is optimizing correctly for what it measures. What it measures no longer captures what matters."

> "The organization loses knowledge not just through attrition but through insufficient formation."

## Key Themes

#concept #cognitive-debt #comprehension #metrics #organizational-knowledge

This is one of the most important conceptual pieces in the agentic coding space. The core argument has three parts:

**The comprehension lag.** Manual coding couples production with absorption -- typing forces engagement. AI decouples them. Output velocity accelerates; understanding velocity doesn't.

**The measurement blindness.** Story points, features delivered, commit rates -- all assume comprehension accompanies shipping. Cognitive debt is invisible to every standard engineering metric.

**The reviewer's dilemma.** AI reverses the review dynamic. Juniors now generate faster than seniors can audit. Deep review becomes the bottleneck, so organizations implicitly choose throughput over comprehension.

The three failure modes are chilling: production code becomes dangerous as systems age without understanding, incidents require debugging code nobody comprehends, and junior engineers never develop architectural intuition because they never struggled with implementation.

Directly connected to [[The Mythical Agent-Month]] (the codebase-level version of the same problem), [[Slowing the Fuck Down]] (the practitioner's response), [[acceleration-flow]] (the psychological experience), and [[Compound Engineering]] (one approach to building the systems that close the comprehension gap).

A senior cousin is [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]]: where this page's debt is "code cheaper to produce than to perceive," that one's is a decision made silently with no signature at all — so the comprehension gap never even registers as a gap to measure.

## Critical Analysis

The diagnosis is sharp. The prescription is where it falls short -- the article identifies the problem but offers no concrete mechanism for making comprehension legible to performance systems. The actionable versions come from elsewhere: [[Pre-Commit Lint Checks]] forces engagement with quality, [[engineering-notebook]] creates a forensic record, [[napkin]] builds an explicit knowledge artifact. [[Agents and Acquiring Debt]] supplies the mechanism this diagnosis lacks — agents answer *what*, not *why* (state and commits, not folklore or rejected alternatives) — and a concrete fix: ADRs written at decision time, which agents are their hungriest readers. [[Principal Drift]] supplies the operational half the same way: a tier-1/tier-2 routing rubric, a builder/reviewer separation rule, and three comprehension-embedding techniques — the concrete mechanism for keeping comprehension legible while output climbs that this page says is missing. But the underlying insight -- that we are optimizing for what we can measure while the thing that matters is unmeasurable -- is not just about coding. It's about every domain where AI accelerates output. [[AI Handles Incidents, Engineers Lose Touch with Their Systems]] makes that point concrete in operations: an AI SRE that auto-resolves routine incidents drains the very practice responders need for the novel ones — Bainbridge's Irony of Automation, restated as comprehension debt.

---
*Sources: [[summary/cognitive-debt]]*
*Last updated: 2026-05-14*
