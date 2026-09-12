# Building low-level software with only coding agents

Lee Robinson built pixo -- a Rust image compression library competitive with mozjpeg -- using only AI coding agents. No hand-written code. Five days, 520 agent runs, 350M tokens, $2,871. The result: 38,000 lines of code, 900+ tests, 85% coverage, zero runtime dependencies. This is the closest thing to a public case study of [[Five Levels from Spicy Autocomplete to the Dark Software Factory|Level 4-5]] development.

---

## Key Quotes

> "I did not write any code by hand."

> "Writing code is no longer the bottleneck."

> "You have to figure out the right things to build."

## Key Themes

#agentic-coding #rust #case-study #testing #cost-of-ai #product-design

The numbers are what matter here. $2,871 and five days for a production-quality compression library. That's the deflationary pressure [[AI Killing B2B SaaS]] and [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] are talking about, made concrete. The 50%+ test ratio and CI/CD-first approach validate [[Addy Osmani's Workflow]]'s emphasis on testing as the primary quality mechanism when you're not reading every line.

The "longest single agent run: three hours" data point is important. Most people use agents for minutes. The capability frontier is much further out than typical usage suggests.

## Critical Analysis

This is simultaneously impressive and misleading. Impressive because the output is real, benchmarked, and open source. Misleading because Lee Robinson is not a random developer -- he's a former Vercel VP with deep systems knowledge. The "I didn't write code" framing obscures that he made hundreds of architectural decisions, chose algorithms, designed APIs, and steered agents away from dead ends. The code was generated; the engineering was very much human.

The cost breakdown ($2,871) is honest and useful. For a personal project it's expensive; for a company shipping a product it's trivial. The real question is whether this approach works for novel problems where you can't benchmark against existing solutions like mozjpeg. Compression algorithms are well-understood; would this work for something genuinely new?

---
*Sources: [[summary/building-low-level-software-with-only-coding-agents]]*
*Last updated: 2026-05-14*
