---
url: https://spec-first.dev/
title: "Spec-First (spec-first.dev)"
author: Spec-First project
date_fetched: 2026-10-28
date_published: 2026-10-27
topics:
  - specifications-as-the-product
  - agent-coding-workflow
---

The page is a single worked demonstration: a complete agent session for building a small todo app (Vite + TypeScript, browser-only, localStorage) using the spec-first workflow and its four slash commands. The human gives a one-paragraph intent ("small todo app... add, tick off, filter open and done, browser only, no backend"), the agent reads `.spec-first/memory/MEMORY.md` and asks a clarifying question (should todos survive a reload?), then `/spec-first:create-spec` produces a spec YAML (`2026-10-27-7b2d-todos-first-version.yaml`) on its own git worktree branch.

The review step is the distinctive part: `/spec-first:review-spec` runs a deterministic `check-evidence.py` script that verifies the spec's evidence resolves, then spawns a fresh-context review agent that makes two edits — "BUILD WITH CUTS. Drag to reorder is out, a check for empty titles is in." Scope is reduced and a validation criterion added before any code exists.

Then `/spec-first:apply-spec` implements in three steps (store, list, tests), 9 tests pass, and a decision record is written to `.spec-first/decisions/` ("localStorage over IndexedDB, because a few hundred todos fit easily"). Finally `/spec-first:deliver-spec` commits behind a build+test gate and opens PR #12. The whole thing is small, legible, and ends with durable artifacts: a spec, a decision record, and a branch named after the spec.
