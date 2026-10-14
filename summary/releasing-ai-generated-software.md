---
url: https://blog.ploeh.dk/2026/10/12/releasing-ai-generated-software/
title: "Releasing AI-generated software"
author: Mark Seemann
date_fetched: 2026-10-14
date_published: 2026-10-12
topics:
  - software-engineering-craft
  - guardrails-and-feedback-loops
---

Mark Seemann responds to Robert C. Martin's (Uncle Bob's) tweet that he no longer reads the code his agents write, instead surrounding them with "extreme constraints" — tests, mutation testing, quality metrics. Rather than doing tweet exegesis, Seemann asks what software engineering becomes when nobody reads the code, and where liability lands when it fails.

His first move is epistemic honesty: humans reading code was never foolproof either. Defects slip through code review; TDD's most common failure is the test you didn't write. So he accepts the self-driving-car framing — the system doesn't have to be perfect, just better than the human process — and sees no inherent barrier to a fully LLM-driven release process, with "teams" of agents playing the roles humans used to play.

The second half is the sharper part: liability. Lower defect frequency is not zero, and serious errors cost money or lives. Courts are unlikely to accept "the LLM wrote the bug" — the person who signed off on the release takes the fall. He illustrates with Danish approval of Tesla FSD, where the legally responsible human will predictably zone out as the system performs well, then face involuntary manslaughter when it eventually fails. His closing question: "Are you ready to go to prison for an AI?"
