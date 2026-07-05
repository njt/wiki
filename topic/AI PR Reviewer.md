# AI PR Reviewer

A GitHub Action that uses Claude to review pull requests automatically. 250 lines of Python, zero external dependencies, costs $0.003-$0.02 per review (~$1-2/month for 20 PRs daily). In practice, it caught an in-memory rate-limiting bug the author had missed -- state that would vanish on deployment and fail across multiple server instances.

---

## Key Quotes

> "When you're deep in how something works, you stop seeing what it assumes."

## Key Themes

#code-review #github-actions #automation #bug-detection #shift-left

The cost/value ratio is the headline: $1-2/month to catch bugs that human reviewers miss. The rate-limiting bug is a perfect example -- it's the kind of issue that's invisible when you're thinking about the feature and obvious when you're thinking about the infrastructure. AI reviewers don't get tunnel vision because they have no tunnel.

250 lines, zero dependencies, one API key. This is the platonic ideal of an AI integration: small, focused, cheap, immediately useful. Compare with [[Fresh Eyes]] (different model reviews at commit time) and [[Learn from PRs Skill]] (learning from past reviews to prevent future ones).

## Critical Analysis

The limitation is important: it "cannot understand codebase conventions beyond the diff itself." That means it catches logic errors, security issues, and missing error handling, but misses architectural violations, naming conventions, and design pattern consistency. It's a complement to human review, not a replacement.

The categorization (Critical/Major/Minor) and verdict system (REQUEST CHANGES) are smart design choices -- they integrate with existing PR workflows rather than creating a parallel process. This connects to [[Addy Osmani's Workflow]]'s emphasis on automation as force multiplier and [[AI Zealotry]]'s argument for building confidence through systems rather than reading every line.

---
*Sources: [[summary/ai-pr-reviewer]]*
*Last updated: 2026-05-14*
