---
url: https://github.com/EveryInc/compound-engineering-plugin
title: Compound Engineering Plugin
author: Kieran Klaassen and Trevin Chow (Every, Inc.)
date_fetched: 2026-09-08
date_published: 2026 (v3.24.0)
topics:
  - agent-coding-workflow
  - coding-agents-and-frameworks
---

# Compound Engineering Plugin

by Kieran Klaassen and Trevin Chow (Every, Inc.), MIT-licensed, v3.24.0.

## What it is

A plugin of 33 skills for AI coding agents that operationalizes the [[Compound Engineering]] philosophy as a loop: **brainstorm → plan → work → simplify → review → compound**. The compound step is the differentiator — each cycle writes a durable "learning" (a solution doc under `docs/solutions/`) that the next brainstorm and plan read as grounding, so the toolset gets smarter with every use. "Run one teaches it. Run two remembers."

## Scope

Runs on 14 agent hosts: Claude Code, Cursor, Codex (app and CLI), Kimi Code CLI, Cline, Grok Build CLI, Devin CLI, GitHub Copilot, Factory Droid, Qwen Code, OpenCode, Pi, oh-my-pi (omp), and Antigravity CLI. The repo root is itself a valid plugin package (`plugin.json` + `skills/`), and ships native manifests for each host (`.claude-plugin/`, `.codex-plugin/`, `.kimi-plugin/`, `.cursor-plugin/`, `.grok-plugin/`, `.devin-plugin/`, `.omp-plugin/`, `.pi/`, `.opencode/`, `.agy/`).

## Two parts

1. **The skills** (`skills/<skill>/SKILL.md` + `references/` + `scripts/`): the actual engineering methodology, written as progressively-disclosed markdown. Each skill has an *outcome*, a *done bar*, and loads its `references/` files only at the acting step. Heavy orchestration (ce-code-review, ce-plan, ce-compound) dispatches generic subagents seeded with skill-local "specialist prompt assets" rather than exposing standalone agents.
2. **The converter** (`src/`, ~8K lines of TypeScript on Bun): parses the Claude-native skill format (frontmatter + body) and converts it to each host's native plugin format — Codex TOML agents, Pi extensions, OpenCode config, Antigravity, Copilot, Kiro, Droid. Uses explicit converters + writers (one per target) and an install-manifest ledger so a writer never overwrites a path it didn't write.

## The loop, concretely

`/ce-brainstorm` (WHAT) → `/ce-plan` (HOW, a unified plan artifact) → `/ce-work` (execute, optionally via a cross-model author) → `/ce-simplify-code` → `/ce-code-review` (report-only multi-agent review) → `/ce-compound` (write the learning). `/lfg` runs the whole pipeline hands-off: plan, work, simplify, review-and-apply-fixes, browser tests, commit, push, PR, CI watch with a bounded repair loop.
