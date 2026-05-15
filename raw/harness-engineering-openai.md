---
url: https://openai.com/index/harness-engineering/
title: "Harness Engineering: Leveraging Codex in an Agent-First World"
author: Ryan Lopopolo
date_fetched: 2026-05-15
date_published: 2026-02-11
fetch_status: blocked_403
secondary_sources:
  - https://www.theneuron.ai/explainer-articles/openais-harness-engineering-playbook-how-to-ship-1m-lines-of-code-without-writing-any/
  - https://www.zenml.io/llmops-database/extreme-harness-engineering-building-production-software-with-zero-human-written-code
  - WebSearch results for "OpenAI harness engineering Codex agent-first 2026"
---

# Harness Engineering: Leveraging Codex in an Agent-First World

Ryan Lopopolo, Member of Technical Staff at OpenAI's Frontier Product Exploration team. Published February 11, 2026 on the OpenAI blog.

## The Experiment

Five months (Aug 2025 – Jan 2026). Three engineers, expanding to seven. Internal Electron-based beta application with real users. Hard constraint: zero manually-written lines of code.

Results: ~1 million lines of agent-generated code, ~1,500 PRs, throughput scaling from 0.25 engineer-equivalent per person to 3-10x.

## Core Definition

Harness Engineering: the engineer stops writing code and instead builds the structure, guardrails, documentation, and automated checks that keep AI agents on track. The name comes from horse tack — the AI is the horse, the engineer is the rider and tack-maker.

Term coined by Mitchell Hashimoto in early 2026.

## Key Practices

1. **AGENTS.md as Table of Contents (~100 lines)**: Not an encyclopedia. Points to structured docs/ directory. Progressive disclosure. Team tried one giant AGENTS.md and it failed — context dilution, stale rules.

2. **Encode Taste Into the Codebase**: Custom lint rules, automated reviewers. "If you can articulate what it is about the code you don't like, the next step is to write that down." Example: Codex kept duplicating a concurrency helper — wrote a custom ESLint rule banning it anywhere except the approved location. Rule itself was written by Codex with 100% test coverage.

3. **Push All Context Into the Repo**: "From the agent's point of view, anything it can't access in-context while running effectively doesn't exist." Slack threads, Google Docs, oral tradition — invisible to agents. Example: security library decision made in Slack caused wrong package pull two months later. Fix: @codex generated 4 guardrail PRs in 15 minutes.

4. **One-Minute Build Ceiling**: Builds must complete in ≤1 minute for the inner loop. Migrated Makefiles → Bazel → Turbo → Nx. Slow builds break agent flow. Treat them as signal to re-decompose the build graph.

5. **Humans Became the Bottleneck — Post-Merge Review**: Pre-merge human review eliminated entirely. Review happens post-merge on representative samples. Human role: identify systemic issues → encode fixes into docs/tests/lints. Review agents (not humans) review PRs, biased toward merging.

6. **Comprehensive Observability Stack**: Vector (logs), VictoriaMetrics (metrics), Grafana, distributed tracing — all bootable by Codex. Agents get Chrome DevTools Protocol access for UI inspection. Session logs aggregated daily; agent loops identify collective improvement areas.

7. **"Garbage Collection" Agents**: Initially every Friday was "slop cleanup" day. Replaced with recurring agent tasks that scan for pattern violations and open targeted cleanup PRs. Most reviewable in under a minute.

8. **Symphony — Multi-Agent Orchestration**: Elixir-based (BEAM/OTP for process supervision). Separate GenServers per task. Binary human decision: mergeable or not. If not → entire worktree + PR trashed, ticket goes to rework with analysis. Eliminated context-switching fatigue (the real bottleneck after ~5-10 PRs/engineer/day).

9. **Minimal Skills, Maximal Leverage**: Only ~6 skills for entire codebase. A `$land` skill coaches Codex through complete PR lifecycle: push → reviews → CI → fix flakes → merge → merge queue → landing.

10. **Architecture as Prerequisite**: "10,000-engineer-level architecture" for 7-person team: ~500 NPM packages, strict interface boundaries, extensive sharding. Rigid patterns limit how much context agents need.

11. **Ghost Libraries & Spec-Driven Distribution**: Software distributed as high-fidelity specs rather than source. Dependency internalization: agents rewrite and inline low-to-medium complexity libraries, stripping bloat. Brett Taylor: "Software dependencies are going away."

12. **Daily Standups Still Critical**: 30-minute standups because code velocity is so high that core architectural patterns can change without anyone noticing for weeks. Faster AI → more human communication needed, not less.

## Agent Legibility Score (Charlie Guo)

Seven-metric scoring system for how agent-friendly a codebase is: bootstrap self-sufficiency, task entry points, validation harness, linting/formatting, codebase map, doc structure, decision records. OpenAI's own Symfony repo scored a B.

## Key Quotes (Ryan Lopopolo)

- "If you can articulate what it is about the code you don't like, the next step is to write that down."
- "It's super easy to add leverage to your codebase by vibing up some new lints."
- "Each one is able to reduce slop in a unique way. But because everyone is invested in putting that knowledge into the codebase, everyone else's coding agents have the best guts of everyone on the team."
- "Context that we give to the agent to show them to what end do you use the tools you have and how do you use your tools well." (on skills)
- "The worker at the steam engine didn't disappear when the centrifugal governor was invented. He moved from turning the valve to designing better governors."

## Control Theory Framing

Lopopolo frames harness engineering through cybernetics (Greek kybernetes = steersman). The engineer becomes a designer of control systems rather than a direct producer of output. This is the same intellectual tradition Böckeler draws on in her Martin Fowler follow-up.

## External Validation

Cursor team independently reached nearly identical conclusions in their own experiments. Basis (YC startup, Series B) presented their implementation: two repos (Arnold for code, Atlas for everything else), skills with designated owners in frontmatter, sub-agents for standards enforcement, "dotnotes" for continuous decision logging.
