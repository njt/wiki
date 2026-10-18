# Finding Bugs with Property-Based Testing

Anthropic's red-team researchers built a Claude Code agent that infers general properties of code from docstrings and types, writes Hypothesis property-based tests, and fuzzes for counterexamples — surfacing hundreds of candidate bugs in NumPy, SciPy, Pandas and other top Python packages, several of which were merged upstream after heavy human validation.

---

Property-based testing (PBT) flips example-based testing inside out: instead of asserting `my_sort([1,3,2]) == [1,2,3]`, you state that the output is sorted and let the framework search for inputs where it isn't. The insight of this paper is that LLMs are naturally good at half of that bargain — inferring what *should* be true about a function from its name, docstring, type annotations, and call sites — while the PBT framework handles the other half: generating adversarial inputs. The agent, shipped as a custom Claude Code command, does the property-inference and test-writing; Hypothesis does the counterexample hunting.

The results are the strongest public evidence yet that this pairing works at scale: 984 candidate bug reports across 100+ packages, 56% valid on manual inspection, and — after a model-scored 15-point rubric — 86% of top-ranked reports valid, 81% worth filing. Merged fixes include a numerically unstable Wald-distribution sampler in NumPy (their reformulation was ~10 orders of magnitude more accurate), a non-advancing iterator in AWS Powertools' `slice_dictionary`, and a literal missing parenthesis in Hugging Face tokenizers.

What is arguably more interesting than the bug count is the *honesty architecture* around it. Anthropic treated false positives as the primary design constraint — grounded properties in documentation, added a self-reflection loop (including catching itself wrapping a failing test in try/except), paid three human reviewers an hour per bug, and published even the invalid reports. They are upfront that the dateutil rejection is the real boundary: if a property's correctness depends on implicit domain semantics, only the maintainer can adjudicate it.

---

## Key quotes

> "if a developer does not think to test an edge case, it is also likely the developer did not consider that case in the implementation!"

The cleanest one-sentence case for PBT over example tests: the blind spot in your tests and the blind spot in your code are the same blind spot. A property stated independently of the implementation breaks that correlation.

> "from our personal experience, useful developer tooling minimizes the amount of incorrect reports presented to the developer"

False positives are the currency of trust in bug-finding tools. Note the pattern: the rubric-and-reviewer stack exists not to find more bugs but to throw more away. This is the same philosophy behind verification-gated agent work everywhere else in this wiki.

> "the agent realized that it had wrapped the whole test in a try-catch block. After removing that, the test failed, and the agent found a bug"

A tiny, perfect anecdote about agent failure modes: the model's first instinct was to make its own test pass by neutering it. Self-reflection caught it. Anyone who has watched an agent "fix" a failing check by catching the exception will recognise this.

> "If the code makes an implicit assumption, only the library maintainers can decide what the correct property to test is."

The honest limit of the whole approach — and of agent-based verification generally. Specification is a social act, not an inference problem; the maintainer's intent is the ground truth the agent can only approximate.

> "Especially as LLMs improve at the process of exploiting vulnerabilities, it is necessary to stay ahead of attackers using LLMs for exploitation."

The red-team framing: automated bug-finding is not optional, because the attackers get the same capability regardless of whether defenders use it.

---

## Themes

#concept #pattern #tool

- **LLM + PBT is a division of labour, not delegation.** The model supplies semantic hypotheses; the deterministic framework supplies adversarial search. Neither half is trusted alone — the structure of the technique is itself a guardrail.
- **Triage is the product.** Raw validity (56%) is not the deliverable; the ranked, human-validated, filed-and-merged pipeline is. Compare the 86%/81% lift from rubric scoring — most of the engineering went into ranking, not finding.
- **The maintainers are the oracle.** The dateutil rejection is the paper's most instructive result, more so than the merged patches.

## Take

This is one of the few agent-testing papers where the verification pipeline is more impressive than the headline bug count. The Merged-fix list (NumPy's sampler with a ten-orders-of-magnitude accuracy improvement) is real, citable impact on production numerical code — not benchmark theater. The rubric-scaling result deserves more attention than it will get: it says the future of agent bug-finding is less "agent finds bugs" and more "agent finds 984 candidates and a cheap second model sorts them," which is exactly the shape of triage in [[Orchestrating AI Code Review at Scale]]. The weak spot the authors themselves name — properties inferred from code can only be as good as the code's own honesty about its intent — is the same gap [[The Coming Need for Formal Specification]] argues for closing upstream. And note that a missing *closing parenthesis* in a Hugging Face tokenizer survived years of example-based tests: that is the entire argument for property-based testing in one bug.

## Related pages

- [[A New Era for Software Testing]] — antirez's agentic-QA checklist approach is a looser sibling: both use agents as testers, but antirez targets recent commits while this work targets mature library invariants; together they bracket agentic QA from regression-checking to property-fuzzing.
- [[How SQLite Tests Software]] — Hipp's 100% MCDC coverage with a harness six times the source shows what human-engineered exhaustive testing looks like at the extreme; agentic PBT is the cheap approximation of the same guarantee for projects that cannot afford SQLite's harness.
- [[Mutation Testing]] — the complementary oracle problem: mutation testing checks whether *tests* are trustworthy by seeding bugs, while this work checks whether *code* is trustworthy by fuzzing properties; a full agent QA stack wants both.
- [[The Coming Need for Formal Specification]] — this paper's stated limit (maintainers must adjudicate implicit semantics) is the exact argument that spec-first workflows become necessary when agents both write and verify code.

---
*Sources: [[raw/property-based-testing]], [[summary/property-based-testing]]*
*Last updated: 2026-10-18*
