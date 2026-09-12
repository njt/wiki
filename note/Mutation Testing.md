# Mutation Testing

Stryker's documentation intro is the clearest short explanation of mutation testing I've read: plant small deliberate bugs in your source, run the suite, and count how many the tests catch. A mutant is *killed* when a test fails against the mutated code, *survived* when the suite still passes — and every survivor is a behaviour the tests never actually pinned down. The page's real argument is a contrast with coverage, delivered via a sandwich metaphor: coverage tells you the bread is 80% covered in paste; mutation testing tells you whether it's chocolate paste or something worse.

---

## Key Quotes

> "Mutation testing introduces changes to your code, then runs your unit tests against the changed code. It is expected that your unit tests will now fail. If they don't fail, it might indicate your tests do not sufficiently cover the code."

The definition in its most honest form. Note the hedge — "might indicate" — which the rest of the page quietly drops. A surviving mutant is *evidence* of a gap, not proof; mutation testing can only ever show you where the net might not bite, not that it will hold.

> "Code coverage would tell you the bread is 80% covered with paste. Mutation testing, on the other hand, would tell you it is actually *chocolate* paste and not... well... something else."

The memorable line, and a rare case where the joke carries the argument. Coverage is a statement about *execution*; mutation testing is a statement about *assertion*. This is the same gap antirez compresses to "covering all the lines does not mean covering all the possible states" in [[A New Era for Software Testing]], and the same false-confidence problem [[Test Validation and the Trustworthiness of Tests]] approaches from the "who validates the tests?" direction.

> "Stryker will only mutate *your source code*, making sure there are no false positives."

The one concrete design claim on the page, and it's doing two jobs at once. Mutating only source (never test code) prevents the trivial false positive where a mutant breaks the harness itself. But "no false positives" is marketing-grade overstatement — mutation testing has its own ghost problems (equivalent mutants, survive-by-accident mutants) that this sentence doesn't even acknowledge.

> "The second mutation in this example is marked as a survivor. This means there is probably a test missing that explicitly tests for age lower than 18."

The worked example paying off: `user.age >= 18` mutated to `return true` survives, which reads the missing test off the output directly. The clear-text reporter turns an abstract metric into a concrete "here is the line, here is the mutation, here is the mutator" — which is the actual product: mutation testing is only useful if the surviving mutant tells you what test to write.

## Key Themes

- **#concept Mutation testing** — the practice of seeding deliberate faults and counting how many the tests kill. The page's framing (killed = wanted, survived = gap) is the standard taxonomy, and it's why mutation score is a measure of suite *effectiveness*, not execution.
- **#concept Coverage is not confidence** — the page's load-bearing contrast. Coverage tells you code ran; mutation testing tells you a test asserted something about it. This is the same claim [[The Ten Properties of Software Quality]] makes from the design side when it lands on mutation testing as "the honest audit of a safety net."
- **#tool Stryker** — the vehicle. One design mentality across JavaScript/TypeScript, C#, and Scala; the pitch is "easy to use and fast to run," with the source-only mutation rule as its only architectural differentiator.
- **#pattern Mutation score as effectiveness metric** — the ratio of killed to total mutants as a number you can put next to the coverage percentage, with the survivor list as a to-do queue of missing tests.

## Critical Analysis

The page is excellent onboarding and thin documentation, and the two should be read as the same object. It teaches the concept with more clarity than most academic introductions — the killed/survived binary, the two-mutator example, the reporter output that turns a score into an instruction — but it's a pitch, not a reference. "Easy to use and fast to run" is the claim every user has to verify themselves, because mutation testing's historical blockers (performance, noise, equivalent mutants) are exactly what the page doesn't discuss.

The strongest idea here is the reframe of the survivor as *signal*: a surviving mutant isn't a failure of the tool, it's a missing test wearing a costume. That makes mutation testing a generator of test work rather than a gate, which is why it pairs naturally with the suite-audit instinct in [[Composable Tests]] — Beck prunes assertions by set-containment reasoning, mutation testing finds the assertions that were never there to prune.

The weakest part is the "no false positives" claim. Mutating only source code does prevent one class of false positive, but it's a sleight of hand that hides the real question: whether a *surviving* mutant always means "test missing" or sometimes means "this mutation is behaviourally identical, and no test could catch it." Stryker's mutators are chosen to minimize that, but the docs treat the hard problem as already solved.

---

*Sources: [[raw/docs]], [[summary/docs]]*
*Last updated: 2026-09-13*
