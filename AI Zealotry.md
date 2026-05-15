# AI Zealotry

Matthew Rocklin makes the case that senior engineers are precisely the people who should be embracing AI coding tools. His argument: implementation is now "mostly free," so the high-level thinking that distinguishes experienced developers -- architecture, edge cases, design clarity -- becomes disproportionately valuable. The practical upshot is a set of specific techniques for trusting AI output without reading every line: hooks over CLAUDE.md, tests over code review, and fresh-agent audits for tech debt.

---

## Key Quotes

> "No, you're not too good to vibe code. In fact, you're the only person who should be vibe coding."

> "Stop doing simple shit."

> "Our ability to zoom in and implement code is now obsolete. Our ability to zoom out and think well is not."

## Key Themes

#agentic-coding #feedback-loops #testing #abstraction #senior-engineering

The central insight -- that AI is another rung on the abstraction ladder, like compilers before it -- is the strongest framing in the piece. Rocklin goes beyond philosophy into specifics: use hooks to enforce standards (because CLAUDE.md gets ignored), build confidence through tests and benchmarks rather than code reading, and separate `plans/` (ephemeral) from `docs/` (lasting). The recommendation to take long walks to protect thinking time is a nice counterweight to the productivity maximalism of most AI coding posts.

The language choice argument (Rust over Python, TypeScript for frontend) is more contentious. Python's ecosystem advantages are durable even if its ergonomic advantages shrink. But the underlying point -- that AI levels the playing field on syntax, so choose languages for their other properties -- connects to [[Addy Osmani's Workflow]] and the broader question of what skills survive automation.

## Critical Analysis

This is one of the better "senior engineer embraces AI" pieces because it's specific where most are vague. The hooks-over-CLAUDE.md insight is genuinely useful. The compiler analogy is apt but also convenient -- it lets him wave away legitimate concerns about code understanding. The weakest part is dismissing the "reviewing feels dehumanizing" critique; that's a real psychological cost, not just nostalgia. The piece would be stronger if it acknowledged that the transition period genuinely sucks even if the destination is good. Connects directly to [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] (Level 3 malaise) and [[The Next Two Years of Software Engineering]] (the skills atrophy question).

---
*Sources: [[raw/ai-zealotry]]*
*Last updated: 2026-05-14*
