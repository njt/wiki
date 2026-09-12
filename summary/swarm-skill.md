---
url: https://github.com/jleechanorg/claude-commands/blob/main/.claude/skills/swarm/SKILL.md
title: "/swarm — Multi-Agent Swarm Orchestration Playbook"
author: jleechanorg
date_fetched: 2026-07-18
date_published: 2026-07
topics:
  - agent-orchestration
---

A playbook for running multi-agent swarms via Claude Code, distilled from real 2026-07 design-retro swarms (~180 agents, ~7M subagent tokens). It covers two fan-out engines — the Workflow tool (ultracode, the default) and Agent Teams (interactive lanes for mid-flight steering) — plus a mandatory sidekick durability layer that wraps every swarm for crash recovery.

Every candidate finding faces adversarial verification: three independent lenses prompted to refute, surviving only when at least two don't refute. Dead verifiers (rate limits, API errors) are not counted as refutations — the playbook requires auditing for false kills and detecting false-empty completions where mass agent death zeroed out real findings.

The sidekick runs as a real tmux Claude Code process, persisting state to a branch-scoped `STATE.md` and a `br` bead after every step. Crash recovery means `/sidekick` in any fresh session. A 5-minute checkpoint cadence caps data loss at ≤5 minutes.

Hard rules, each learned from failure, cover: explicit cost-routing per agent (haiku for mechanical, sonnet for miners/verifiers, top-tier only for adversarial judgment); rate-limit resilience via staggered fan-outs and serializing across concurrent swarms; disjoint output dirs per lane; commit-and-push per completed artifact; pre-building static datasets for miners; a mandatory publishability gate that redacts secrets, checks cross-doc consistency, and re-verifies freshness; and a cross-model cold review (different model family) before any merge-readiness claim.

Canonical phase shapes are provided for retrospectives, solution-hardening, code-quality sweeps, innovation passes, and fleet triage — all domain-general, substituting non-coding artifacts (market research, ops audits, content pipelines) freely. The playbook also warns about CI-triggering fan-outs overwhelming self-hosted runner capacity and about under-utilization from blocked gates unnecessarily freezing sibling work.
