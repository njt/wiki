---
url: https://github.com/pullboard-dev/pullboard
title: "Pullboard — a coordination queue for agents building software from a spec"
author: Corey Olson (pullboard-dev)
date_fetched: 2026-10-11
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

Pullboard is a local-first coordination layer for teams of coding agents working one repo. It installs as an npm CLI (`pullboard init`) and gives agents a shared spec (`SPEC.md`), a rules file (`DOCTRINE.md`), separate git worktrees ("lanes"), and a git-hook + gate system so nothing counts as done until the tests pass at the submitted commit *and* a second, different agent verifies it. Everything lives in the repo: the board is a single SQLite file inside `.git`, shared by all worktrees and never committed; no account, no hosted service, no model calls.

The model is a hierarchy with the human at the top: you approve rows of the spec, one "coordinator" agent plans lanes and items and fans work out, "builder" agents claim items atomically (one agent per item, one lane per worktree), and a "verifier" agent that built none of it checks the exact pinned commit and accepts or rejects with a written reason. Questions flow up the chain ("shouts"), and only decisions that are genuinely yours reach you. The board is a state machine — open → claimed → submitted → verified/withdrawn — with the transitions declared once in `src/machine.js` and enforced against every command.

The README pitches it as "vibe code a real product": a vibe-first, spec-driven, test-proven methodology where judgement is reserved for the human in the loop. The authors dogfood it ("nearly one in three items goes back"), and ship receipts showing real reject/accept cycles with evidence.
