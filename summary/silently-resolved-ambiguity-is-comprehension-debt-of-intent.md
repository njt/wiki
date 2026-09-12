---
url: https://blog.bl00cyb.org/2026/08/silently-resolved-ambiguity-is-comprehension-debt-of-intent/
title: Silently Resolved Ambiguity Is Comprehension Debt of Intent
author: bl00cyb
date_fetched: 2026-08-25
published: 2026-08
topics:
  - agent-coding-workflow
---

A short blog post naming a specific, senior form of debt in AI-assisted development: **comprehension debt of intent**. Where comprehension debt of *implementation* is the familiar gap between shipped code and understood code, comprehension debt of *intent* is what accrues when an agent resolves ambiguity silently — an issue underdetermines a behavior, the agent picks something statistically probable, and nothing surfaces that a decision was made at all. It is "a debt entry with no signature."

The author's remedy is not "understand every change" (too slow — "spicy autocomplete") and not pure vibe coding (gets shockingly far, then builds something that can't be changed while maintaining trajectory). The middle path is intentional decision points backed by data: decisions made with awareness reduce debt but interrupt flow ("incur naps"); decisions delegated to the agent preserve the nap but add debt. So you must *decide which decisions you should be making*, and set your system up to surface them to you alongside relevant data.

The open design problem is the tripwire: how do you know when an ambiguity needed authored intent rather than an agent's silent guess — without routing *everything* to the human? The author points to provenance as the catch: an outcome should tie back to something you defined, a verifiable reference, or probability/LLM. Knowing a decision's provenance lets you improve the model through instruction, reducing the debt going forward. Ethan Zuckerman is named as starting to take on provenance.

The post closes by teasing the "golden egg" of the thread: how to build reliable, deterministic tools using stochastic parrots.
