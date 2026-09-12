# Composable Tests

Kent Beck separates two properties from his Test Desiderata that everyone conflates: **isolation** (one test's result doesn't depend on another's) and **composition** (a suite gives confidence no single test gives alone). The payoff is a technique he knows people hate — deliberately *removing* assertions from a test when an earlier test already covers them — and an N×M argument that 4 computation variants × 5 reporting variants needs 10 tests, not 20, if the dimensions are demonstrably orthogonal. The essay's weakest move is dismissing the objection to it as "fear, not principle"; its own comment section supplies the technical objection Beck skipped.

---

## Key Quotes

> Isolation—the result of running one test should be completely independent of the results of other tests.
> Composition—??? tests should run together ??? Isn't that the same thing as isolation?

Beck quoting his own Desiderata list with the question marks intact. Worth noting how rare this is: an author publishing that one of his own twelve named properties was under-specified for years, and that the blocker was never the theory. "I finally got an example—examples are always the hardest part."

> If a test runs by first setting up its own test fixture, creating from scratch all the data it will be using as input, then that test is guaranteed to be *isolated*. ... (This is the same property as referential transparency in functional programming.)

The cleanest one-line definition of test isolation available, and the FP framing earns its keep — it explains *why* fresh-instance-per-test is the xUnit default rather than an arbitrary convention. Beck's aside that NUnit reuses test instances, "opening the door to breaking isolation," is a real and checkable framework difference.

> Notice that test2 can't pass if test1 fails. All non-compliant programs caught by test1 will also be caught by test2.

This is the whole essay in two sentences. Once you see that `test2`'s first assertion is a strict subset of `test1`, the redundancy is not a matter of taste — it's a set-containment fact. The three responses (keep both, delete `test1`, trim `test2`) all preserve coverage exactly.

> From a purely aesthetic standpoint (& don't discount aesthetics), leaving both tests as is offends my sensibilities. They are redundant! Something *must* be wrong.

Aesthetics as a diagnostic instrument rather than decoration — the same stance Sandi Metz takes in [[99 Bottles of OOP]], where "programming aesthetic" is the named antidote to evaluative anesthesia. Beck's parenthetical is doing load-bearing work: he is claiming that the feeling of wrongness is *evidence*, and the rest of the section is him cashing the feeling out into the specific property it was detecting.

> Deleting test1 loses us another property from the Test Desiderata—tests should be *specific*. That's the property of tests where, when one fails, you know exactly where the problem is.

The step that makes the argument non-obvious. The naive dedup move — delete the subsumed test — is the wrong one, because coverage is not the only thing tests carry. Diagnostic resolution is a separate asset, and deleting `test1` spends it.

> When I've explained what I mean by composable tests, I often receive shocked reactions from experienced testing-developers. "I would *never* reduce the assertions in a test." This seems to me to be a reaction based in fear, not in principle.

The most quotable line and the least defensible. See the critical analysis below — the reaction has a principle behind it, and one of Beck's own commenters states it precisely.

## Key Themes

- #concept **Isolation vs. composition** — isolation is a property of a test relative to its neighbours; composition is a property of the suite as a whole. Optimising every test individually does not optimise the suite.
- #pattern **Pruning subsumed assertions** — when `testB` strictly subsumes `testA`, trim `testB` rather than deleting `testA`. Coverage held constant, specificity preserved or improved.
- #pattern **N×M decomposition** — test M variants on one axis, N on the other, plus *one* wiring test. 4+5+1 instead of 4×5.
- #concept **Specificity as a separate asset from coverage** — "when one fails, you know exactly where the problem is." Deduplicating on coverage alone destroys it.
- #concept **Aesthetics as diagnostic signal** — the feeling that redundancy is wrong is treated as evidence to be cashed out, not indulged.
- #person Kent Beck — see also [[The Cost YAGNI Was Never About]] and [[Martin Fowler and Kent Beck on Reinventing Software]].

## Critical Analysis

**The N×M argument is the strongest thing here, and also the most conditional.** Beck says it plainly — composition "requires some thought, some inference, some design (to make the orthogonal dimensions demonstrably orthogonal)" — but he doesn't give you a way to *check* that the orthogonality claim holds. That's the whole risk. If the reporting code branches on interest type anywhere, the 4+5+1 suite passes green while the specific (compute-variant-3, report-variant-2) interaction is broken, and the single wiring test almost certainly exercised variant 1 of each. The 20-test brute force catches that; the 10-test composition doesn't. This is the combinatorial-testing tradeoff that all-pairs literature has been arguing about since the 1990s, and "demonstrably orthogonal" is doing enormous unexamined work in that sentence. The honest version of the claim is: *composition converts a testing cost into a design constraint.* You now have to keep the dimensions independent, forever, and nothing in the test suite will tell you when someone stops.

**The best objection in the piece is in the comments, not the body.** A reader points out that if `doSomething()` is broken, `nowSomethingElse()` runs in an unknown state — so a failure in the trimmed `test2` may be a downstream artifact rather than a real bug in the code under test. Beck's reply to the general form of this objection was "fear, not principle," but this *is* a principle, and the commenter even supplies the fix: `assumeTrue(object.isReadyToDoSomethingElse())`. That's the right patch, and it's better than either of Beck's three options. A failed *assumption* is not a failed assertion — frameworks like JUnit report it as a distinct status — so you keep the diagnostic signal ("test2 didn't fail, its precondition didn't hold") without restoring the redundant assertion Beck wanted gone. Beck's technique survives the objection, but only because a reader repaired it.

**"Fear, not principle" is the paragraph to push back on.** Assert-everything-always is a robust heuristic precisely because it doesn't require design judgment. Beck's technique is strictly better *in the hands of someone who can reliably tell orthogonal from entangled* — and strictly worse in the hands of someone who can't, because it removes the redundancy that was silently covering for their misjudgment. That makes composable tests a craft technique presented as a rule. Telling practitioners their hesitation is emotional, rather than engaging with the failure mode, is the move an author makes when the technique's safety envelope is hard to specify. Compare [[Ways of Checking]], which catalogues verification failure modes without ever asserting that the people worried about them are just scared.

**Note what Beck is *not* doing: he isn't abstracting.** This matters because it looks superficially like the move Sandi Metz warns against in [[The Wrong Abstraction]] — seeing duplication and reaching for a fix. It isn't. Beck's remedy is *deletion*, not extraction: no shared helper, no test base class, no parameterised fixture. Nothing new is created that later callers will be tempted to bend. That's why the two positions don't actually collide. Metz's rule is that duplication is cheaper than the *wrong abstraction*; Beck's target is duplication that can be removed without building any abstraction at all. The dangerous misreading of this essay would be someone extracting `assertDoSomethingWorked()` into a shared helper and calling it composition — that's the DRY reflex Metz spent a career warning about, and it re-couples the tests Beck just decoupled.

**The AI-era relevance Beck doesn't mention.** Copy-paste-extend is *exactly* how coding agents generate test suites — it's the dominant pattern in generated tests, and nobody is pruning them, because as [[Test Validation and the Trustworthiness of Tests]] argues, AI-generated tests largely escape the review that AI-generated production code now gets. Beck reports seeing this pattern repeated "6 or 7 times" by humans; an agent will do it forty times without fatigue. Composition is the discipline that would fix it, and it is precisely the discipline that requires the design judgment agents are worst at. There's also a straightforward economic angle Beck leaves on the table: a composed suite is smaller and faster, which in an agentic loop means cheaper CI and fewer tokens spent reading test output — the same argument [[The Economic Benefit of Refactoring]] makes empirically for production code.

**On the source itself:** the fetched text carries an unmarked CodeRabbit advertisement — four sections of vendor copy about AI code review — wedged between Beck's conclusion and the reader comments. It is not Beck's argument and is labelled as sponsorship only at its end. Anyone quoting from the raw file should be careful where Beck stops.

---

## See Also

- [[Frozen Test Fixtures]] — Radan Skorić's "a test should test only that which it is meant to test, no more and no less" is Beck's pruning principle arrived at from the fixture side; both are about assertions that quietly claim more than they should
- [[Test Validation and the Trustworthiness of Tests]] — Typemock asks who validates the tests; Beck supplies one concrete criterion (strict subsumption) for a test that shouldn't exist in its current form
- [[The Cost YAGNI Was Never About]] — the same author reasoning the same way: a practice everyone thinks is about saving effort is really about a property (optionality there, specificity here) that effort-based framings can't see
- [[The Wrong Abstraction]] — the near-miss: Metz on why duplication beats bad abstraction, and why Beck's delete-don't-extract remedy sits outside her warning
- [[99 Bottles of OOP]] — Metz on programming aesthetic as a real design instrument, which is Beck's "don't discount aesthetics" at book length
- [[Ways of Checking]] — a catalogue of verification failure modes; the counterweight to trusting a smaller suite
- [[A New Era for Software Testing]] — antirez on agentic QA, the generation-side counterpart to Beck's curation-side discipline
- [[Litmus (.NET Testing Priority Tool)]] — tooling that ranks *where* to test; Beck's argument is about how many tests that coverage actually requires
- [[Software Engineering Craft]] — testing craft as durable practice

---
*Sources: [[raw/composable-tests]], [[summary/composable-tests]]*
*Last updated: 2026-08-21*
