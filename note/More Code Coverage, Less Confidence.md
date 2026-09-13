# More Code Coverage, Less Confidence

Typemock's argument that code coverage and test confidence are different quantities that only sometimes move together. Coverage measures execution; confidence requires verification, isolation, and distinct behaviors — and the gap between the two is exactly where an AI told to "increase coverage to 90%" will operate. A vendor essay, but the diagnosis section is a genuinely crisp statement of why a green coverage dashboard can mean less than it appears to.

---

## Key Quotes

> "Code coverage can tell us: **This code executed.** It cannot tell us: **This behavior was correctly verified.**"

The article's load-bearing distinction, delivered via a two-line C# example: a test that runs the premium-discount path but asserts `result >= 0` covers the branch while accepting 0.01, 0.10, or 0.19 as a 20% discount. This is the entire gap between the two quantities in one snippet. Everything else in the article is this example scaled up to a suite.

> "The problem starts when we turn code coverage from a **signal** into a **score**."

Goodhart's law in six words, applied to the oldest testing metric. As a signal — "this business logic has zero coverage," "this PR collapsed coverage" — coverage is honest and useful. As a score, it becomes the thing teams optimize, and optimization detaches the number from the thing it was proxying. The lines-of-code KPI analogy (four identical `if` statements, "KPI looking fantastic...") makes the absurdity visible; coverage gets the same caution only because its absurdity is harder to see.

> "Five tests. Perhaps they use slightly different values or assertions, but fundamentally they exercise the same behavior."

The duplicate-test catalog — `CalculateDiscount_PremiumCustomer`, `CalculateDiscount_PremiumCustomer_ReturnsDiscount`, `CalculateDiscount_WithPremiumCustomer`, `PremiumCustomerGetsDiscount`, `CalculateDiscount_WhenPremiumIsTrue` — is the mechanism by which dashboards improve while safety nets don't. The numbers look excellent (Tests ↑, Coverage ↑, Build ✓) but "the amount of unique protection may have barely changed." AI generation makes producing this catalog nearly free, which is the article's real occasion.

> "Maintenance cost: ??? Unique confidence added: ??? Developer attention: ???"

The three numbers that never appear on a coverage report, and the honest admission that they are "much harder to measure than a percentage." Every test is code someone owns; every external resource a supposedly isolated test touches is a future mysterious CI failure; every duplicate multiplies the cost of a legitimate production change. Coverage reports none of this — which is why Suite A (1,200 tests, 95%, slow, duplicative) and Suite B (750 tests, 85%, fast, isolated) can't be ranked by the percentage alone.

> "Tell an AI: Increase code coverage to 90%. ... Technically, it succeeded. But did we ask it the right question?"

The AI section is the article's most current contribution. Coverage is "a very measurable objective" — the agent can generate, run, inspect, and keep generating until the target is hit (82% → 86% → 91%, mission accomplished). A measurable objective is precisely what makes it a bad instruction: the agent optimizes the proxy, not the intent. The proposed replacement — "increase meaningful protection of important application behavior" — is deliberately unmeasurable, because the measurable version is what produced the problem.

> "Because sometimes you can have **more coverage and less confidence**."

The title thesis, stated as the closing line. The prescribed discipline is modest and practical: celebrate the green arrow, then ask *why* it went up — newly covered behavior, a real edge case, a regression guard, or just another collection of tests that happen to execute more lines.

---

## Key Themes

- **#concept Coverage ≠ Confidence**: Execution and verification are different properties. Coverage answers "what code ran during the test"; confidence answers "would I trust this suite to catch a regression in behavior I care about." The article's contribution is refusing to treat the first as a proxy for the second, even while defending coverage as a useful signal.

- **#pattern Signal vs. Score**: A metric is useful while it is observed and corrupts when it is optimized. The 72% 😟 → 97% 🏆 emoji ladder is the corruption made visible. This is the software-engineering instance of Goodhart's law, and AI test generation is the force that makes the corruption automatic rather than merely likely.

