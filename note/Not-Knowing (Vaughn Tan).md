# Not-Knowing (Vaughn Tan)

Vaughn Tan's multi-year essay series building a diagnostic framework for uncertainty. His core argument: institutions fail at navigating uncertainty because they apply risk-management tools to situations those tools were never designed for. Formal risk — where actions, outcomes, and probabilities are all knowable — "almost never exists outside a casino." True uncertainty has four distinct types (actions, outcomes, causation, value), each requiring different responses. Misdiagnosis produces worse results than having no framework at all. The series builds from motivation through deep foundations to a practical four-compartment toolkit.

---

## Key Quotes

> "Applying the wrong framework produces worse decisions than having no framework at all"

The strongest version of the argument. This isn't "risk tools are suboptimal" — it's that they're actively destructive when misapplied. The parallel in agent engineering is applying prompt-level guardrails where deterministic enforcement is needed ([[Guardrails and Feedback Loops]], [[claude-ctrl]]).

> "The real skill is accurate diagnosis of what type of not-knowing you face before reaching for any tool at all"

Tan's version of "slow down to go fast" — the diagnostic step is the work, not a preamble to it. Same sensibility as [[Slowing the Fuck Down]] and the pre-read phase in [[Before Reading Code]].

> The formal risk mindset "prevent[s] you from seeing non-risk unknowns at all"

This is the insidious part. The framework doesn't just fail — it renders the failure invisible. Tan uses the Bank of England's macroeconomic modeling and WHO's Feb 2020 COVID response as exhibits. The AI parallel: believing a benchmark measures capability when it only measures benchmark-proximity ([[Benchmark Exploitation]]).

> "The only thing definitionally true of all innovation... is that it requires not-knowing"

Innovation management that tries to eliminate uncertainty is trying to eliminate the thing it claims to produce. This inverts the standard business-school framing. Most "innovation" programs copy big-company practices designed for formal risk, guaranteeing mediocrity at best.

> "It feels right"

Tan's explanation for why the formal risk mindset persists: it's emotionally satisfying. Quantification gives the illusion of control. This is the same dynamic behind [[Cognitive Debt]] — the dopamine of production eclipses the ability to judge the product.

---

## Key Themes

#concept #pattern #decision-making #uncertainty

The **four-type taxonomy** is the framework's intellectual core. Actions, outcomes, causation, and value each produce distinct forms of not-knowing with distinct sources and remedies. The taxonomy resists the urge to collapse them into one category (the "risk" bucket), which is the standard institutional move.

The **diagnostic-first discipline** is the framework's practical contribution. Tan insists that tool selection follows diagnosis, not the reverse. This parallels the harness-engineering insight that you design enforcement mechanisms around failure modes, not around intentions ([[Harness Engineering]]).

**Generative uncertainty** is Tan's term for uncertainty deliberately designed into a system — clear guardrails, encouragement of emergence, flexible support. Contrasts with bad uncertainty (the kind that produces anxiety and paralysis). This maps to the distinction in agent design between giving an agent freedom within a sandbox vs. constraining it with prompts ([[yolo-cage]], [[A Deep Dive on Agent Sandboxes]]).

The **fountain effect** describes how the four types trigger each other unpredictably over time. Understanding causal not-knowing changes what actions you consider; experimenting with new actions reveals new outcomes; new outcomes shift value assessments. Moderna and RNA therapeutics are Tan's example of an organization that deliberately leveraged this self-generativity rather than trying to suppress it.

---

## Critical Analysis

Tan's framework is genuinely useful, but it has a blind spot: it assumes institutions *want* to navigate uncertainty well. Many don't. The formal risk mindset persists not just because it "feels right" but because it serves organizational functions — it distributes accountability, justifies budgets, and provides cover when things go wrong. A risk register that says "we don't know" is a risk register that gets you fired. Tan gestures at this in the "social shame" essay but never fully confronts the incentive structure that rewards false confidence.

The four-type taxonomy is cleaner than reality. In practice, not-knowing types are entangled. Tan acknowledges this with the fountain effect, but the diagnostic-first approach still asks you to classify before acting. There's an unresolved tension: if the types trigger each other unpredictably, then accurate diagnosis may be impossible at the moment of decision. The framework is strongest as a retrospective tool and weaker as a prospective one.

The series' greatest contribution might be the vocabulary itself. Giving people words for "not-knowing about value" vs. "not-knowing about causation" makes it possible to have conversations that the word "risk" forecloses. This is the same move [[A Non-Anthropomorphized View of LLMs]] makes — precise language creates the possibility of precise thinking.

The woodworking metaphor (Krenov and Maloof studying material irregularities rather than trying to remove them) is the essay series' best image and its most honest argument: not-knowing isn't just something to tolerate, it's the medium. You don't make furniture despite the wood's irregularities; you make furniture *with* them. The same is true of uncertainty in any complex system.

The cross-link to [[poietic]] is non-trivial. Tan co-founded poietic with Erik Garrison, building a task coordination system (wg) for human-machine collaboration. The not-knowing framework isn't just academic — it's the theoretical substrate for how poietic handles coordination under uncertainty. The dependency graph isn't a project plan; it's a structure for making not-knowing legible.

---

- [[The Art of Decision-Making]] — Rothman's essay extends Tan's taxonomy from the other direction: not just "what type of not-knowing is this?" but "what if the choice changes who I am such that my current values can't evaluate it?" The philosophers Rothman surveys (Ullmann-Margalit, Paul, Callard) are working the same problem as Tan — the inadequacy of decision theory for choices where values aren't stable
- [[How to Choose a Subproblem]] — Choosing a subproblem is itself a diagnosis problem: the author's red flags (too vague, too hard, too easy, too useless) and the "plan of attack" as a difficulty gauge are a diagnostic-first discipline for research, cousin to Tan's four-type taxonomy.

---
*Sources: [[summary/notknowing]]*
*Last updated: 2026-08-01*
