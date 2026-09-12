---
title: "Chief of Staff"
url: https://doneyli.substack.com/p/i-built-an-ai-chief-of-staff-that
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - personal-agents
  - agent-memory-and-context
---

# I Built an AI Chief of Staff

By Doneyli De Jesus, published March 2026 in "Signal Over Noise" newsletter.

## The Problem

Overwhelming information management across four email accounts, multiple calendars, family activities, and professional obligations. Critical items being missed amid noise.

## Core Architecture Patterns

### Two-Tier Processing
- Tier 1 (30-minute intervals): Rule-based scanning with zero LLM cost for urgent detection
- Tier 2 (daily 5 PM run): LLM classification and draft generation for accumulated emails

This reduced API costs by ~80% by reserving expensive model calls for judgment-requiring tasks.

### Graduated Autonomy
Trust operates across three escalating levels:
- Level 1: Full human approval required
- Level 2: Auto-send under strict conditions (edit distance <10%, confidence >0.9)
- Level 3: Autonomous sending with hardcoded exceptions for VIP/family contacts

Trust is a rolling 90-day window, not a permanent achievement. Can be revoked based on performance degradation.

### Three-Layer Memory
1. Observations: Zero-cost structural logging of corrections and preferences
2. Memories: Daily LLM synthesis creating durable institutional knowledge
3. Retrieval: Full-text search (BM25) capped at ~550 tokens per query

Memory decay prevents context pollution -- older memories fade unless repeatedly accessed.

## Key Quote

"When your agent sends an email it shouldn't have at 2 AM, 'I don't know how this layer works' is not acceptable."

## Technical Implementation

- Repurposed 2022 M1 MacBook Pro as dedicated agent server
- Docker for ClickHouse, Langfuse, Postgres; Python with Poetry
- macOS launchd for scheduling (7 background jobs)
- Tailscale for secure access; Signal for encrypted notifications
- Monthly LLM: ~$100 (Claude Max subscription supporting 26 agents)
- ~50 emails daily per tenant with >80% send rate accuracy

## Key Stats

313 commits, 43,000 lines of Python. 25 database tables, 133 test files (52 security tests). ~2,000 active observations, ~400 synthesized memories. Tier 1 alerts average 3-5 urgent catches daily in <10 seconds.

## Actionable Patterns

1. Deterministic Fast Paths: Route 80% through rule-based systems; reserve LLM for the 20% requiring judgment
2. Earned Trust: Never deploy agents with full autonomy; implement measurable graduation criteria
3. Bounded Memory: Three-layer capture-synthesize-retrieve with explicit decay functions
