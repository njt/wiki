# Agentic Verification

Addy Osmani's essay on the shift from reviewing code to building verification loops: agents make checking cheap, so quality comes from equipping the agent with fast, deterministic oracles — E2E tests as ground truth, property-based tests as constraints that can't break, differential tests that make the old system the oracle — rather than from humans reading diffs.

---

Osmani's premise is blunt: "Today's models are good enough that the major bottleneck is not code-writing but the ability to verify that the code they've written is correct." Writing checks used to be expensive; agents make them cheap to produce and cheap to improve. So the craft moves: "our work is less 'review every line' and more 'build loops the agent can run itself'." When the agent errs, write the correction down — shared config, a hook, a test — so taste compounds and the mistake can't happen twice. The human still owns what ships, but now reviews *evidence*, not just diffs.

---

## Key quotes

> I would always prefer a much better feedback loop to a slightly better model.

The essay's thesis in one line, and its most quotable claim: model quality is substitutable, loop quality is not. An agent that can't verify its output is "essentially guessing," and the human-dominated guess-and-report loop "is not scalable to more than one human."

> Tests are... how your agent senses the world and knows when it's done.

A genuine redefinition. Tests stop being a human QA artifact and become the agent's sensory apparatus. This reframes test investment: not every test is equally valuable, and value is now measured by how well the oracle steers the agent.

> An E2E test, on the other hand, is very hard to trick. Either the whole thing works, or it doesn't.

The argument for E2E as the single oracle worth having: a mock-based unit test can pass while agent and test share the same misunderstanding. E2E also "defines done" — "make this E2E test pass" is a target the agent can chase autonomously, where "implement this feature" is not.

> The property forces the agent to deal with any input generated, not just ones it chooses.

Property-based testing (fast-check, Hypothesis, proptest) directly attacks the agent's failure mode of inventing examples that succeed by its own internal logic. Round-trip encoding, invariants, idempotency, and equivalence are named as properties nearly every project has. Osmani's migration advice is equally sharp: the classic rewrite failure is passing the new suite while diverging from the old system in undocumented ways — users rely on "what actually works, not what the specification says should work," so treat the old system as the oracle and diff against it.

> If it's possible to retry a failing test and have it work once in a while, it'll pick retrying as its preferred strategy.

The loop-as-training-exercise insight: flaky or slow tests teach the agent to rerun rather than fix, and both failure modes degrade output "without any visible sign of failure." A flaky test in the main suite "is worse than no test at all, because it breaks trust in every other result."

## Key themes

#concept #pattern #tool

- **Verification as the bottleneck** — generation is solved; oracle design is the craft.
- **The oracle spectrum** — E2E for ground truth, properties for "what must never happen," differential testing for migrations, with the old system as oracle.
- **Loop hygiene as incentive design** — the agent learns from the loop's incentives; speed and determinism are quality controls on the agent's behaviour, not just developer convenience.

## Analysis

The essay is the sharpest short statement of the verification-turn in agentic coding practice, and its strongest move is economic rather than technical: the reason to invest in oracles is that agents made them cheap, flipping the old cost calculus where tests were the expensive part of "Good Code." Its honesty about trade-offs — E2E is slower and flakier, so prefer a small set of high-value tests — keeps it out of sermon territory.

The embedded reply thread does real work the main text can't. Lauren's point (give the agent hands and eyes: a maintained CLI to launch, drive and observe the real system, plus a feature map) quietly extends the thesis beyond tests into tooling. Boris Cherny's note on Lean/TLA+ for state machines points at formal verification as the extreme end of the same spectrum. And the sharpest objection comes from a commenter, not Osmani: if the same agent writes the feature and the E2E test that defines done, "don't they share the same misunderstanding you describe with mocks?" Osmani never answers it — property and differential oracles partly do, but the "who writes the check" problem remains the open wound in verification-first practice.

There's also a quiet tension with Osmani's own [[Own the Outer Loop]] framing: here the human reviews "evidence, not just diffs," which is attractive until you ask whether evidence produced inside the same loop the agent optimises is really independent. Evidence needs provenance from outside the agent's authority — which is exactly the constraint other loop-engineering work makes explicit.

## Related pages

- [[Loop Engineering Optimizing Code Generation and Verification]] — strengthens the same bottleneck thesis with more machinery: conformance contracts, an independent evaluator outside the builder's authority, and per-check detection/correction/cost accounting — the formal answer to the "who writes the check" problem this essay leaves open.
- [[A New Era for Software Testing]] — antirez's agentic-QA take complements this: both treat tests as the agent's quality mechanism, but Osmani pushes further into property and differential oracles as specifications, not just checks.
- [[Addy Osmani's Workflow]] — this essay is the verification-specialised evolution of Osmani's earlier iteration loop (plan + implement + test + commit); the shift from "always review" to "review evidence, not diffs" marks how far his position has moved.
- [[Five Mistakes People Make with Agentic]] — intellectronica's "agents without taste or a way to check their work" failure mode is precisely what this essay's compounding-taste and oracle-building prescriptions fix.

---
*Sources: [[raw/agentic-verification]], [[summary/agentic-verification]]*
*Last updated: 2026-10-18*
