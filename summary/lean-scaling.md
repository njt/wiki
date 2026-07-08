# Lean Software Scaling Laws (Summary)

Gwern proposes using frozen LLM perplexity over codebases as a weak proxy for programming language design quality at scale. The core insight: well-designed languages produce codebases that become *increasingly* predictable as you see more of the system (you learn structure, invariants, patterns), while poorly-designed languages become *decreasingly* predictable as scale exposes bug-prone interactions and hidden coupling. By measuring perplexity scaling laws across languages, we could rank them by LLM-compatibility and predict crossovers — the point where a formally-strong language like Lean (high constant, good exponent) overtakes a popular-but-loose one like Python (low constant, bad exponent). The proposal is cheap enough for a grad student to run, fails gracefully (even null results about ecosystem-over-language would be useful), and directly addresses the chicken-and-egg problem of formal methods adoption: if Lean's scaling exponent is demonstrably better, the investment case for bootstrapping Lean training data becomes quantitative rather than religious.

---
*Source: [[raw/lean-scaling]]*
*Last updated: 2026-07-08*
