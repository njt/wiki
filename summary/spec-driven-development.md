---
title: "Spec-Driven Development"
url: https://www.dbreunig.com/2026/03/04/the-spec-driven-development-triangle.html
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - specifications-as-the-product
  - agent-coding-workflow
---

Spec-driven development isn't a linear process (specs -> tests -> code) but a feedback loop where implementation informs and improves specifications. Introduces "Plumb," a tool to keep specs, tests, and code synchronized.

The Triangle Model: specs define requirements, tests validate behavior against specs, code implementation generates decisions that update the spec. "The spec defines what tests need to be written, and what code needs to be written. Tests validate the code. But the act of implementing code generates new decisions."

Historical parallels: Margaret Hamilton's Apollo work revealed complex codebases exceed individual cognitive capacity. The 1960s Software Crisis emerged from similar management challenges.

"When you can't see over your code, you can't oversee your code." (Hamilton's Law)

"Our current Software Crisis is our inability to manage complex codebases new models allow."

Plumb tool: git hook integration that intercepts commits to identify decisions, extracts decisions from code diffs and agent traces, generates decision logs with intent documentation, maps spec requirements to code and tests, prevents commits until decisions are reviewed/approved.

Core insight: "Implementing the code helps us improve our spec." Code implementation clarifies intent -- it's not just execution. The system must remain simple enough for developers to understand. Decision capture transforms code review from pure validation to intent documentation.