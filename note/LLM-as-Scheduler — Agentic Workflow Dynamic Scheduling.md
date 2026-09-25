# LLM-as-Scheduler — Agentic Workflow Dynamic Scheduling

Xiang et al. (ACL 2026) propose LAS, a two-stage cascade — a lightweight gate scoring each agent's output, then an LLM scheduler using query features plus gate signals — that routes each query to the lightest workflow that will suffice. Reported: −43% tokens, −36% latency, ≤1.4pp accuracy drop versus a strong fixed workflow.

---

The paper's premise is one the practitioner literature keeps arriving at from the other direction: multi-agent pipelines with verification and testing steps buy accuracy, but "in practice, many queries do not need such heavy processing and can be handled well by a single strong agent." LAS's answer is per-query adaptive routing rather than a one-size workflow, with the scheduler itself an LLM reading both query features and cheap gate signals from agent outputs.

Key quotes and commentary:

> "While those extra steps can improve accuracy, they also increase latency and token costs."

This is the cost side of the orchestration tax — the same accounting that shows up in dark-factory cost audits and token-cost discussions. Verification is not free, and paying it on every query is a tax on the easy cases.

> "LAS cuts token usage by 43% and reduces end-to-end latency by more than 36%, while causing at most a 1.4 percentage-point drop in accuracy."

The interesting number is the *trade-off being made explicit*. Most orchestration writing either ignores the accuracy cost of routing or ignores the token cost of not routing; LAS names both axes. The gate-first design also echoes classical cascade/multi-stage classifier work: cheap discriminator first, expensive adjudicator only on the ambiguous residue.

Key themes: #concept (adaptive workflow routing), #pattern (two-stage cascade: gate → scheduler), #tool (LLM-as-a-router).

**Opinionated take.** The genuinely valuable idea here is treating *workflow choice* as a first-class routing decision rather than baking one pipeline shape into the product. But note what the evaluation hides: a 1.4pp accuracy drop is measured against the fixed workflow's average — the interesting question is the *tail*, the hard queries the gate waves through and the easy ones it over-routes. Cascade systems fail interestingly at the boundary, and the abstract says nothing about error analysis there. There's also a recursion joke waiting to happen: an LLM scheduler deciding how many agents to spend is itself an agent whose output should probably be gated.

How this source relates to the rest of the wiki:

- It strengthens [[The Advisor Strategy]]: both are "don't pay the expensive model unless you must" schemes — Anthropic's advisor-executor pattern routes at the *model* level, LAS routes at the *workflow* level; the cascade shape is identical, only the granularity differs.
- It nuances [[Dynamic Workflows in Claude Code]]: that piece has Claude author custom orchestration harnesses per task class; LAS automates the decision those patterns leave implicit, and its gate signals are a cheaper mechanism than full fan-out-then-judge.
- It complicates [[Managing AI Coding Costs at Scale]]: Databricks' playbook treats intelligent routing as one lever among many; this paper is the academic case that routing *is* the lever, quantifying the token/latency savings a router can buy at a named accuracy price.
- It sits oddly against the heavy-verification culture of [[Agentic Code Review]]: if most queries don't need multi-step verification, the open question is what fraction of *code changes* are similarly routable to light review — which is exactly where the 1.4pp tail lives.

---
*Sources: [[raw/2026-acl-long-581]], [[summary/2026-acl-long-581]]*
*Last updated: 2026-09-25*
