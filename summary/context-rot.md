---
title: "Context Rot"
url: https://roampal.ai/blog-context-rot.html
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - agent-memory-and-context
---

# Context Rot is Real: How Roampal Built Learning Memory

## Key Arguments

**The Core Problem**: Larger context windows don't solve retrieval quality. Chroma's research confirmed that performance degrades regardless of window size. More critically, current systems lack feedback loops—memories get retrieved and used, but nothing tracks whether they actually helped.

**The Decoupling Issue**: Retrieval and generation operate independently. Rerankers and hybrid search optimize for similarity, not success. As the article states: "they can't learn from outcomes. They optimize for similarity, not success."

## Three Technical Solutions

**1. Wilson Scoring for Cold Start**
The system addresses the problem where new memories appear artificially valuable (1/1 success) versus proven ones (90/100). Wilson scoring accounts for statistical confidence, ensuring memories must prove themselves over time.

**2. Dynamic Weighting**
Trust shifts as memories accumulate feedback:
- New memories: 80% embedding similarity, 20% outcome-based learning
- Proven memories: 20% similarity, 80% outcome-based learning

**3. Frictionless Scoring**
Rather than requiring manual feedback buttons, the LLM infers outcomes from user responses: "Thanks, that worked!" indicates success, while "No, that's wrong" signals failure.

## Benchmark Results

**Adversarial Testing (semantic trap scenarios):**
- ChromaDB baseline: 0% accuracy
- Roampal with learning: 60% accuracy

**Token Efficiency (memory retrieval accuracy):**
- Naive RAG: 1% retrieved helpful memory in top 3
- Roampal: 67% success rate

Cost savings: approximately 19 tokens per retrieval versus 50-90 for standard RAG, yielding potential annual savings of $18-37K for high-volume usage.

## Memory Architecture

Five collection types with different purposes:
- **working**: 24-hour auto-cleaning current conversations
- **history**: 30-day decay graduated memories
- **patterns**: Promoted solutions
- **memory_bank**: User facts and preferences (static)
- **books**: Permanent uploaded documents (static)

## Conclusions

The system develops learned patterns through accumulated feedback rather than hardcoded rules. Three knowledge graphs handle routing (collection selection), content relationships, and action tracking. The outcome: adaptive retrieval that improves with usage, reducing both token costs and retrieval latency while maintaining local data privacy.
