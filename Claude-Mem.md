# Claude-Mem

A Claude Code plugin that captures everything Claude does during coding sessions, compresses it with AI, and injects relevant context into future sessions. The architecture: lifecycle hooks capture observations, a Bun-powered worker service stores them in SQLite + Chroma vector DB, and a 3-layer search system (index -> timeline -> full observations) achieves ~10x token reduction compared to naive retrieval.

---

## Key Quotes

> "Claude Code plugin that automatically captures everything Claude does during your coding sessions, compresses it with AI, and injects relevant context back into future sessions."

## Key Themes

#memory #context-management #plugins #vector-search #token-efficiency

The 3-layer progressive disclosure pattern is the smartest design choice. Layer 1 returns compact indexes (~50-100 tokens), Layer 2 adds chronological context, Layer 3 delivers full observations (500-1,000 tokens) only for filtered results. This mirrors how human memory works -- you don't recall every detail, you recall enough to know whether to dig deeper.

The privacy controls (`<private>` tags) and citation system (reference past observations by ID) show thoughtful design for real-world use. Compare with [[CodeMira]] (similar concept for OpenCode) and [[Planning With Files]] (file-based persistence instead of database-backed memory).

## Critical Analysis

The fundamental bet is that past session context is valuable enough to justify the overhead of capturing, compressing, storing, and retrieving it. For long-running projects, this is probably true. For one-off tasks, it's overhead. The Chroma dependency adds significant complexity -- SQLite alone might be enough for most use cases.

The interesting comparison is with [[Awesome Agentic Patterns]]' Context & Memory category (20 patterns). Claude-Mem implements several of those patterns in a single opinionated package. The question is whether one-size-fits-all memory works or whether different projects need different memory strategies. The plugin approach (install and forget) trades flexibility for accessibility -- the right tradeoff for adoption, possibly wrong for advanced users.

---
*Sources: [[raw/claude-mem]]*
*Last updated: 2026-05-14*
