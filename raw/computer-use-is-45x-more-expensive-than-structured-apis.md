---
url: https://reflex.dev/blog/computer-use-is-45x-more-expensive-than-structured-apis/
title: "Computer Use is 45x More Expensive Than Structured APIs"
author: Palash Awasthi
date_fetched: 2026-05-14
date_published: 2026
---

# Computer Use is 45x More Expensive Than Structured APIs

By Palash Awasthi, Reflex.

## Overview

Benchmark comparing two approaches for enabling AI agents to operate the same admin panel: vision agent (screenshots + clicks via browser-use 0.12) vs. API agent (structured HTTP endpoints via tool-use). Same underlying application logic, same task.

## The Setup

Test application: admin panel for managing customers, orders, and reviews (modeled on react-admin Posters Galore demo).

- Path A: Vision agent — Claude Sonnet with browser-use 0.12, operating via screenshots and clicks
- Path B: API agent — Claude Sonnet with tool-use, calling HTTP endpoints directly

Task: Find customer "Smith" with the most orders, locate their most recent pending order, accept all pending reviews, mark the order as delivered. Touches three resources, requires filtering, pagination, cross-entity lookups, read and write operations.

Both agents call identical underlying application logic.

## Results

The vision agent couldn't complete the task with a simple 6-sentence prompt — found only 1 of 4 pending reviews, never paginated. With a 14-step explicit walkthrough, it succeeded but took ~14 minutes and ~500k input tokens.

| Metric | Vision Agent (Sonnet) | API (Sonnet) | API (Haiku) |
|--------|----------------------|--------------|-------------|
| Steps/Calls | 53 ± 13 | 8 ± 0 | 8 ± 0 |
| Wall-clock Time | 1003s ± 254s (~17 min) | 19.7s ± 2.8s | 7.7s ± 0.5s |
| Input Tokens | 550,976 ± 178,849 | 12,151 ± 27 | 9,478 ± 809 |
| Output Tokens | 37,962 ± 10,850 | 934 ± 41 | 819 ± 52 |

Haiku couldn't run the vision path (browser-use 0.12 schema limitations) but completed the API path in under 8 seconds for under 10k tokens.

## Key Insight: The Structural Gap

The cost differential is architectural, not model-dependent. Vision agents must render every intermediate state as a screenshot — each rendering = one screenshot = thousands of input tokens. Better models reduce per-screenshot error rates but cannot reduce the screenshot count needed. The interface determines step count.

Vision agent: reads pixels, must render every intermediate state.
API agent: reads structured responses from the same handlers.

## Hidden Costs

Vision agent lacked signals for incomplete data. API agent received pagination metadata ("page 1 of 4 with 50 results per page"); vision agent saw only rendered pixels. Each numbered instruction in the 14-step walkthrough represents engineering work not captured in token counts.

## Variance

- Vision: 749s to 1257s wall-clock, 407k to 751k tokens, 43 to 68 steps
- API: identical 8 calls every trial, ±27 tokens variance

## When Each Approach Applies

Vision agents remain appropriate for uncontrollable applications — third-party SaaS, legacy systems, unmodifiable systems. For internally-built tools, the math favors APIs.

Reflex 0.9 includes a plugin auto-generating HTTP endpoints from application event handlers, making the API path feasible without a parallel codebase.

## Reproduce

Benchmark code: github.com/reflex-dev/agent-benchmark (seed data, patched react-admin demo, both agent scripts, raw results).

Methodology: API path 5 trials, vision path 3 trials (capped due to 14-22 min/run and 400-750k tokens).
