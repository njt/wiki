# YAGNI

Martin Fowler's canonical 2015 bliki entry defining YAGNI ("You Aren't Gonna Need It") — the Extreme Programming principle that you should not build software capabilities until you actually need them. Fowler provides the most complete economic analysis of why building ahead is wrong, even when your predictions are correct. This is the ur-text that Kent Beck's later reframing ([[The Cost YAGNI Was Never About]]) builds upon.

---

## Key Quotes

> "Yagni originally is an acronym that stands for 'You Aren't Gonna Need It'. It is a mantra from ExtremeProgramming that's often used generally in agile software teams."

Fowler opens by grounding YAGNI in XP's Simple Design practice. The origin story — Kent Beck repeatedly telling Chet Hendrickson "you aren't going to need it" on the C3 project — captures the spirit: not "you'll never need it," but "you don't need it *now*, and that's reason enough not to build it."

> "Yagni argues against this, it says that since you won't need piracy pricing for six months you shouldn't build it until it's necessary."

The insurance example is the article's pedagogical engine. A startup in Minas Tirith (Fowler's Tolkienesque whimsy) sells shipping insurance. They're building storm-risk pricing now and know piracy-risk pricing is coming in six months. The temptation: build both now while you're in the pricing engine. YAGNI says no — wait four months, start in two. The example makes the costs concrete enough to refute the "cheaper to build now" intuition.

> "The first argument for yagni is that while we may now think we need this presumptive feature, it's likely that we will be wrong."

The headline argument, supported by Kohavi et al.'s finding that only ⅓ of features at Microsoft improved the metrics they were designed to improve. But Fowler doesn't stop here — he knows smart engineers will counter with "yeah, but what if we're right?" The rest of the article is the answer to that question.

> "If we'd instead put our energy into building the sales software for weather risks, we could have put a full storm risks feature into production and be generating revenue two months earlier."

The cost of delay. Even when your prediction is perfect and the Gondor Navy doesn't wipe out the pirates, you've traded revenue now for revenue later. This is the argument that professional software engineers — who are used to being right about technical predictions — most need to hear.

> "The extra complexity from having the piracy-pricing feature in the software might add a couple of weeks to how long it takes to build the storm insurance sales component."

The cost of carry — complexity that taxes every subsequent change. This cost compounds: every feature built between now and when the presumptive feature ships pays the tax. If the presumptive feature never ships, every feature pays the tax until someone removes it (and removing it has its own cost).

> "Development teams are always learning, both about their users and about their code base. … you often realize that a feature coded six months ago wasn't done the way you now realize it should be done."

The cost of repair — the most subtle cost. Even when you're right about *what* to build, you may be wrong about *how* to build it, because in six months you'll know things you can't know now. The presumptive feature accumulates [[TechnicalDebt]] before it ever reaches production.

> "Yagni only applies to capabilities built into the software to support a presumptive feature, it does not apply to effort to make the software easier to modify."

The crucial boundary condition. YAGNI is not an excuse to skip refactoring, testing, or CI/CD. Those are *enabling practices* — they make code malleable, and malleable code is the precondition for YAGNI to be safe. "Yagni has the curious property that it is both enabled by and enables evolutionary design."

> "Often that thought experiment is enough to convince them that it won't be significantly more expensive to add it later."

The practical technique Fowler uses when mentoring: imagine the refactoring you'd need to do later. Most of the time, deferring doesn't actually make things harder — it just *feels* like it will. And when it would be harder, the right answer isn't "build the whole thing now" — it's "add something minimal now that reduces the later cost without adding meaningful complexity."

> "Small yagni decisions like this fly under the radar of project planning. … a lot of small yagni decisions add up to significant reductions in complexity to a code base, while speeding up delivery of features that are needed more urgently."

YAGNI operates at every scale. The big decisions (should we build the piracy pricing engine now?) are visible and debated. The small ones (should I add a regex capture-group feature to my highlighting code before I have a use case?) accumulate invisibly. Fowler's highlighting-code example — he hasn't needed group-based highlighting yet, so he hasn't built it — is the micro-scale discipline that separates YAGNI practitioners from YAGNI lip-service.

---

## Key Themes

- **#concept YAGNI** — the principle that you shouldn't build capabilities until you need them; not thrift, but economic discipline about value timing
- **#concept Presumptive features** — Fowler's term for code that supports a feature not yet available for use; the unit of analysis for YAGNI decisions
- **#concept Four costs of building ahead** — cost of build (wasted effort on wrong guesses), cost of delay (revenue deferred), cost of carry (complexity tax on every subsequent change), cost of repair (fixing the right feature built wrong)
- **#concept Evolutionary design** — YAGNI's enabling partner: you can only safely defer features if your codebase is malleable enough to accept them later
- **#pattern Imagine the refactoring** — Fowler's mentoring technique: mentally simulate adding the capability later to test whether deferring actually makes things harder
- **#person Martin Fowler** — author, co-author of the Agile Manifesto, and the most influential explicator of software design practices of his generation
- **#person Kent Beck** — origin of the YAGNI phrase on the C3 project; creator of Extreme Programming
- **#concept Enabling practices** — refactoring, SelfTestingCode, ContinuousDelivery: the infrastructure that makes YAGNI safe rather than reckless
- **#comparison YAGNI vs. thrift** — the common confusion Fowler heads off: YAGNI is about value timing, not effort minimization; refactoring effort is never a YAGNI violation

---

## Critical Analysis

**Fowler's article is the definitive YAGNI reference for a reason.** It does three things that most "agile principle" explainers don't: (1) it takes the "but what if we're right?" objection seriously and answers it from first principles, (2) it names and taxonomizes the costs so they become discussable in design reviews, and (3) it draws clear boundaries around what YAGNI is *not* (an excuse to skip refactoring). The article has aged well because it's built on economics, not methodology fashion.

