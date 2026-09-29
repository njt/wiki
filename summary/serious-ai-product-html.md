---
url: https://blog.glyph.im/2026/09/serious-ai-product.html
title: "What a Serious AI Product Would Look Like"
author: Glyph (Glyph Lefkowitz)
date_fetched: 2026-09-29
date_published: 2026-09
topics:
  - ai-product-and-business
  - guardrails-and-feedback-loops
---

Glyph Lefkowitz argues that current AI products "do not appear to take their own premises seriously": every chatbot admits in fine print that it makes mistakes, yet none ships the tools that admission implies. His yardstick is simple — if a product tells you to double-check its output but gives you zero affordances for checking, it is not a serious tool for problem-solving.

The essay then enumerates the features a serious product would have. Verification as a first-class feature: a checkbox and notes column next to every claim, not a legal disclaimer; for coding, review affordances before diffs hit expensive test compute. Citations presented as the primary artifact — verbatim program-extracted quotations with metadata, front and center, with AI summary de-emphasized underneath, rather than 16-pixel icons. No first-person voice, no apologies. Task-specific structured UIs instead of trusting natural language to take destructive actions. Data provenance indicators distinguishing tool/API-derived input from hallucinated output, with programmatic verification shown visibly. User control of reproducibility (temperature visibility, replay, forking) rather than the illusion of one authoritative answer. Context visibility — showing the user the context window, compactions, and harness-generated prompts — since context management is "the premier engineering difficulty." And a sandbox that actually works: harness-level filesystem scoping, mandatory snapshots, no auto-mode, batch plan approval, and execution against mock services.

The closing twist lands on organisations and the labs themselves. Deploying orgs need shift rotations against vigilance decrement (aviation-style rest rules), deliberate skill practice to offset skill atrophy (the dockworker/crane analogy), and mental-health resources with a usage "dosimeter." The final accusation is the sharpest one: after years and hundreds of billions of dollars, shipping none of these features suggests the labs know that honest measurement tools would reveal AI tools provide, in aggregate, zero net value — and Glyph states the null hypothesis outright while inviting to be proven wrong.
