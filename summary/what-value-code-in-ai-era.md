---
url: https://jerf.org/iri/post/2026/what_value_code_in_ai_era/
title: "How Do We Value Code In A World Of Free Code?"
author: jerf (John E. R. Ferrell-style pseudonymous Go community figure)
date_fetched: 2026-10-03
date_published: 2026 (2026)
topics:
  - software-engineering-craft
  - ai-product-and-business
---

An open-ended essay by jerf (moderator of /r/golang) on what makes code valuable when producing it is trending toward free. The core thesis: a line of code is still a liability and a capability is still an asset — and AI sharpens rather than dissolves that distinction, because AIs are finite and cognitive resources are spent on whatever code exists.

Key moves:

- **LoC as a liability** — management culture will adopt LOC metrics again because the chart goes up, and the correction will come only after failed projects sacrifice themselves (possibly Windows).
- **AIs are finite** — an infinite AI wouldn't care about code volume; finite ones do better with code that costs less cognition to understand. Invariants and typed domain values resist this: jerf can't get AIs to create or preserve them; they "spray code like a firefighter sprays foam," and AI is even better at that bad style than humans were.
- **Tested against the real world is the value** — a bespoke accounting system summoned in ten minutes isn't a solution because it hasn't been dragged through reality (courts, users, time). SaaS survives and strengthens: value concentrates in codebases with the most real-world exposure, and the gap between them and instant bespoke systems grows, not shrinks.
- **Moderation insight** — the /r/golang flood of AI side-projects is valueless not because of how they were made but because they lack real-world testing; the right rubric is exposure to reality, not effort or AI-percentage.

The coda predicts AI-facing APIs designed to be machine-consumed and AI needed to manage the complications AI-generated code creates. Ends with three footnotes, including a plea that AI-written code stay human-legible.
