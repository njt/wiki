---
url: https://codewithandrea.com/articles/ai-agents-migrate-site-astro/
title: "AI Agents Migrate a Site to Astro"
author: Andrea Bizzotto
date_fetched: 2026-08-14
topics:
  - agent-coding-workflow
---

A case study of using AI coding agents to migrate Code With Andrea — a 400+ page
static site built on a custom Swift generator (Publish) — to Astro, preserving
100% feature parity. Done solo over a two-week holiday with only a couple of
hours a day at the keyboard, running no more than two agents in parallel inside
tmux on a VPS.

The through-line is a five-stage workflow that keeps human judgment at every
high-stakes decision. First, "feature parity" is turned into a *migration
contract* — a testable description of every route, redirect, metadata field,
asset, interaction, and intentional exception. A throwaway prototype proves the
riskiest assumption (Astro content collections could preserve canonical URLs)
before bulk migration. Content conversion is handled by a *deterministic,
one-way migration tool* with its own test harness that fails loudly on unknown
placeholder syntax rather than guessing. Verification is split into fast,
focused checks for normal agent runs and a full production-parity suite (plus
curated screenshots) at release gates.

The central claim: agents make implementation cheaper, but they cannot know
which legacy behavior is intentional, which is obsolete, or which trade-off is
acceptable to your users. The author keeps the final product and production
decisions — including the cut-over — in human hands. Roughly 100 issues and 80
reviewed PRs later, the site went live (the article itself is served from the
migrated Astro build).
