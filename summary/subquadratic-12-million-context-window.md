---
url: https://thenewstack.io/subquadratic-12-million-context-window/
title: "The context window has been shattered: Subquadratic debuts a 12-million-token window"
author: Frederic Lardinois
date_fetched: 2026-05-15
date_published: 2026-05-05
publication: The New Stack
topics:
  - ai-research-and-models
---

Article body not fully retrievable via WebFetch (truncated by page length). Content supplemented from SiliconANGLE coverage (https://siliconangle.com/2026/05/05/subquadratic-launches-29m-bring-12m-token-context-windows-ai/) and web search results.

## Summary

Subquadratic (subq.ai), a Miami-based AI startup founded in 2024, emerged from stealth on May 5, 2026 with $29M in seed funding and a claimed breakthrough: SubQ, an LLM using Sub-Quadratic Sparse Attention (SSA) to achieve a 12-million-token context window with linear O(n) scaling instead of the Transformer's quadratic O(n^2). The 13-person team is led by CEO Justin Dangel and CTO Alexander Whedon (ex-Meta, former head of GenAI at TribeAI). Backers include Javier Villamizar (ex-SoftBank Vision Fund) and Justin Mateen (Tinder co-founder), with a reported ~$500M valuation.

## Key Claims

- 12M token context window (vs. ~1M for Claude Opus 4.7 and Gemini 3.1 Pro)
- SSA is 52x faster than FlashAttention-2 at 1M tokens
- RULER @ 128K: 95% accuracy at $8 vs. Claude Opus 94.8% at ~$2,600 (~300x cheaper)
- SWE-Bench Verified: 81.8% vs. Opus 4.6 80.8% (Opus 4.7: 87.6%)
- MRCR v2 (1M tokens): 65.9% vs. Opus 4.6 78.3%
- 12M token recall: 92% (no frontier model can attempt this)
- Decoding speed: 150 tokens/s

## Products (Private Beta)

- SubQ API: full 12M-token context
- SubQ Code: CLI coding agent that loads entire codebases in one pass
- SubQ Search: deep research tool (initially free)

## Notable Quotes

CEO Justin Dangel: "We are very focused on the problem of how we transition from a dense attention, quadratic scaling architecture to a sparse attention linear architecture."

CTO Alexander Whedon: "If you double the input size with quadratic scaling laws, you need four times to compute; with linear scaling laws, you need just twice."

Dangel: "The fundamental scaling laws imposed by the transformer architecture and dense attention have been broken through."

Whedon on prior RAG/chaining workflows: "I used to manually curate prompts and retrieval systems and evals and conditional logic to chain together the workflows... a waste of human intelligence and also limiting to the product quality."

## Caveats

No published paper or open-source code. All benchmarks self-reported. Model card and technical report "coming soon." Community reactions range from excitement to "AI Theranos" skepticism. Model will not be open-weight in the near term.
