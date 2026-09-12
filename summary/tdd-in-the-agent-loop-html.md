---
url: https://martinfowler.com/articles/exploring-gen-ai/tdd-in-the-agent-loop.html
title: "TDD inside the agent loop - theater or actual value?"
author: Thoughtworks technologist (Exploring Gen AI series; byline not captured in fetch)
date_fetched: 2026-09-13
date_published: 2026-09-02
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A Thoughtworks technologist (Exploring Gen AI series, reviewed by Ivett Ördög, Matteo Vaccari, Emily Bache and others) runs a small controlled experiment: three greenfield Python business-logic tasks in small/medium/large sizes, built by Sonnet 4.6 with and without strict TDD instructions, judged blind by Opus 4.8, with TDD adherence itself audited from session transcripts by an independent agent.

The headline result: **no discernible quality difference** between TDD and non-TDD runs — and Opus more than once ranked the non-TDD solutions slightly higher on design and test quality. Mutation scores showed no meaningful TDD advantage either. TDD runs cost 3x–8.5x more tokens (with the caveat that cache-read token accounting inflates the true dollar gap). In five batches, the non-TDD pair took #1 and #2 in the small/medium tasks; a TDD solution won only once, after the prompt was strengthened with an explicit design-and-refactor step — while its identical-prompt sibling ranked last.

Why: Opus's trace analysis found the non-TDD and test-first runs designed the full architecture, data types, edge cases and contracts *up front*, while TDD instructions actively suppressed that step — the design "emerged from the sum of many locally-minimal decisions," locking in whatever shape the first test dictated, and untested behavior simply never got implemented. Ivett Ördög's theory: models trained on completed functions have an internal representation of requirements→code, not of the step-by-step process for getting there.

The article then walks TDD's classic goals one by one — avoiding tautological tests, testability, red-green regression proof, design-driving, YAGNI, localized feedback, and Kent Beck's fear management — asking of each whether it survives when the human leaves the loop. Most don't, or only probabilistically: a red test is only proof if someone checks *why* it went red, and an agent writing the test the same instant it plans the implementation feels none of the friction test-first exists to create.

The conclusion is a general one: prescribing *how* a model should work is not sustainable — monitor outcomes and give (automated) feedback instead. The author has stopped telling agents to write tests first, and gets regression quality via mutation testing, refactoring via structural review triggers and static analysis, and confidence from Ivett Ördög's Approved Scenarios approach — while acknowledging the eval is tiny, greenfield-only, and judge-dependent.
