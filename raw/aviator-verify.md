---
url: https://www.aviator.co/verify
title: Aviator Verify — Replace Code Reviews with Verified Intent
author: Aviator (corporate)
date_fetched: 2026-07-18
date_published: unknown
---

# Aviator Verify — Product Page Overview

**Source:** Aviator product website (aviator.co/verify)

**Author:** None listed (corporate marketing page)

**Published:** No date indicated (copyright 2026 on page)

---

## Hero Section

"Replace code reviews with verified intent" is the central pitch. The tool captures developer intent before a pull request is opened and then checks every acceptance criterion against running code with attached evidence.

---

## The Problem

"AI can generate in minutes what takes hours to review," leading to reviewer fatigue, rubber-stamped PRs, and what the company calls compliance theater. The page contrasts a 5-minute AI code generation time against a 45-minute average human review, claiming a 91% slowdown in review times.

The framing shifts the question: rather than asking whether code looks okay, the team argues the right question is "Does this match our intent and expected behavior?"

---

## Verification Methods

Criteria are routed through three layers:

1. **Scenarios** — The agent exercises the change end-to-end, capturing screenshots, tool calls, and request/response data.
2. **Invariants** — Past team review comments encoded as reusable checks so recurring flagged patterns never come back.
3. **Code-scan** — AST and structural checks for things like endpoint existence, return type matching, and dependency changes.

An LLM fallback handles "complex/custom criteria" with a confidence threshold, and the verdict is labeled accordingly when AI is used.

---

## How It Works (Four Steps)

**01 — Work locally with your agent** through Claude, Cursor, or Copilot. No new tooling is required. The example shows a developer asking Claude to add rate limiting to API endpoints, iterating until aligned.

**02 — Submit intent with the Aviator MCP.** The agent captures the intent, generates acceptance criteria from what was built, and submits everything to Aviator in a single tool call.

**03 — Aviator verifies end-to-end.** Scenarios are generated, team invariants applied, and evidence streams in during the run.

**04 — Review the behavior, not the diff.** Reviewers see each criterion with verdict, evidence, and comments, and can run ad-hoc scenarios, waive with reason, or approve.

---

## What You Review

Three review surfaces are presented:

- **Intent** — what the developer and agent agreed to build becomes the contract reviewers approve against
- **Evidence** — screenshots, matched invariants, code-scan results attached to each criterion
- **Decisions** — reviewers can ask the agent to test another scenario or check a preview deployment, then approve

---

## vs. AI Code Review

The page draws a direct contrast: AI code reviewers "read the diff" and "post comments" without knowing intent, whereas Aviator Verify runs scenarios, checks invariants, and scans code against every criterion the team agreed to. Key differentiators include reproducibility (same evidence per criterion), team patterns encoded as invariants, and an immutable audit record.

---

## Compliance

"Every verification produces an immutable audit record" — spec approval, implementation, verification, and deployment are all linked and timestamped. The page highlights segregation of duties (separate actors for spec approval, implementation, and verification), full traceability from intent to deployment, and one-click export for SOC 2, ISO 27001, or custom frameworks.

---

## Stack & Call to Action

The tool integrates with existing toolchains. The CTA offers MCP installation to "Submit your first intent" with no credit card required. A counter on the page displays "0 Pending reviews" and "23 Verified today" (presumably dynamic metrics rendered as static placeholder values in the captured content).
