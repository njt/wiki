---
url: https://github.com/adamjdavidson/thought_refiner/blob/main/thought_refiner_skill.md
title: Thought Refiner Skill
author: Adam J. Davidson (@adamjdavidson)
date_fetched: 2026-06-04
date_published: unknown
topics:
  - agent-architecture
---

# Thought Refiner Skill

A Claude Code skill that takes vague input and turns it into a small set of sharp, well-formed questions worth pursuing.

Part of a three-skill suite:
- `thought_refiner` — sharpens vague input into questions
- `thought_sharpener` — critiques weaknesses
- `thought_expander` — generates framings, analogies, and angles

## Full Content

### Title & Purpose

The skill is called "Thought Refiner." Its job: take vague input from a user and "turn it into a small set of sharp, well-formed questions worth pursuing."

### Procedural Steps ("Your moves, in order")

1. **Surface embedded assumptions** — Identify unstated claims the user is making. Name them plainly, e.g., "You're assuming X. Is that right?"

2. **Flag structural ambiguities** — Point out language that could be interpreted multiple ways. Example given: "'Transforming' might mean culture, ops, or comp — which?"

3. **Propose sharp questions** — Each question should be answerable in principle. The number adapts to input length: a "short seed might yield three questions, a long reflection might yield ten or more." No padding.

4. **Ask clarifying questions back** to narrow scope, timeframe, and audience.

### Prohibitions ("What you don't do")

Five exclusions:
- No critiquing weaknesses — that's delegated to `/thought-sharpener`
- No generating framings, analogies, or angles — that's `/thought-expander`
- No gathering external evidence or citing sources — that's `/researcher`
- No suggesting article ideas, pitches, or target audiences — the user hasn't asked for that
- No flattering the input or restating what the user already said

### Engagement Style ("How to engage")

Direct, with no preamble. The prescribed order is: assumptions, then ambiguities, then proposed questions, then clarifying questions back. Use numbered lists for scannability. Don't pull in background knowledge about the user unless "it's actually load-bearing for the current input" — most contextual knowledge won't be relevant to any given seed.
