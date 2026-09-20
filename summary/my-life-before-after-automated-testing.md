---
url: https://blog.danlew.net/2026/09/15/my-life-before-after-automated-testing/
title: "My life before & after automated testing"
author: Dan Lew
date_fetched: 2026-09-20
date_published: 2026-09-15
topics:
  - software-engineering-craft
  - guardrails-and-feedback-loops
---

Dan Lew makes the classic case for automated testing through a single worked example — adding job cancellation to a work queue — told twice: once as he used to develop (manual testing at major checkpoints, missed concurrency bugs, QA catching the fallout, a coworker's unrelated change silently breaking the feature a month later) and once as he develops now (tests written first against empty function stubs, run constantly during implementation, catching the concurrency gap and the coworker's breakage before code review).

The two benefits he isolates are tight feedback loops during development — the cost of manual testing means you only check your work every few hours, and late feedback makes course correction expensive — and cheap, consistent regression testing, which amortises its upfront cost over the feature's lifetime and protects everyone who touches the code afterwards, not just the original author.

The second half is refreshingly honest about failure modes: you still need manual testing to validate the tests themselves, flaky tests are worse than none, retroactive test-writing for existing code forfeits most of the value, and combinatorial integration-test explosions should be avoided. Adoption is framed as incremental, not all-or-nothing.
