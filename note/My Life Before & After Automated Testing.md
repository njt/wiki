# My Life Before & After Automated Testing

Dan Lew's 2026 post is a conversion narrative: a developer who once thought automated testing was a waste of time explains, via one feature built twice, why he now considers it the practice that most improved his coding. It matters to this wiki because it is the clearest statement of the *pre-agent* case for tests — and the agent-era debate about TDD and verification is best understood against exactly this baseline.

---

## The argument in one paragraph

Automated testing improves software development primarily through two mechanisms: tight feedback loops during development (the near-zero cost of running tests means you verify constantly rather than at major checkpoints, and constant nudges make course correction cheap) and cheap, consistent regression testing after release (which amortises its upfront cost, protects future maintainers who never wrote the feature, and makes refactoring safe). Lew claims these two benefits — not bug prevention per se — are what "dramatically changed" how he develops code. This is falsifiable: if the feedback-loop and regression benefits could be obtained another way (say, by fast manual harnesses or agent-driven QA), the practice's centrality would collapse, which is precisely the bet several agent-era sources are making.

## Key quotes

> It’s hard for me to overstate how much my coding has improved since embracing automated testing. And yet, I used to think of it as a waste of time!

The framing device for the whole post, and an honest one: Lew writes for his former self, which means the post is aimed at sceptics rather than preaching to the converted. Most testing advocacy skips this step and loses its audience.

> Tight feedback loops can be all the difference between strong and weak code because it’s much easier to course correct if you’re constantly getting nudges in the right direction.

This is the load-bearing claim, and it is an *economic* argument, not a quality argument: the value comes from lowering the cost of verification until it happens continuously. That framing is exactly what transfers to agents — and exactly what the "tests as harness" school of agentic coding borrows.

> A test that randomly passes sometimes is worse than no test at all.

The sharpest line in the caveats section. A flaky test doesn't just fail to help; it actively trains the team to ignore red, corroding the very signal the whole practice depends on. Lew's prescription is ruthless — fix or delete — with no middle ground.

> You lose a *lot* of the advantages of automated testing this way (all the “development” parts of it), plus the code probably already works.

An underrated point that cuts against testing culture's puritan streak: retroactive coverage campaigns capture the cost of testing without its main benefit, because the feedback-loop value only exists while the code is being written. Tests are a development tool first and a safety net second.

> That’s why __Martin Fowler’s “Refactoring”__ essentially boils down to “write tests, then refactor.”

Lew compresses the refactoring canon into one sentence, and it's a fair compression: the safety of refactoring is entirely borrowed from the test suite. Without it, "refactoring" is just rewriting with extra confidence.

## Critical analysis

The non-obvious move here is the parallel-story structure. Rather than listing benefits abstractly, Lew runs the *same feature* — job cancellation in a work queue — through both workflows, and the divergence points are telling: the concurrency bug appears in both stories, but in the before-story it surfaces via QA and slapdash testing, while in the after-story it surfaces as "a few more tests" written before refactoring. The lesson isn't that testing prevents mistakes; it's that testing changes *when and by whom* mistakes are caught, which is a far more defensible claim.

The weaknesses are the ones Lew mostly waves at. The before-story is something of a strawman — no competent manual-testing regime involves "even more slapdash" testing during a concurrency refactor — so the contrast is rigged in the practice's favour. The claim that adapting code for testing improves architecture is asserted with an explicit "you'll just have to trust me," which is the weakest link in an otherwise evidence-flavoured post. And the cost side is underexplored: the "high upfront cost... amortized" argument assumes the feature lives long enough to amortise over, which is exactly the assumption YAGNI-minded developers contest.

What the post leaves out — and what makes it newly relevant — is the agent. Lew wrote a pre-agent argument in 2026, and its two benefits map directly onto the agent-era debate: tight feedback loops are precisely what makes agentic coding work (the agent runs the tests, not you), while the caveats — tests only as good as their writing, flaky tests worse than none — become the failure modes of agent-driven verification. The post never mentions LLMs, which is either a blind spot or a deliberate grounding: the case for tests predates the agent and will outlast whatever the agent era does to TDD ritual.

## Related

- [[Agentic Manual Testing]] — Willison's claim that automated tests are necessary but insufficient (they pass while the server crashes on startup) nuances Lew's position: Lew's "you still need manual testing" caveat is the same argument, but Willison operationalises it as an agent running `curl` and browser automation rather than a human clicking buttons.
- [[TDD Inside the Agent Loop — Theater or Actual Value?]] — complicates Lew's write-tests-first workflow: the Thoughtworks experiment found that moving Lew's exact loop (tests first, implement against them) inside an agent's loop produced no quality gain at 3–8.5x token cost, suggesting the benefits Lew credits to the practice may be human-psychology mechanisms that evaporate without a human in the loop.
- [[More Code Coverage, Less Confidence]] — strengthens Lew's "don't religiously test everything" caveat with the inverse evidence: more coverage metrics can actively decrease confidence in a suite, which is the systemic version of Lew's warning about combinatorial explosions and bending over backwards to test.
- [[A New Era for Software Testing]] — extends Lew's regression-testing benefit to its agent-era conclusion: antirez argues AI agents make QA cheap enough to compensate for AI-written code quality, which is Lew's amortisation argument applied to a new writer of the tests.

---
*Sources: [[raw/my-life-before-after-automated-testing]], [[summary/my-life-before-after-automated-testing]]*
