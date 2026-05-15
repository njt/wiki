# Slowing the Fuck Down

Mario Zechner's argument that deliberate friction in AI-assisted development is a feature, not a bug. Agents generate code without learning from mistakes, eliminate healthy pain signals, make locally optimal decisions that create globally complex messes, and have worse search recall as codebases grow. The fix: set rate limits on code generation aligned with your capacity to actually review it.

---

## Key Quotes

> "I would like to suggest that slowing the fuck down is the way to go. Give yourself time to think about what you're actually building and why. Give yourself an opportunity to say, fuck no, we don't need this."

> "Set yourself limits on how much code you let the clanker generate per day, in line with your ability to actually review the code."

> "An agent has no such learning ability. At least not out of the box. It will continue making the same errors over and over again."

> "The bigger the codebase, the lower the recall."

## Key Themes

#agentic-coding #code-quality #human-judgment #friction #complexity

This is the philosophical counterweight to every "agents are amazing" post. Zechner identifies four compounding problems: no learning from mistakes, no pain signals triggering improvements, local decisions creating global complexity, and degrading search as codebases grow. Each one is individually manageable; together they create the "unrecoverable mess of complexity" he warns about.

The connection to [[The Mythical Agent-Month]] is direct -- McKinney's "brownfield barrier" is what happens when you don't slow down. And [[Cognitive Debt]] describes the organizational consequences: the gap between code production velocity and human comprehension velocity widens until the system breaks.

The practical prescription -- agents for scoped self-evaluating tasks, humans for architecture and quality gating -- aligns well with [[How to Write a Good Spec for Agents]], which provides the mechanics of how to actually structure that division of labor.

## Critical Analysis

Zechner is right about the diagnosis but the prescription of "just slow down" is incomplete. The more actionable version comes from [[Compound Engineering]] and [[Pre-Commit Lint Checks]] -- don't rely on human discipline to slow down, build mechanical constraints that make speed without quality impossible. "Set yourself limits" is good advice that almost nobody follows. "Your CI pipeline rejects this" is enforcement that actually works. Still, the core insight -- that removing friction removes learning -- is one of the most important ideas in agentic coding.

---
*Sources: [[raw/slowing-the-fuck-down]]*
*Last updated: 2026-05-14*
