---
url: https://brittany-ellich.offprint.app/a/3mrjj34puva23-108-prs-in-eight-days-accidentally-discovering-loop-engineering
title: "108 PRs in eight days: Accidentally discovering loop engineering"
author: Brittany Ellich
date_fetched: 2026-09-13
topics:
  - agent-coding-workflow
  - agent-orchestration
---

Brittany Ellich ran an agent against her personal task board for about eight days and got 108 tasks through the full pipeline — task picked up, code written in an isolated worktree, PR opened, CI green, review feedback answered — versus her previous average of 5–10 PRs per week. She is careful about the enabling conditions: a two-to-three person team with high trust and no mandatory second human reviewer, and review that moved rather than vanished — she still inspects every change, but at the testing-and-outcome level instead of line by line in a PR tab. A contrarian corollary: with merges this cheap she has flipped from wanting continuous deployment to preferring CI plus a preview/stage environment with batched releases, because it limits the regression-testing surface to once or twice a day.

She then discovered the practice she'd iterated toward has a name — loop engineering, coined by Addy Osmani in June, building on Boris Cherny and Peter Steinberger — but found most writing on it "stops at 'give it a clear goal and a turn cap.'" Her system has three layers: a protocol (a markdown rules file that lives next to the board, so editing rules and editing the board are the same act), a loop (reads the board, runs gh, updates task frontmatter, dispatches work, writes no code), and a worker (writes code in its own worktree, reports back a structured block, never touches the board, never talks to her directly).

The supporting mechanics are the substance of the piece. The board is a folder of one markdown file per task (queryable via Obsidian Bases) after a single-file table version collided with itself; the loop is the single writer to both board and memory, so concurrent workers can't corrupt state. Every worktree branches from the default branch, never her half-finished checkout. Bare Claude Code /loop with no interval is self-paced — short delays while a PR is active, long when quiet — and it ends itself when an explicit stop condition holds (nothing in progress, nothing waiting on CI, nothing merge-ready), so tokens stop burning the moment work runs out. A hard-capped memory file accumulates codebase facts and recurring defects with confirmation counts; anything confirmed often enough graduates into a task of its own (e.g. workers eventually tasked themselves with deleting a stale repo skill).

The honest coda: her board sits at zero queued and eight tasks waiting for her to test, so she is the bottleneck now. Both ends of the pipeline are spec work — deciding what goes in the queue and verifying what comes out — "being a software engineer in 2026 is just being a product manager and QA." Her advice for getting started is to write down precisely what "done" means before anything else; she has open-sourced her loop setup.
