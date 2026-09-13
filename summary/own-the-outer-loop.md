---
url: https://www.oreilly.com/radar/own-the-outer-loop/
title: "Own the Outer Loop"
author: Addy Osmani
date_fetched: 2026-09-13
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Addy Osmani's essay on accountability in agentic engineering, republished on O'Reilly Radar from his blog (undated, but it cites GitLab's June 2026 AI accountability research and Sonar's 2026 State of Code report, so it postdates mid-2026). The thesis: agents have leverage, and leverage creates obligations — engineers must own the outer loop, the accountability layer around agent systems.

The vocabulary does the work. An agent is a model plus a harness; the loop is investigate, implement, verify, repeat; a factory is loops at scale. Three terms anchor the argument: **Quality** (the checks installed before the system runs, which produce evidence), **Verdict** (the human production decision that evidence licenses — ship, block, redirect, narrow, add a guardrail, or reject: "The model may write the line, but the Verdict is mine"), and **Answerability** (the guarantee that if someone asks, you can explain why). Inside the system there is only capability; outside it there is only agency — the agency to decide, verify, approve, and own.

The evidence base: Sonar's 2026 State of Code survey puts AI-generated or significantly AI-assisted committed code at 42% and still growing, while GitLab's June 2026 research finds review and validation are the bottleneck and governance typically happens *after* code creation, once the risk is already accepted. Generation got cheap; review, validation, understanding, and maintenance did not — a trust-verification gap.

Three hidden costs get named and quantified. **Cognitive surrender**: per a Wharton study, when the AI was wrong, nearly three-quarters of people accepted the answer anyway — and felt more confident than they would have without AI. **Cognitive debt**: an Anthropic randomized trial found engineers who leaned on AI scored 17 percentage points lower on a comprehension quiz (50% vs 67%). **The orchestration tax**: spinning up many agents is easy; your cognitive bandwidth doesn't parallelize with them.

The operating model: treat quality as backpressure — grant agents only as much autonomy as the backpressure can regulate, trusting ordinary engineering signals (type checks, tests, hooks, sandbox limits, audit logs) to keep the system honest. Humans belong in four loops, not the inner loop: the **constraints loop** (what inputs, architectures, invariants?), the **sampling loop** (how much output to review?), the **audit loop** (what evidence to keep?), and the **ownership loop** (what part of the production boundary to own?). Brownfield is the frontier — legacy behavior "lives in the scars," and stewardship means turning implicit knowledge into explicit constraints, formalized into tests and specs. Roles unbundle (prototype, build, sweep, grow, maintain), careers ride on alpha, decay, and taste, and the closing proposal is an **accountability contract** per change: the checklist understood at acceptance, the evidence, the accountable person, the system status. The bottleneck moves from "Can we build this?" to "Should this exist? Can we answer for it?"
