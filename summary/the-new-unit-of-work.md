---
url: https://lexifina.com/blog/the-new-unit-of-work
title: "The new unit of work for document collaboration"
author: Lexifina
date_fetched: 2026-10-14
date_published: unknown
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Lexifina argues that a prompt — one agent run that writes, lints, compiles and tests before handing anything back — is a new unit of work sitting between the keystroke and the commit, and that it deserves its own checkpoint in version control. The essay grounds this in why code never had multiplayer: half-typed code doesn't compile, so collaboration collapsed into commits, which keep the state but lose the reasoning.

The interesting move is applying this to documents, not code. A contract behaves partly like code — change a defined term and the rest is wrong until the edit lands — so lawyers already work in a commit-like rhythm of emailed redlines. Lexifina's system has agents propose edits as tracked changes: one run is one set of proposals, and the author, the approver and the reason all attach to the same character. This goes further than git, which separates author from committer but leaves the approver living in the pull request, invisible to blame.

The result is a three-row accountability table — person (keystrokes, up to 30-second active slices), person-operated agent (one run per prompt, with tokens and cost of every model call), subagent (the same, plus each instruction and every proposed tracked change). Text selection can show who wrote each part, when, and who accepted it — per-character blame that git cannot do. The agent itself benefits: before editing a clause it learns whether a human rewrote or rejected its earlier wording there, so it stops repeating mistakes. Open questions are admitted plainly: a prompt is only atomic if its checks pass, and a contract has no compiler, so document checks are weaker, warn-level heuristics.