- **#pattern The Duplicate-Test Multiplier**: Near-duplicate tests inflate every quantity metric while adding roughly zero unique protection, and they convert one legitimate production change into N test updates. AI amplifies this because the model doesn't know what the suite already pins down — the same dynamic the sibling Typemock article identifies.

- **#concept Complementary Signals**: "Test Result → Did the test pass? Coverage → What code executed? Test Review → What deserves attention?" No single metric can grade a suite, and the article argues that's how it should be — software engineering rarely fits one percentage.

- **#tool Typemock Test Review**: The commercial product this essay motivates: analysis of whether tests duplicate existing behavior and whether "isolated" tests touch external resources. Like its sibling article, the argument is a category pitch; the diagnosis is the durable part.

---

## Critical Analysis

**This is the diagnosis half of a pitch the wiki has already catalogued.** [[Test Validation and the Trustworthiness of Tests]] is the product thesis — "who is validating the tests?" — while this essay supplies its motivating metric argument: the reason test validation needs to exist is that the one metric everyone already watches (coverage) measures the wrong thing. Read together, the pair is coherent vendor content: this article explains why dashboards mislead, that one sells the tool that tells you what the dashboard concealed. The concrete `CalculateDiscount` example here is the best single piece of exposition either article produces — one test, one weak assertion, the whole fallacy visible in ten lines.

**The AI section is a textbook Goodhart story, and the article doesn't name its own law.** "Turn a signal into a score" *is* Goodhart's law; [[Goodhart's Law and AI Benchmarks]] documents the same dynamic in evaluation, where benchmarks optimized by models decouple from the capability they proxy. Coverage was arguably the first software metric to get the Goodhart treatment (mandatory coverage gates long predating LLMs), but AI makes the optimization loop closed and cheap: the agent can run the coverage tool itself and iterate until the number is hit. The article's proposed fix — prefer the unmeasurable objective ("meaningful protection of important behavior") — is directionally right but operationally hollow: an instruction no tool can score is an instruction no autonomous loop can pursue. The real answer sits between the two, and the article doesn't reach for it.

**The article never mentions the technique that already answers its question.** Mutation testing measures whether tests *verify* behavior rather than merely execute it: plant small mutants, count what the suite kills. A suite of `Assert.IsTrue(result >= 0)` tests scores terribly under mutation analysis no matter what its coverage says — the exact divergence this essay describes, made quantitative. This is the same gap the sibling note flagged ("test validation isn't a new idea"). [[Mutation Testing]] is the honest way to audit the safety net the article worries about, and the omission is understandable only as product positioning: mutation scoring is a commodity, runtime duplicate-detection is a differentiator.

**The TDD-in-the-loop experiment supplies the empirical shadow of this argument.** [[TDD Inside the Agent Loop — Theater or Actual Value?]] found that agents told to practice red-green-refactor produced no measurable quality gain — and no mutation-score gain — at 3–8.5x the token cost. That is this article's prediction confirmed from the other side: when the loop's objective is a proxy metric, the agent satisfies the proxy and the confidence never arrives. The two sources together suggest the practical rule: whatever you tell the agent to maximize, assume it will maximize exactly that and nothing else — so the objective had better be the thing you actually want.

**What's missing:** any data (how much of a typical suite is duplicate?), any acknowledgment of property-based testing or mutation tools, and any engagement with the obvious counterargument — that a *floor* on coverage, used as a tripwire rather than a target, captures most of coverage's value without the Goodhart corruption. The suite A/B thought experiment concludes "there's not enough information to answer," which is honest but also conveniently unfalsifiable; a real essay about test quality might have proposed the second metric that would settle it.

---

*Sources: [[raw/more-code-coverage-less-confidence]], [[summary/more-code-coverage-less-confidence]]*
*Last updated: 2026-09-13*
