---
url: https://github.com/mattpocock/skills
title: "Skills For Real Engineers (mattpocock/skills)"
author: Matt Pocock
date_fetched: 2026-09-25
date_published: 2026 (ongoing, v1.2.3)
topics:
  - agent-coding-workflow
  - claude-code
---

Matt Pocock's personal collection of 25+ agent skills, shipped as a Claude Code plugin (`claude plugins install mattpocock-skills`) and via skills.sh for editable copies. The explicit positioning: "skills for real engineering — not vibe coding," and an argument against all-in-one process frameworks (GSD, BMAD, Spec-Kit) that "own the process" and make process bugs hard to resolve. Instead: small, composable, hackable skills that work with any model.

The repo is essentially pure prose engineering — a handful of bash scripts, no application code. Each skill is a `SKILL.md` (20–140 lines) plus reference docs, organized into `engineering/` and `productivity/` buckets with an `in-progress/` staging area. A `CONTEXT.md` at the repo root is itself written in the domain-modeling format the skills teach (terms, relationships, flagged ambiguities) — the repo eats its own dog food.

The core artifact is the **main flow** documented in the `ask-matt` router skill: `grill-with-docs → to-spec → to-tickets → implement → code-review`, plus on-ramps (triage for incoming issues, diagnosing-bugs for hard bugs, wayfinder for multi-session efforts). Supporting primitives include `grilling` (frontier-based interview rounds), `domain-modeling` (CONTEXT.md glossary + ADRs written inline), `codebase-design` (deep-module vocabulary: seam, depth, leverage, locality), `wizard` (generated bash scripts for human-only steps), and `handoff`/`clear`/`compact` phase-boundary management around the "smart zone" (~150k tokens) of sharp reasoning.

Notable engineering details: a two-axis code review (Standards vs Spec) run in parallel sub-agents that are explicitly never reranked against each other; a triage state machine with `ready-for-agent`/`ready-for-human` roles and mandatory AI-generated-content disclaimers; an explicit inter-skill invocation convention ("Call the Skill tool with…") and a user-invoked vs model-invoked split enforced in both Claude Code and Codex frontmatter; and `diagnosing-bugs`' refusal to theorize before a tight red-capable feedback loop exists.
