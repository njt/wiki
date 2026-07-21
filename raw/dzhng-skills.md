---
url: https://github.com/dzhng/skills
title: Skills — AI skills for building software factories
author: dzhng
date_fetched: 2026-07-21
date_published: unknown
---

Skills is a personal library of domain-agnostic agent skills, reused across every project. Small, composable, hackable, and compatible with Claude Code, Codex, opencode, Cursor, duet.so, and 70+ other harnesses. MIT licensed.

## Installation

```bash
npx skills add dzhng/skills
```

Use `--list` to pick individual skills, or copy a `skills/<category>/<name>/` folder into a harness's skills directory (e.g., `.claude/skills/`).

## Philosophy

Software development is shifting from discrete tasks toward autonomous "factories" — agents pursuing goals independently until results are trustworthy. The hard part is breaking it into independently verifiable pieces, and knowing where the pieces even are.

The approach treats the unknown as "fog of war": map the terrain, carve it into isolated verifiable territories, and recursively re-slice anything hiding more map. "The spec is a living document, updated and re-sliced mid-implementation" as the agent learns the plan is stale. Each iteration gets "less wrong, until the goal is done."

Before the loop advances, every piece must survive architecture review, code review, and visual review against a stored baseline.

One unattended Codex run: "a single goal for 1d 16h on top of these skills, slicing and iterating until done."

## Workflow

1. **Plan** — Invoke `/write-spec` for a goal. The agent interviews you, researches unknowns, and produces a spec under `specs/<feature>/` — a graph of independently verifiable slices.

2. **Build** — Run the loop: `/goal /implement-spec specs/<feature>`. Add framing like a target branch or which agent implements vs. reviews.

3. **Autonomous execution** — The spec tells the loop when to call other skills: `/review` after each slice, `/screenshot-critique` and `/compare-screenshots` on visual work, `/close-spec` when done. The plan updates itself mid-implementation.

## Skills Inventory

### Engineering — slice, build, verify, repeat

| Skill | Purpose |
|-------|---------|
| explore-unknowns | Maps a task's unknowns across four quadrants — known knowns, interviews, reactable artifacts, blindspot passes |
| write-spec | Breaks large features into human-reviewable slices with "API seams and playable checkpoints" |
| implement-spec | Builds an existing spec to completion, "delegating independent slices in parallel" |
| implement-spec-with-codex | Runs implement-spec with Codex writing code while you orchestrate and review |
| close-spec | Archives a shipped spec, rewriting it from a build plan into "a durable rationale record" |
| refactor-clean | Refactors by moving ownership to "one clean concept instead of layering compatibility sediment" |
| write-tests | Writes tests as "tracer bullets that pin real behavior" — not implementation details or lucky samples |
| write-docs | Produces docs as "a glossary of principles and pointers, never a mirror of the code" |
| code-review | Audits diffs for stale names, dead references, needless complexity — ends with a clean/not-clean verdict |
| audit-choices | Audits an implementer's decisions rather than the diff — "a pure, never-blocking audit" |
| review | Combines refactor-clean, code-review, and write-docs into a single closeout pass |
| codex | Uses local Codex CLI as an independent second agent for review and delegated implementation |
| claude | Uses Claude Code (`claude -p`) as an independent second agent for consultation and delegation |

### Visual Review

| Skill | Purpose |
|-------|---------|
| compare-screenshots | Judges which image is "less wrong" against a target, shipping a reusable diff script |
| screenshot-critique | Uses an unprimed subagent as a second set of eyes on visual work; mandatory before declaring visual bugs fixed |
| preview-shots | Opens curated image shots in macOS Preview for user eyeballing |

### Authoring (skills maintenance)

| Skill | Purpose |
|-------|---------|
| write-skills | Creates or revises agent skills — triggers, leading words, progressive disclosure, failure modes |
| eval-skills | Evaluates skills against golden cases using "blind runs in fresh subagents, a separate judge, and gap-driven edits" |

### Graphics

| Skill | Purpose |
|-------|---------|
| renderer | Builds, debugs, or reviews WebGPU renderer work — three.js/TSL, node materials, WGSL passes, depth semantics |

## Design Principles

- Small, composable, and hackable — each skill is a standalone folder others can copy or customize
- Harness-agnostic — works with any tool that supports a skills directory
- Autonomous loops — the spec drives its own re-planning and re-slicing as implementation reveals new information
- Verify at every step — each slice must pass architecture review, code review, and visual review before moving on
- MIT licensed — fully open for reuse
