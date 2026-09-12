---
title: "Building low-level software with only coding agents"
url: https://leerob.com/pixo
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-coding-workflow
---

# Building Low-Level Software with Only Coding Agents - Lee Robinson

Lee Robinson built `pixo`, a Rust-based image compression library, using only AI coding agents over five days. Zero hand-written code.

## Stats
- 38,000 lines of code (50%+ tests)
- 520 agent runs, 350M tokens, $2,871
- Performance comparable to mozjpeg
- Zero runtime dependencies
- 900+ tests, 85% code coverage

## Key Arguments

**Expanding Agent Capabilities:** Modern agents can run for hours autonomously. Longest single run: three hours. Dramatic shift from typical 1-30 minute sessions.

**Verification Through Testing:** Agents perform better with verifiable outputs. CI/CD with tests, lints, benchmarks early enables self-correction.

## Notable Quotes

"I did not write any code by hand."

"Writing code is no longer the bottleneck."

"You have to figure out the right things to build."

## Main Conclusions

1. Product design matters more than coding capacity
2. Generalist knowledge remains valuable — understanding domains enables better decisions
3. Trade-offs require human judgment
4. Developers should experiment immediately rather than over-optimizing workflows
