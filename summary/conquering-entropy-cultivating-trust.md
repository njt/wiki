---
url: https://kaeruct.github.io/posts/2026/09/06/conquering-entropy-cultivating-trust/
title: Conquering Entropy: Cultivating Trust
author: kaeruct
date: 2026-09-06
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A practitioner's essay (part of the "Conquering Entropy" series) arguing that the real problem with AI-generated code is not the code but trust. The author walks through a chain of trust questions — the ticket, the PR, the agent's implementation, the test suite, CI/CD, observability, the AI SRE, even GitHub itself — and concludes that trust is hard to earn and easy to lose, so engineering teams must deliberately cultivate it.

The core prescription is a culture of accountability: engineers should be told "you are accountable for what you ship; if it breaks production and you authored the PR, you should be there to fix it" — and, in exchange, given the agency to use whatever methods (including AI agents) they prefer, plus the room to install quality measures so the codebase doesn't degrade.

The author lists seven practices that enable this accountability: clear coding guidelines (for both humans and agents), deterministic tooling (typed languages, linters, dead-code and security scans, CI/CD), enforcing small/reviewable PRs, human-defined tests ("coding agents can implement the tests, but they should be defined by humans"), taste in product, throwaway prototyping (try three radically different approaches), and focusing on the outcome rather than the code ("sometimes the code is not the result").

A recurring, deflationary theme: perfect quality was never the bar. "Even before coding agents most code was already a buggy mess," so "reasonable" quality delivered with speed is an acceptable trade — the discipline is about keeping slop out without obsessing over perfection.
