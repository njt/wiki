# The Coming Need for Formal Specification

Ben Congdon's argument that AI code generation inverts the economics of software engineering: when code is cheap to produce but expensive to review, the bottleneck moves upstream to specification — and formal methods become the only systematic answer. The article connects Kleppmann's prediction about AI-driven formal verification with Congdon's own observations about what tasks feel safe to delegate to LLMs.

---

## Key Quotes

> "Code is a poor map for the territory of what a system should actually do."

Congdon draws on the map–territory distinction to make a point that echoes across this wiki: implementation detail is the wrong level of abstraction for reasoning about system behavior. Knowing the molecular structure of a car doesn't help you estimate braking distance. This is the same insight that drives [[Specifications as the Product]] — code is the wrong artifact to treat as durable.

> "Writing tests was among the first tasks I felt comfortable delegating because unit tests follow predictable patterns in-distribution for models."

This is a sharp counterpoint to the 2022 prediction that engineers would "transition from writing code to writing tests." Instead, Congdon found that tests were *easier* to delegate than implementation. The real shift is further upstream: specification and design intent. This aligns with [[ThoughtWorks Future of Software Engineering Retreat]]'s finding that rigor migrates to five destinations, not just testing.

> "As the cost of producing code trends towards zero and the cost of reviewing code lags further and further behind, we need systematic tooling for dealing with this mismatch."

The economic argument in one sentence. Review doesn't scale linearly with generation — it's the bottleneck [[Cognitive Debt]] diagnoses: code becomes cheaper to produce than to perceive. Congdon's answer is formal methods, not more review.

> Kleppmann on seL4: "8,700 lines of C code, but proving it correct required 20 person-years and 200,000 lines of Isabelle code."

The ratio is staggering: 23 lines of proof for every line of code. Kleppmann's prediction — that AI will invert this by making proof generation the cheap part — is either visionary or wishful. The seL4 team's effort wasn't just about proof volume; it was about understanding the system well enough to formalize it. [[Correct by Construction]] shares the philosophy but not the tooling: structural correctness through anchors/attributes/links vs. formal verification.

> "You can probably fit every TLA+ expert in the world in a large schoolbus." — Hillel Wayne

The talent bottleneck is real. Even if AI lowers the mechanical cost of writing TLA+ or Rocq, the number of humans who can judge whether a specification is *correct* remains tiny. This is the same adoption problem [[Spec-Driven Development]] and [[OpenSpec]] face: specs only work if someone reads them.

> "Undergraduate Computer Science programs should allocate some of their curriculum to formal verification."

The education take is the most radical part of the argument. If AI handles implementation, CS education shifts from *how to build* to *how to specify what should be built and verify it was built correctly*. This parallels [[The Next Two Years of Software Engineering]]'s finding that junior roles are already shifting.

---

## Key Themes

#spec-driven #formal-verification #tla+ #rocq #review-bottleneck #cs-education #map-territory

---

## Critical Analysis

Congdon has identified the right bottleneck but prescribed the wrong tool for most teams.

The core diagnosis is correct and increasingly consensus: AI makes code generation cheap, which makes specification the scarce resource. This wiki has documented the same inversion from multiple angles — [[Specifications as the Product]], [[Spec-Driven Development]], [[Write Only Code]], [[Cognitive Debt]]. Congdon's contribution is connecting this directly to Kleppmann's formal verification thesis, which adds a specific (and controversial) destination for where the rigor lands.

**Where the argument is strong:** The economic framing is crisp. If code costs approach zero and review costs don't, something has to give. The map–territory analogy is genuinely useful for explaining why implementation detail is the wrong level of abstraction. And the education argument — that CS curricula need to shift toward specification and verification — is provocative in the right way, even if the timeline is optimistic.

**Where it's weak:** Congdon conflates two very different activities. *Specification* (describing what a system should do, in English or structured formats) and *formal verification* (proving properties about a system using mathematical logic) are not the same thing. The gap between them is the gap between a PRD and a TLA+ model. Most of the benefits Congdon describes — reducing ambiguity, catching design errors early, creating a durable artifact that outlasts the code — come from specification, not formal verification. You can write a good spec in markdown with [[Specsmaxxing]]'s YAML format or [[How to Write a Good Spec for Agents]]' five principles. You cannot casually verify it in Rocq.

The jump from "we need better specifications" to "we need formal verification" is a leap the article makes without defending. The seL4 example cuts both ways: yes, AI might make proof generation cheaper, but the 23:1 proof-to-code ratio suggests the problem is deeper than tooling. Formal verification requires specifying what "correct" means with mathematical precision. That's a human judgment problem, not a generation problem. LLMs can generate plausible-looking TLA+ the same way they generate plausible-looking code — but for formal verification, plausible is worthless. The spec has to be *right*.

**What's missing:** Any mention of the current tools that address the gap between English specs and formal verification without requiring a PhD. [[Guardrails and Feedback Loops]] argues that linters and deterministic checks beat prompts — the same principle applies here. Property-based testing, contract checking, and schema validation occupy the practical middle ground Congdon skips over. You don't need to prove your system correct in Rocq to get 80% of the benefit; you need to specify its behavior clearly enough that a linter can catch violations.

The article also ignores the social dimension. Formal methods have been "about to go mainstream" for 40 years. The barrier has never been tooling cost — it's that formal specification rewards precision work that most organizations don't value because the bugs it prevents are hypothetical until they're catastrophic. AI doesn't change that incentive structure. It just makes the tools cheaper to use, which is necessary but not sufficient.

**Bottom line:** Read this for the bottleneck diagnosis, not the prescription. The coming need is for specification — clear, testable, living documents of intent. Whether that specification is formal or structured or narrative depends on the domain. Congdon is right that the economics have shifted. He's probably wrong that the answer involves Rocq for most teams.

---

## Cross-Links

- [[Specifications as the Product]] — the synthesis page that makes Congdon's thesis the organizing principle of this wiki
- [[Spec-Driven Development]] — Breunig's spec-test-code triangle; the middle ground Congdon skips
- [[Cognitive Debt]] — the review bottleneck Congdon frames as the driver for formal methods
- [[Write Only Code]] — if nobody reads the code, the spec is the only human-readable artifact
- [[ThoughtWorks Future of Software Engineering Retreat]] — "where does the rigor go?" — Congdon answers "formal verification," ThoughtWorks says five destinations
- [[The Future of Software Engineering is SRE]] — operational signals as the backstop when review fails
- [[Correct by Construction]] — structural correctness philosophy without the formal verification toolchain
- [[Specsmaxxing]] — YAML specs with stable IDs; what specification looks like today for practitioners
- [[Radical Accountability]] — taste is all that's left when AI handles implementation
- [[Harness Engineering]] — feedforward/feedback framework; formal verification as the ultimate computational feedback
- [[Slowing the Fuck Down]] — deliberate friction; formal methods as the most extreme form
- [[Compound Engineering]] — add a system, not manual review; formal verification as the system

---

*Sources: [[raw/the-coming-need-for-formal-specification]]*
*Last updated: 2026-05-14*
