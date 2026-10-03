---
url: https://www.oreilly.com/radar/evaluating-ai-generated-frontend-code-what-should-we-actually-test/
title: "Evaluating AI-Generated Frontend Code: What Should We Actually Test?"
author: Unnamed (O'Reilly Radar; author's note says views are their own, employer not named)
date_fetched: 2026-10-03
date_published: unknown (fetched 2026-10-03)
topics:
  - guardrails-and-feedback-loops
  - agent-coding-workflow
---

An O'Reilly Radar essay on how to evaluate UI code that AI generated from a short description. Its central claim: the first version of generated frontend code looks more complete than it is — it compiles, renders, and arrives with tests — and that polish makes reviewers *less* likely to slow down. The proposed discipline is "generated frontend code is a draft until the user behavior has been checked."

The essay lays out an evaluation ladder in increasing depth: (1) build/render are only the starting line; (2) check semantic structure — buttons as `<button>`, linked labels, native HTML over divs-plus-ARIA; (3) walk the keyboard path, since keyboard testing exposes a broken interaction model that mouse-path demos hide; (4) test focus behavior explicitly (focus into modals, back to the opener, to the first validation error); (5) exercise loading, error, and empty states, where generated code is weakest because it optimizes the happy path; (6) test full user flows with Playwright-style end-to-end tests, protecting the flows that matter rather than automating everything; (7) run automated accessibility checks but never treat a passing scan as an accessibility review.

Two arguments stand out. First, generated tests need review with the same care as generated code: they tend to document what the implementation already does, and the review question is "would this test fail if keyboard navigation broke?" — if not, it's protecting nothing. Second, evaluation should be matched to risk: a copy tweak is not a checkout flow, and the goal isn't a checklist per PR but clarity on what evidence is enough. The closing reframe is that AI's value in frontend is not faster code but the chance to move engineering attention toward evaluation, user behavior, and quality.
