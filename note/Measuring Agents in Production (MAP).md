# Measuring Agents in Production (MAP)

The first systematic empirical study of LLM agents running in production: 20 in-depth interviews with deployment teams plus a survey of 306 practitioners (86 deployed systems across 26 domains), conducted by a Berkeley-led consortium from April to November 2025. Its headline is a paradox — the most sophisticated agent teams deliberately choose the least sophisticated-looking methods, because reliability in production is won by systems design, not model cleverness.

---

## What the data says

The headline numbers, all from deployed systems:

- **Bounded autonomy**: 68% of agents execute at most 10 steps before human intervention; 47% stop at 5. Research prototypes run far longer chains — autonomy gets consolidated during deployment, not expanded.
- **Prompting over training**: 70% use off-the-shelf models; only 5 of 20 case studies fine-tune at all, and just 1 uses RL.
- **Human evaluation dominates**: 74% rely on human-in-the-loop verification; LLM-as-a-judge (52%) is never used alone — judges score confidence and route low-confidence cases to experts.
- **Frameworks lose at scale**: 61% of surveyed teams use a framework (LangChain/LangGraph leads at 25%), but 85% of in-depth case studies build custom implementations with direct API calls, and two teams migrated off frameworks for production.
- **Latency doesn't matter (usually)**: 66% allow minutes-or-longer responses. Agents win 10x over human baselines even at that speed, so teams buy correctness with time.
- **Internal first**: 52% of agents serve internal employees — a deliberate choice so errors land where oversight is cheap and consequences are low.

## Key quotes

> "Practitioners achieve reliability through best practices in system-level design rather than model-level or algorithmic advances."

This is the paper's thesis and its most important contribution: it's evidence, not opinion. Read-only SRE agents, sandbox verification, wrapper APIs, role-based access — the containment toolkit is classic software engineering, and it works well enough to ship.

> "Teams deliberately choose simple, controllable methods not from lack of sophistication, but because they offer reliable agent performance and fast development cycles."

Crucial nuance against the "practitioners just don't know better" reading. The same teams track the research frontier and reject it — SFT and RL are "brittle to model upgrades, requiring costly retraining when versions change." Prompting survives model churn; weights don't.

> "Newer or more capable models do not guarantee improved agent performance."

Half the multi-model teams keep multiple models for one reason: scaffolds, prompts, and evals lock onto specific model behaviours, so upgrades can break deployed workflows. This is a quiet indictment of the field — the products are pinned to model snapshots the way old code was pinned to OS versions.

> "75% evaluate without benchmark sets, relying instead on A/B testing, user feedback, and production monitoring."

And when benchmarks exist, they're built from scratch per deployment — golden QA sets grown from real interactions. A wide-open research problem hiding in a survey statistic.

## Themes

#concept #pattern #tool #comparison

## Opinionated take

This is the paper the agent hype cycle needed: a large-n, practitioner-first counterweight to capability demos. Its strongest finding — that humans are not a temporary crutch but an *architectural component* — directly contradicts the autonomy maximalism of the research literature, and it does so with data from the teams that actually shipped. The 10-step ceiling is the most portable idea here: it quantifies what bounded autonomy means in practice and gives harness builders a concrete design target.

Two caveats. First, the sample skews to the leading edge — Berkeley-network organisations deploying agents successfully — so this documents the winners' playbook, not the median attempt (the authors say as much). Second, "reliability via constraint" may be a phase, not a principle: the same teams report plans for RL, and verification gaps (only coding agents get compile-and-test feedback) explain much of the conservatism. The paper is honest about both.

The finding most likely to age badly is the anti-fine-tuning one — it's contingent on model-update churn, and sample-efficient post-training that survives upgrades is exactly the research direction the authors call for. But the human-in-the-loop finding looks durable: in domains like insurance and healthcare, delayed real-world consequences mean *no* automated verification signal will exist for a long time.

## Related pages

- [[Reliability Is Outside the Model (O'Reilly)]] — the same thesis from the practitioner-essay side; MAP supplies the empirical base rate (70% off-the-shelf models, human verification at 74%) that essay argued from experience.
- [[The Human-in-the-Loop Fallacy (Choudary)]] — complicates it sharply: Choudary argues HITL is an organisational trap; MAP's data shows production teams treating it as a deliberate, enduring design principle. The tension is real and unresolved.
- [[LLM-as-a-Verifier]] — MAP describes the actual production pattern this page theorises: judge confidence scores routing low-confidence outputs to human experts.
- [[Shipping AI Agents to Production]] — a practitioner field guide whose advice (containment, internal rollout, human gates) MAP now backs with cross-industry numbers.

---
*Sources: [[raw/2512-04123v4]], [[summary/2512-04123v4]]*
*Last updated: 2026-11-20*
