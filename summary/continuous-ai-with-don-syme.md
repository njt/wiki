---
url: https://gist.github.com/njt/022cc46bcfca6d72131b198e9075ba0c
title: "Continuous AI (with Don Syme)"
author: Nat Torkington
date_fetched: 2026-08-26
site: gist.github.com
topics:
  - agent-coding-workflow
---

# Continuous AI (with Don Syme)

Notes and transcript of an **AI Native Dev** podcast episode (hosts Guy Pigiani and Simon Maple) featuring **Don Syme**, principal researcher at GitHub. Syme introduces **Continuous AI** as a distinct category alongside CI and CD, and details **GitHub Agentic Workflows**, GitHub's implementation of it, now in public preview.

## The core idea

CI establishes that software works; CD establishes that it is correctly deployed — both deterministic. Continuous AI covers the *subjective* activities — bug triage, documentation, performance improvement, duplicate detection, issue labeling — that need to run on a permanent, operational basis inside a bounded context, but have no binary pass/fail outcome. Syme is emphatic that this is not AI-inside-CI: "It's CI CD, they stay as they are."

The shift that matters, Syme argues, is from **individual productivity** (the chat modality) to **situated, collaborative automation** — AI folded into the change/event-driven rhythm that already governs software teams, with the repository as the natural bounded context.

## Guardrails, review, and cost

Syme's running metaphors: "the stronger the train tracks, the faster you can run" — guardrails are an enabler of speed, not a limit — and "you've got to be able to sleep at night." Information-flow integrity (never expose private repos to public-repo agents, default to ignoring untrusted contributors) is existential.

Human review is the bottleneck, so the factory creator's job is to *equip* the reviewer: evidence (before/after CI timings), risk assessment, trade-offs, and quality gates that throw away trash before it reaches a human. "If it produces rubbish, you throw it away."

Cost control, Syme claims, is "the thing that's missing from the harness discussion." GitHub Agentic Workflows ships per-run cost visibility, budgeting, and "examination testing" — running the same work with multiple models simultaneously to compare performance.

## The workflow debate

Pelle de Hallio's "Agent Zoo" explores dozens of workflow patterns. But for maintaining real repos, Syme prefers one broad-spectrum workflow (his own **RepoAssist**: labels issues, does first-response research, optionally implements fixes, never auto-merges) for conceptual simplicity, cost control, and attention management across many repos.

## Open questions

The transcript leaves human-review scaling (auto-merge pressure), evaluating subjective automation quality, cross-repo/org context, and prompt-injection security largely unaddressed — Syme acknowledges them as future work.
