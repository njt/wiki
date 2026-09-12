---
url: https://www.aviator.co/verify
title: "Aviator Verify — Replace Code Reviews with Verified Intent"
author: Aviator
date_fetched: 2026-07-18
date_published: unknown
topics:
  - agent-coding-workflow
---

Aviator Verify is a product that replaces traditional code review with
intent-based verification. The core pitch: instead of asking whether code
looks okay, ask whether it matches the team's stated intent and expected
behavior. Developers capture what they and their AI coding agent agreed to
build *before* opening a PR, and Aviator checks every acceptance criterion
against running code with attached evidence.

Verification runs through three layers: end-to-end **scenarios** that
exercise the change and capture screenshots, tool calls, and
request/response data; **invariants** that encode past team review comments
as reusable checks so flagged patterns don't recur; and **code-scan**
checks for structural concerns like endpoint existence, return-type
matching, and dependency changes. An LLM fallback handles complex criteria
with a confidence threshold, and the verdict is labeled accordingly.

The workflow integrates with existing AI coding tools (Claude, Cursor,
Copilot) via an MCP server. The agent captures intent, generates acceptance
criteria, and submits everything in one tool call. Reviewers then see each
criterion with a verdict, evidence, and the ability to run ad-hoc
scenarios, waive with reason, or approve. Every verification produces an
immutable audit record with full traceability from intent to deployment,
targeting SOC 2 and ISO 27001 compliance needs.

The page contrasts Verify against AI code reviewers that "read the diff
and post comments" without knowing intent, emphasizing reproducibility,
encoded team patterns, and the audit trail as differentiators.
