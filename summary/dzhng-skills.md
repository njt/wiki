---
url: https://github.com/dzhng/skills
title: "Skills — AI skills for building software factories"
author: dzhng
date_fetched: 2026-07-21
date_published: unknown
topics:
  - agent-coding-workflow
---

A personal library of domain-agnostic agent skills designed to be small, composable, hackable, and harness-agnostic — compatible with Claude Code, Codex, Cursor, and 70+ other harnesses. MIT licensed. Installed via `npx skills add dzhng/skills`.

The core philosophy treats software development as an autonomous factory: break work into independently verifiable slices, then loop through planning, building, and verifying until the goal is done. The spec is a living document that gets re-sliced mid-implementation as the agent discovers the plan is stale. Every slice must survive architecture review, code review, and visual review before the loop advances.

The workflow runs in three stages. **Plan** uses `/write-spec` to interview the user, research unknowns, and produce a spec under `specs/<feature>/` — a graph of independently verifiable slices. **Build** runs `/goal /implement-spec specs/<feature>` in a loop, optionally routing implementation and review to different agents. **Autonomous execution** lets the spec drive its own course: calling `/review` after each slice, `/screenshot-critique` and `/compare-screenshots` on visual work, and `/close-spec` when done.

The skills inventory spans three areas. Engineering skills cover slice-and-verify (explore-unknowns, write-spec, implement-spec, close-spec), review passes (code-review, audit-choices, review), test and docs generation, and dual-agent delegation to Codex and Claude Code. Visual review skills compare screenshots, critique visual output with an unprimed subagent, and open images for human eyeballing. Authoring skills cover creating and evaluating agent skills themselves. One specialized skill handles WebGPU / three.js rendering work.

Design principles: each skill is a standalone folder; harness-agnostic; autonomous loops with self-correcting specs; verify at every step; MIT licensed for open reuse.