**The four-cost framework is the article's lasting contribution.** Before Fowler, YAGNI was mostly justified by the observation that requirements change. Fowler's innovation was showing that *even with perfect foresight*, building ahead loses money — through delay and carry costs. This is the same move Beck later made with options pricing in [[The Cost YAGNI Was Never About]], but Fowler got there first with cost-of-delay, and his framework is more operational: these are costs a team can actually estimate and discuss.

**The ⅔ failure rate is undersourced but directionally correct.** Fowler cites Kohavi et al.'s Microsoft study finding only ⅓ of features improved target metrics. That's a single-company study from an environment (massive-scale A/B testing on consumer products) that doesn't generalize cleanly to enterprise or infrastructure software. The number is rhetorically useful but shouldn't be treated as a universal constant. The stronger argument is the one Fowler doesn't lean on: even at 50/50 odds, the asymmetric downside (you can always build it later if you're wrong to defer, but you can't un-build it if you're wrong to build now) favors waiting.

**"Imagine the refactoring" is underdeveloped.** Fowler mentions it as a mentoring technique but doesn't elaborate. In practice, this is a skill that takes years to develop — novices consistently overestimate the cost of adding capabilities later and underestimate the cost of carrying unused abstractions. The technique would be stronger with concrete heuristics: when is "later" genuinely more expensive? Database migrations, API versioning, and security model changes are the canonical cases where deferring carries real cost. Fowler's silence on these edge cases leaves the technique feeling more like a Jedi mind trick than an engineering practice.

**The "enabling practices" boundary is both essential and fragile.** Fowler is right that YAGNI without refactoring discipline is reckless — you're deferring features into a codebase that can't absorb them. But this creates a bootstrapping problem: teams that lack refactoring discipline also can't safely practice YAGNI, which means they're stuck building ahead because their codebase is too rigid to do otherwise. The virtuous cycle (refactoring → malleable code → safe deferral → less complexity → easier refactoring) has no obvious entry point for teams caught in the vicious one. Fowler doesn't address this.

**The article predates AI code generation and is stronger for it.** Written in 2015, the article has no engagement with LLMs or coding agents. But its arguments are structural, not technological — the costs of building ahead don't change when code becomes cheaper to produce. In fact, as Beck argues in [[The Cost YAGNI Was Never About]], cheap generation amplifies the YAGNI trap: speculative structure is trivially cheap to produce, so the temptation is higher, and AI-generated code you didn't write is harder to understand and maintain (compounding the cost of carry). Fowler's 2015 framework is more relevant in the AI era, not less.

**The "small yagni" section is the most practically useful part of the article.** The big YAGNI decisions are debated in planning meetings. The small ones — adding a parameter you "know" you'll need, writing an abstraction for a second implementation that doesn't exist yet, stubbing out a method for next sprint — happen dozens of times a day, unremarked and ungoverned. Fowler's highlighting-code example (deferring the regex group feature until he has a concrete use case) is the discipline that separates teams with clean codebases from teams that don't know how they got so complex. This is also where YAGNI connects most directly to [[Ponytail]]'s decision ladder and the "lazy senior dev" persona.

**What's missing: when to *ignore* YAGNI.** Fowler briefly acknowledges that "there are times when applying yagni does cause a problem" but waves it away with availability bias — we remember the failures more than the successes. This is true but incomplete. There are genuine cases where building ahead is correct: platform investments that unlock multiple future features, shared abstractions where consistency matters more than deferral, and situations where the cost of building later is vastly higher (think database schema changes across a deployed fleet). Fowler's decision to bracket these cases as rare exceptions is defensible for a bliki entry but leaves the reader without a framework for identifying them.

---

## Connections

- [[The Cost YAGNI Was Never About]] — Kent Beck's 2026 reframing: YAGNI as options pricing and NPV, not thrift. Beck extends Fowler's cost-of-delay argument into formal price theory and adds the AI-era warning that cheap generation amplifies the trap
- [[The Lindy Effect in Software]] — "Lindy is YAGNI for architecture": the same logic of deferring commitment until you have better information, applied to technology choice
- [[The Economic Benefit of Refactoring]] — Fowler's 2026 experiment: 83% token reduction from refactoring agent-generated code. Closes the loop on his 2015 argument that refactoring enables YAGNI — and provides the first empirical measurement
- [[Ponytail]] — the "lazy senior dev" plugin that operationalizes YAGNI as a concrete decision ladder: YAGNI → stdlib → native → one-line before writing code. An implementation of Fowler's "imagine the refactoring" technique
- [[Martin Fowler and Kent Beck on Reinventing Software]] — 25 years after the Agile Manifesto, the two authors grapple with AI; Fowler's YAGNI article is the foundation that makes his skepticism credible
- [[Software Engineering Craft]] — hub for fundamentals that don't change; YAGNI is one of the few principles whose relevance has increased with AI
- [[TechnicalDebt]] — the cost-of-repair category in Fowler's framework; the connection between presumptive features and accumulated technical debt
- [[The Minimum Viable Unit of Saleable Software]] — buy-vs-build economics; Fowler's cost-of-delay is the same logic applied to feature timing
- [[99 Bottles of OOP]] — Sandi Metz's workbook on OO design as evidence-driven decision-making; the same discipline of not building ahead of the evidence
- [[Constraint Decay]] — agents perform worse with structural constraints; speculative abstractions are the constraints that YAGNI would have prevented

---
*Sources: [[raw/yagni-html]], [[summary/yagni-html]]*
*Last updated: 2026-08-08*
