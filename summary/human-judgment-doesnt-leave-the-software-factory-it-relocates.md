---
url: https://www.oreilly.com/radar/human-judgment-doesnt-leave-the-software-factory-it-relocates/
title: "Human Judgment Doesn't Leave the Software Factory — It Relocates"
author: Addy Osmani (originally on Elevate; republished on O'Reilly Radar)
date_fetched: 2026-09-29
date_published: 2026-09 (late)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Osmani's field report on running a "lights-on" software factory in practice, built around one thesis: as the share of code physically typed by humans falls, human *ownership* doesn't have to fall with it — judgment gets relocated, not removed. Upstream on intent, system shape, and the quality bar; downstream wherever automated back-pressure goes weak.

The practical core: you can get surprisingly far with a stock harness (Claude Code or Codex, multiple sessions, specs with verification baked in), and a factory only earns its keep when work needs to be repeatable and event-driven. When it does, the mechanical problems are queueing, locking, and handoff — Warp's four-state issue triage (ready-to-implement, ready-to-spec, needs-info, wait-to-implement) does triple duty as queue, lock, and human parking spot. Vercel's run taxonomy (success / flawed / blocked / manual, only "success" ships) is the same idea applied to runs.

The second half is about the cost side that factory advocates underplay. Verification is a budget, thought of like a performance budget: fast checks (lint, types) early, heavy checks (mutation, browser, security) near the PR. In his 82-minute sample run, a 7-minute no-rejection task sat beside a 56-minute one with two rejections and a human decision — taxonomy without per-stage timing hides what finding out cost. And parallel sessions create comprehension debt: after merging a favoriting feature whose tests were green, he returned to the code days later and couldn't explain how it worked. His prescription: have agents record their trajectory and reasoning, not just the diff, because "code often preserves a decision that was made, but not why."
