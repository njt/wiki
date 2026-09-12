---
url: https://claude.com/blog/the-ai-native-sdlc-playbook
title: "The AI-Native SDLC Playbook"
author: Anthropic
date_fetched: 2026-08-22
date_published: undated
topics:
  - agent-coding-workflow
---

# The AI-Native SDLC Playbook

**Author:** Anthropic (Applied AI team)
**Published:** undated, on the Claude engineering blog
**Length:** long-form guide / playbook

## Core Thesis

The traditional software development lifecycle (SDLC) was designed for an era when writing code was the most expensive, slowest stage. Agentic coding tools like Claude Code have inverted that: code is no longer the bottleneck, so the bottlenecks — and the controls — have migrated to the stages around the build (plan, review/test, deploy), which still run at human speed. Anthropic argues the SDLC needs the same transformation the implementation phase has undergone: a reimagined **AI-native SDLC** that keeps the old control objectives but changes the enforcement — a loop with AI embedded at every point, rather than a linear flow of human handoffs.

## The Core Idea: The Committed Artifact

The thread running through the whole playbook is the committed artifact. Every stage ends by writing something to version control — `intent.md`, `spec.md`, `plan.md`, the diff and its tests, the PR with its review findings, the incident record — and the next stage begins by reading it. Early stages use `.md` files because a product owner and an agent can both read and act on the same file. The chain of commits doubles as the audit trail: who asked for what, what the agent produced, who approved it.

## The Six Stages

Each stage is organized into modular "plays" (what changes / getting started / concrete steps / governance / how to measure it):

1. **Plan** — capture intent as `intent.md`, a proto-spec in the originator's own words, brainstormed with Claude. A person with an idea, a filed ticket, or an incident alert can all seed it.
2. **Design** — requirements and design collapse into one session; policy (brand, security, compliance, UX) is applied via skills *while* the spec is written, producing `spec.md`.
3. **Build** — nothing is implemented without an accepted `plan.md`. Institutional knowledge lives in CLAUDE.md and skills; hooks are deterministic guardrails; parallel sessions and subagents multiply throughput; Claude is given a feedback loop to verify its own work.
4. **Test** — continuous evals in CI are the AI-native equivalent of stage-gate QA, gating any change to CLAUDE.md, skills, or hooks.
5. **Deploy** — review runs in both directions (Claude reviews PRs and addresses comments on its own); hooks act as approval gates; CI/CD runs Claude non-interactively behind gates. The agent acts up to the production gate and nothing past it.
6. **Maintain** — the loop closes: a deterministic trigger (control-band breach, ticket, channel message) invokes Claude with no human in the invocation path, and what it finds re-enters the pipeline as a new `intent.md`.

## Governance Stance

Humans remain accountable for every decision requiring judgment; human attention concentrates at the gates, reviewing what the agent flagged rather than starting each stage from scratch. Skills are *advisory* controls (they make violations rare); hooks are the *deterministic* layer behind them (they make violations near-impossible). The document also stresses fitting around legacy systems of record (Jira, ServiceNow, Figma): for every artifact the process produces, name one system as the source of truth.
