---
title: "AI PR Reviewer"
url: https://dev.to/adamai/i-built-an-ai-pr-reviewer-and-it-already-caught-bugs-i-missed-115p
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - guardrails-and-feedback-loops
---

# AI PR Reviewer - AdamAI

GitHub Action using Claude to automatically review PRs. Caught bugs human reviewers missed.

## Real Example
Flagged an in-memory rate-limiting bug -- state would disappear on deployment and fail across multiple server instances.

## Technical Details
- Setup: One GitHub workflow file, one API secret
- Cost: ~$0.003-$0.02 per review; ~$1-2/month for 20 PRs daily
- Implementation: 250 lines Python, zero external dependencies (only urllib, json, re)
- Model Options: Claude Sonnet (default) or Opus for complex reviews

## Issue Categories
Critical, Major, or Minor. Verdicts like "REQUEST CHANGES."

## Strengths
Missing error handling, SQL injection risks, auth logic flaws, edge cases.

## Limitations
Cannot understand codebase conventions beyond the diff itself.

## Key Insight
"When you're deep in how something works, you stop seeing what it assumes."
