---
url: https://claude.com/blog/how-to-prepare-for-ai-driven-code-modernization-projects
title: "How to prepare for AI-driven code modernization projects"
author: Anthropic Forward Deployed Engineers
date_fetched: 2026-09-29
date_published: 2026 (exact date not stated in source)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Anthropic's forward-deployed engineers lay out a six-step preparation process for running AI-driven code modernization on critical and regulated systems: define the **target** (transform vs. reimagine vs. uplift), define the **certificate** (machine-checkable conditions every change must meet), agree a **promotion policy** (tiered human review written down in advance), stage the environment, build a customized agentic workflow with Claude Code and its code-modernization plugin, pilot on a small slice before scaling, and treat the workflow, certificate, and evidence trail as reusable outputs.

The central observation is organizational: agents remove the bottleneck of *writing* changes, so the constraint moves to mobilizing the organization around them — change management, review, approval — processes built on the assumption that a human wrote and a human reviewed each diff. The certificate must be checkable without a human in the loop so the agent can iterate until it passes or flags itself; the promotion policy front-loads SME hours to the start (reviewing samples early instead of diffs at the end); and in regulated settings the directive for lighter review paths should come from the top, so responsibility for a bad change is shared rather than pinned on the individual approver.

On cost, they recommend piloting on a slice of the codebase and extrapolating token usage, moving compute-heavy verification behind cheaper gates, using cheaper models (e.g. Sonnet) for mechanical work the certificate fully checks and reserving stronger models for hard transformations and adversarial review — with careful analysis of whether several cheap retries cost more than one expensive attempt. Uplift modernizations (in-place, on a live codebase) proceed by partitioning from the leaves inward, freezing one partition at a time, and gating CI/CD so new commits cannot undo a modernized partition.
