---
title: "Learn from PRs Skill"
url: https://github.com/NTCoding/claude-skillz/blob/main/learn-from-prs/commands/learn-from-prs.md
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - guardrails-and-feedback-loops
---

# Learn From PR Feedback Command

Analyzes patterns in pull request feedback from code review tools (CodeRabbit, SonarQube) and human reviewers, then recommends configuration updates to catch similar issues locally before PR submission.

## Procedure

1. **Authentication Check** — Verifies GitHub CLI auth
2. **PR Discovery** — Fetches recent PRs (default: 5 merged)
3. **Feedback Collection** — Gathers review comments, PR reviews, discussion comments; categorizes as automated or human
4. **Pattern Analysis** — Groups recurring themes: code style, logic errors, security concerns, performance, testing gaps, documentation
5. **Configuration Discovery** — Searches for .editorconfig, linter configs, CLAUDE.md, identifies enforcement gaps
6. **Recommendations** — Proposes specific config updates, prioritizing earliest intervention point
7. **Summary Report** — Copy-paste-ready configuration snippets by file type

## Key Principle
"This command only suggests changes, never modifies files" — focuses on issues appearing multiple times across PRs.
