---
url: https://www.adamhjk.com/blog/a-practical-guide-to-reducing-token-spend
title: A Practical Guide to Reducing Token Spend
author: Adam Jacob
date_fetched: 2026-07-25
date_published: 2026-07-16
---

# A Practical Guide to Reducing Token Spend

Adam Jacob describes how replacing a traditional AI agent skill with a "swamp workflow" dramatically reduced token consumption and improved performance. His Garfield code review extension dropped from ~4.5 million tokens to ~500,000 tokens (an 8x reduction) and cut execution time from ~12 minutes to ~6.5 minutes (a 2x improvement).

## The Problem: David Cramer's Experience

David Cramer of Sentry reported spending over $10,000 on tokens in a single week. He observed that "new models are too expensive to use broadly at Sentry," estimating an average spend of $1,000/week/developer. Cramer's Garfield skill used a coordinator agent dispatching rolling sub-agents to review code against policy standards. This design "puts the LLM in the hot path" for parts of the workflow that don't need AI intelligence.

## How Garfield Originally Worked

The original skill consumed ~4.5 million tokens, ran for ~12 minutes, and employed 23 sub-agents. It used a coordinator agent to dispatch review cycles, with sub-agents evaluating code and reporting back to the coordinator for adjudication. A final verification pass checked tests, formatting, and lint.

## The Swamp Workflow Solution

Jacob rebuilt the skill as a swamp extension. The new version used ~500,000 tokens, ran ~6.5 minutes, and required only 3 agents total. The key insight was replacing the non-deterministic coordinator loop with "deterministic code, farming out work to agents when we need their intelligence." He stored sub-agent results as "versioned, typed data" for full visibility and to avoid costly re-work.

## Step-by-Step Guide

### Step 0: Installation
Install swamp and initialize the repo using `swamp repo init --tool codex`. The `--tool` flag customizes initialization for specific harnesses.

### Step 1: Understand Current Behavior
Read the existing skill thoroughly. Jacob recommends asking an LLM to summarize the skill's behavior, noting that even if you already understand it, "I needed that information in the context window for the translation."

### Step 2: Translate to a Swamp Extension
Ask the agent to produce a plan first, then implement. The critical check is ensuring the plan captures the intended *outcome*, not the implementation details.

### Step 3: Black Box Testing & Measurement
Jacob advocates for black box UAT — using AI to build a test suite that runs both the original and new workflow side by side.

Benchmark results:

| Case | Treatment | Result | Tokens | Agents | Time |
|---|---|---|---|---|---|
| contained-dry-run | garfield (original) | pass | 4,639,565 | 23 | 12.8 min |
| contained-dry-run | workflow-garfield (swamp) | pass | 506,484 | 3 | 6.6 min |
| payment-idempotency | garfield | fail | 12,277,282 | 45 | 30.2 min |
| payment-idempotency | workflow-garfield | fail | 1,570,242 | 6 | 10.5 min |

Notably, the original skill "failed *open*" (declared success while leaving defects), while the workflow "failed *closed*" (reported unresolved findings). Rather than interpreting this guardrailing as a weakness, Jacob found it a desirable fail-secure pattern for responsible AI code audit.

### Step 4: Refactor and Refine
Jacob emphasizes that "you still have to refactor, even when you use AI." Swamp workflows are easy to refactor because they are "normal" deterministic code.

## Why It Works

Swamp provides primitives for building "an adaptive model of the problem at hand." The approach concentrates LLM usage where it delivers the most value and moves deterministic logic into reusable building blocks. Jacob sums it up: "You use the Agent to build the program that minimizes the need for the Agent itself."

The article closes with the Latin phrase **"Lex Aedificandi, Lex Credendi"** — as we build, so we believe.

*Footer credit: "Made by Gerald McClaw," described as Jacob's agent and "resident Hobbit at large," with links to GitHub and swamp.club.*
