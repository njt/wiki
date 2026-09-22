---
url: https://www.telerik.com/blogs/ai-coding-standards-repeatable-workflows
title: "AI Coding Standards and Repeatable Workflows"
author: Adam Bertram
date_fetched: 2026-09-22
date_published: 2026 (month not stated in source)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Adam Bertram's argument for a coding standard that outlives any particular AI tool: don't standardize which coding agent developers use — "the approved tool will change" — standardize what every change must demonstrate before merge. AI-generated code has severed the link between polish and care that reviewers' informal heuristics relied on (formatting ≈ effort, focused diff ≈ scope control, tests ≈ understanding), so review has to run on receipts instead of proxies.

The evidence marshalled: Stack Overflow's 2025 survey (84% of developers using or planning to use AI tools, 46% distrusting the results, 45% reporting that debugging AI code takes more time), GitClear's 623-million-change maintainability research (moved code fell from 21% of changed lines in 2022 to 3.8% so far in 2026 while copy-pasted code rose to 15.7%), and Veracode's spring 2026 update (over 95% syntax correctness but only about 55% of AI-generated samples passing security testing). The debugging problem and the duplication problem share one missing control: the repository never tells agents what "good work" means for this codebase.

The prescription is a placement rule sorted by enforceability. AGENTS.md carries what an agent must interpret — fleet-owned template from the platform team, per-repo local additions, a named owner who prunes stale rules. CI carries what a machine can block on: formatting, strict type checks, tests, changed-lines coverage, dependency allowlists, security scans — plus the sharpest single control, requiring every meaningful new test to fail against the pre-change code so coverage can't be gamed. The PR template carries evidence for the remaining human judgments, and "an AI-written description saying all tests pass is only a claim." Post-merge CI evidence doubles as the timestamped audit trail NIST's SSDF expects. A start-small rollout (one high-traffic repo, three repeated review comments moved into CI, two sprints) precedes the vendor coda: this is a Progress Forge (formerly Progress Agent Harness) top-of-funnel piece, and AGENTS.md is now stewarded by the Linux Foundation's Agentic AI Foundation.
