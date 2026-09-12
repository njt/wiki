---
url: https://newsletter.getdx.com/p/five-studies-that-are-changing-how
title: "Five studies changing how I think about AI in software engineering"
author: Brian Houck
date_fetched: 2026-07-18
date_published: 2026-07-10
topics:
  - agent-coding-workflow
---

Brian Houck surveys five recent research papers that converge on a shared
story: AI is compressing the upstream work of software engineering (coding),
but the downstream systems — review, verification, integration, understanding
— haven't caught up.

**Heilman et al.** found a dose-response relationship between Copilot usage and
PR throughput: the highest-usage weeks yielded ~40% more completed PRs per
coding hour, with the strongest effects on larger PRs (7+ files).

**Demirer et al.** showed that coding gains attenuate dramatically on the path
to shipped software. Autocomplete tools drove ~40% more commits, interactive
agents ~140%, and autonomous agents ~180% — but the effect on releases topped
out at ~30%. The elasticity of substitution between AI output and human effort
was low (~0.25), suggesting AI and humans remain complements, not substitutes.

**Vella & Blincoe** ran a six-month longitudinal study and found a
"productivity-experience paradox": productivity perceptions stayed strongly
positive, but the share of engineers reporting worse developer experience
nearly doubled (14% → 27%). Flow state eroded most; feedback loops improved.
Productivity and DevEx appear to be decoupling over time.

**Choudhuri et al.** surveyed 860 Microsoft developers and mapped 22 AI tools
developers want beyond code generation. Three themes dominate: the
"right-shift" burden (more code to review, more incidents to debug),
verification tools (auto-assembled incident case files, business-logic-aware
PR review), and "bounded delegation" — developers want AI for assembly work
(docs, edge-case tests) but refuse to delegate core logic, architecture, or
critical decisions, even when AI could plausibly handle them.

**Storey** argues the technical-debt metaphor is insufficient. AI reduces
technical debt but accelerates *cognitive debt* (erosion of shared system
understanding) and *intent debt* (unclear or forgotten goals, constraints, and
rationale). These three debts compound: intent debt causes cognitive debt,
which causes technical debt, which amplifies cognitive debt. The prescription:
treat shared understanding as a first-class deliverable.

Houck's synthesis: per-engineer efficiency gains are real, but they don't
survive to shipped software at the same magnitude; the bottleneck has shifted
downstream; developer experience is more uneven than productivity numbers
suggest; and the deepest cost may be the unmeasured erosion of shared
understanding. Tools, metrics, and team design haven't caught up — that's
where the next several years of work will happen.
