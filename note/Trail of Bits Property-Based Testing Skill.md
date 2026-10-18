# Trail of Bits Property-Based Testing Skill

Trail of Bits' property-based-testing skill for coding agents — covering how to spot, write, review, and triage property tests across languages (Hypothesis for Python, Echidna/Medusa for EVM invariants) — paired with an unusually honest account of how the skill was measured: deliberate context-budget cuts to its own reference files, an eval harness kept out of the skill's context, and a detailed postmortem of a classifier bug that had corrupted its own published trigger-rate numbers.

---

The skill itself has four jobs: spotting PBT candidates (encode/decode pairs, validators, normalizers, pure functions with wide domains, smart-contract invariants), writing properties well (strategy design, `@example` pinning, `deadline=None`), reviewing existing property tests for tautologies and vacuous `assume()` calls, and triaging shrunk counterexamples into real bug / wrong property / ambiguous spec.

What makes the page genuinely interesting is the meta-layer. Three reference files were deleted with recorded rationale:

> `st.integers(min_value=1)` and `fc.string()` are not knowledge a current model lacks, and paying context for them crowds out the judgment it does lack.

And one was restored after an initially wrong cut:

> `refactoring.md` — **restored** in trimmed form. The cut was wrong. Both SKILL.md and the strength ordering send you to "this code is a poor PBT candidate", and without this file that is a dead end where an answer exists.

The stated principle — "a deletion with no rationale is indistinguishable from an oversight" — is a documentation discipline most skills never attempt: every removal is an argued decision, not silent accretion. Similarly, the eval harness (900 lines of bash, a requirements.txt, deliberately-broken fixtures) lives outside the skill directory, so guidance the model reads never ships alongside tests that are broken on purpose.

The measurement methodology is where the page gets almost confessional. Two methodological corrections:

> **A failed session is not a non-trigger.** A crash, a timeout or a rate-limit produces no Skill call, which is indistinguishable from a model that considered the skill and declined.

> **But an invocation outranks a failure.** A Skill call is positive and final: nothing later in the session can un-call it.

The corrupted-numbers postmortem is the best part. An earlier table recorded recall 5/7 because the classifier checked exit status *before* checking whether the skill had been invoked:

> In one salvaged sample, **9 of 12 sessions were being thrown away and all 12 had invoked the skill**.

And the bias was systematic, not random: longer, more exploratory queries are both likeliest to hit a turn cap and the ones whose positive evidence matters most — so the bug depressed recall on exactly the queries under study, and the earlier conclusion that the Echidna/Solidity path "triggered poorly" does not survive.

Finally, the effectiveness gate is brutal about value:

> Run the baseline before adding to this skill. Opus already writes competent Hypothesis suites unaided, so content that does not move a number against `NOPLUGIN=1` is costing context without buying anything.

---

**Key themes:** #concept #tool #pattern

**Analysis.** This is the most rigorous skill-engineering writeup in the Trail of Bits style: every parameter (timeout, turn cap, gate floor) traced to a measurement, every deletion argued, every number stamped with its model (`opus`) because "trigger rate is a property of a description *and* a model." Two things deserve pushback. First, the honesty has a cost — this README is 80% harness methodology and 20% testing guidance, and someone wanting to actually learn PBT gets almost nothing; the page is for skill maintainers, not skill users. Second, the gate floor of 13 (rather than the measured 14) is an unprincipled fudge dressed as rigor — they admit "one valid sweep is not enough to tighten a gate," which is true, but the same reasoning argues the current floor is also arbitrary. Still, the epistemics here — baseline-first, measured-over-taste, postmortems of your own published numbers — is exactly what most agent-skill publishing lacks.

Notably, one eval was *removed* as unmeasurable: the triage query (is a shrunk counterexample a real bug?) sat at 0/3 because the model answers directly and never reaches for guidance — and neither `should_trigger` label was true. The honest answer was "unmeasured."

**Relations.**
- This is the tooling counterpart to [[Finding Bugs with Property-Based Testing]]: Anthropic showed agents finding bugs with PBT at scale; Trail of Bits ships the skill and, crucially, shows the discipline of proving the skill actually fires and actually helps over a no-plugin baseline.
- It nuances [[A New Era for Software Testing]]: antirez's cheap agentic QA checklists are a world away from $36/45-session sweeps with self-tests in CI — this is what evaluation looks like when taken seriously.
- It strengthens [[Eval-Driven Development (Airbnb)]]: the same "measure, don't guess" ethos applied not to an LLM feature but to a skill's trigger description, including baseline ablation and the refusal to trust a single stochastic run.
- It complicates [[Claude Code Skills System]]-style takes: here the skill's own content is treated as a context budget to be cut, with deleted files kept as recorded rationale — a maturity step beyond "add more references."

---
*Sources: [[raw/property-based-testing-github]], [[summary/property-based-testing-github]]*
*Last updated: 2026-10-18*
