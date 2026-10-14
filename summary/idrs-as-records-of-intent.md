---
url: https://wrla.ch/log/2026/10/idrs-as-records-of-intent/
title: "IDRs as records of intent"
author: Will Lachance
date_fetched: 2026-10-14
date_published: 2026-10-10
topics:
  - specifications-as-the-product
  - agent-coding-workflow
---

Will Lachance reflects on IDRs (Implementation Decision Records, a pun on Architecture Decision Records) that he created for two purposes: getting better output out of LLMs, and leaving a durable record of decisions made during implementation.

His core realisation: left alone with a vague prompt, an LLM will happily write an IDR that merely describes what it planned to implement anyway. He shows a bad example — a DuckDB serialization IDR whose problem statement is vague and whose context is "almost inscrutable LLM" filler — that reads as a lossy English restatement of the diff, arguably worse than the diff itself.

The counter-example is a short IDR for publishing repo-parser to PyPI from GitHub Actions: a problem statement describing the actual "itch", terse goals, explicit non-goals (no automated version bumping), and a proposed solution pointing at the official packaging guide and a known-good implementation. Every sentence steers the agent toward the desired outcome, acts as a clear guardrail rather than misinterpretable instructions, and is what he actually wants to find when revisiting the repo later. If more detail is needed, look at the diff.

The prescription: an effective IDR must keep returning to **intent** — the itch and the decisions taken to support it — otherwise it is just a long-winded way of restating the diff in English.
