# The PM's Playbook for Shipping AI Features

Gaurav Savla's practical field manual for the gap between AI demo magic and production reality: latency budgets, four-level fallback chains, quality pyramids, the statistical traps of A/B testing nondeterministic systems, and why "we'll harden it later" is the sentence that kills AI features. The thesis: most AI feature failures aren't model problems — they're engineering discipline problems wearing an ML costume.

---

## Key Quotes and Ideas

> "Most often than not, it's not a model problem but an engineering discipline problem."

This is the article's thesis and its best single line. The industry has spent two years blaming models for failures that are really failures of latency budgeting, fallback design, drift monitoring, and evaluation rigor. Demos prove the model *can* do it; production proves the team didn't build the scaffolding.

> "Degraded beats dead every time."

Savla's mantra for fallback design. A template-driven response that's merely adequate beats a blank screen or an error. This is the #pattern that separates production AI from demo AI: the willingness to plan for partial failure. The four-level hierarchy (model → cache → template → omission) is worth memorizing.

> "Ten user interviews will surface failure modes that no amount of statistical analysis will catch."

Counterintuitive in a data-driven culture, but correct. Nondeterministic outputs create "intratreatment variance" that inflates required A/B test sample sizes by 3–5x. Most teams run AI experiments with normal sample size assumptions and "are probably looking at noise and calling it signal." Savla's prescription — Bayesian methods, 2–3x traffic budgets, qualitative pairing — is the most underrated section of the article.

> "You won't know it's broken until your users tell you, and by then they're angry."

On model drift: the three types (data drift, provider drift, evaluation drift) are well-known, but the framing as "slow, invisible rot" captures the operational reality better than most ML literature. The GPT-4 March–June 2023 behavior shift is the canonical example, and his fix — pin model versions — is the minimum viable defense.

> "Prompt engineering is software engineering."

Becoming a cliché, but Savla earns it with specifics: version control, regression suites of 200–500 cases, canary rollouts with 72-hour comparison windows, parameterized templates with defined injection points. This is the article at its best — taking a fuzzy intuition ("treat prompts like code") and turning it into an operational checklist.

---

## Key Themes

- **#concept** Latency budgets — p50 lies, cold starts are 10x worse, streaming as a perception hack, and three interaction types with hard budgets (sync <1s, progressive <5s, async <20s)
- **#pattern** Four-level fallback hierarchy — model → cache → template → omission. Each level degrades gracefully, transitions should be invisible, users should never see an unhandled AI failure
- **#concept** Quality pyramid — Safety (binary, nonnegotiable) → Factual correctness (domain-specific) → Usefulness (user-centered metrics) → Delight (experimental, hardest to measure)
- **#pattern** A/B testing for nondeterministic systems — intratreatment variance inflates sample size 3–5x, Bayesian methods preferred, qualitative research as essential complement
- **#concept** Model drift taxonomy — data drift (world changes), provider drift (APIs change silently), evaluation drift (metrics go stale). Daily/weekly/monthly monitoring cadence
- **#concept** Prompt engineering as production discipline — regression suites, canary rollouts, version pinning, parameterized templates, 72-hour stabilization windows
- **#tool** Model-as-judge — viable middle ground between automated evals and human evaluation; validate quarterly against human judgment targeting 85% agreement

---

## Critical Analysis

**What it gets right:** The article is refreshingly free of AI mysticism. Every recommendation is operational — specific numbers, specific cadences, specific architectures. The latency budgets are concrete (sub-200ms for consumer products, p90 not p50). The fallback hierarchy is actionable (here's what to do when each layer fails). The A/B testing section is statistically literate in a way most product writing isn't. This is what "production engineering for AI" should look like.

**What's missing:** The article doesn't grapple with the organizational cost of its own advice. A four-level fallback hierarchy means 4x the test surface. Daily evals on 1–5% of production traffic costs money. Weekly human evaluation of 100–500 examples at $15–30 each is $1,500–$15,000 per week. The checklist is correct, but implementing it requires an organizational commitment to quality that most teams don't have — and the article doesn't help PMs make that case to leadership.

**The uncomfortable truth:** Many of these practices (canary rollouts, regression suites, latency budgets, fallback chains) are standard software engineering. They're not novel. The article's real contribution is pointing out that the AI industry has been skipping them — shipping probabilistic systems with deterministic-system quality practices, then being surprised when things break. Savla's message is basically: *the rules of production engineering still apply, and AI features don't get a free pass.*

**The model question dodged:** "Not a model problem but an engineering discipline problem" is partially true and partially convenient. Better models DO reduce hallucination rates and latency. Engineering discipline is necessary but not sufficient — the best fallback hierarchy in the world won't save you if the base model has a 15% hallucination rate. The article subtly blames teams for what is, in part, a technology maturity issue.

**Bottom line:** Best read as a checklist for PMs who've only ever shipped deterministic features and are now being asked to ship AI. The advice is solid, specific, and actionable. Just budget for the infrastructure cost before you commit to the timeline.

---

## See Also

- [[Guardrails and Feedback Loops]] — Hub for eval, testing, and quality practices
- [[The Agentic Product Standard v2.0]] — Canonical framework for agent capability levels and production readiness
- [[Demystifying Evals for AI Agents]] — Deep dive on evaluation frameworks for AI systems
- [[Running an AI-Native Engineering Org]] — Fiona Fung's field report: the org structure that supports production AI
- [[Smart Models Dumb Pipes]] — Architectural pattern complementary to Savla's fallback design
- [[Writing Code vs. Shipping Code]] — The empirical evidence that shipping is harder than building
- [[Software Engineering Craft]] — Fundamentals that AI features still depend on
- [[How Hightouch Built Their Long-Running Agent Harness]] — Production context management as the real engineering challenge
- [[Specifications as the Product]] — Savla's prompt engineering advice is a spec-first argument in disguise
- [[Not-Knowing (Vaughn Tan)]] — Uncertainty taxonomy; AI nondeterminism fits Tan's "genuine unknowns" category

---
*Sources: [[summary/the-pms-playbook-for-shipping-ai-features]]*
*Last updated: 2026-06-15*
