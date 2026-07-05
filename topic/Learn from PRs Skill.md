# Learn from PRs Skill

A Claude Code command that analyzes patterns in pull request feedback -- from CodeRabbit, SonarQube, and human reviewers -- then recommends configuration updates to catch those same issues locally before the next PR. It's a feedback loop that turns review comments into preventive rules: scan 5 recent PRs, find recurring themes (style issues, logic errors, security concerns, testing gaps), discover existing config files, and propose specific updates prioritizing the earliest intervention point.

---

## Key Quotes

> "This command only suggests changes, never modifies files."

## Key Themes

#feedback-loops #code-review #automation #configuration #shift-left

The "learn from your mistakes" pattern is powerful because it's self-improving. Every PR review becomes training data for your local config. The shift-left philosophy -- catch issues as early as possible in the pipeline -- connects to [[AI PR Reviewer]] (catching at PR time) and [[Fresh Eyes]] (catching at commit time). This skill pushes the boundary even further: catch it before you even start the next task.

The read-only design ("only suggests, never modifies") is a good trust-building choice. You want humans reviewing the proposed config changes, not auto-applying them.

## Critical Analysis

The concept is excellent; the execution details matter enormously. The value depends entirely on pattern quality -- can it distinguish between "reviewer has a style preference" and "reviewer caught a genuine bug class"? Grouping by frequency and severity helps, but false positives in config recommendations could create friction that makes developers disable the whole thing. See [[Awesome Agentic Patterns]] for the broader pattern catalogue this fits into (Feedback Loops category), and [[14 More lessons from 14 years at Google]]'s point about small PRs and code review standards.

---
*Sources: [[summary/learn-from-prs-skill]]*
*Last updated: 2026-05-14*
