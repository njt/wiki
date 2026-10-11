# The Awesome Power of Constraint Solvers

A tutorial that models the synthesis of an optimal NAND-gate full-adder circuit as a constraint satisfaction and optimization problem, demonstrating with CPMpy and Google's CP-SAT what classical AI solvers can prove in milliseconds — and why most programmers have never touched these tools.

---

Most programmers have never used a SAT solver, an SMT solver, or a constraint programming engine, and don't know that the same machinery verifies distributed protocols, generates mechatronic architectures, and — via Lean's `Grind` tactic — powers the automated parts of the LLM math proofs flooding the feed. The author spends his career in these tools and builds the tutorial around one question: can you write a *program* that finds the minimal, shallowest NAND-only full-adder?

The worked example is a masterclass in modeling, not solving. Decision variables per gate (two inputs, output, depth), boolean connection variables between every gate pair, exactly-one constraints on input ports, and the pedagogically deliberate mistake first: constraining the adder on a *single* input/output pair lets the solver return nonsense that happens to satisfy one row of the truth table. The fix is the universal quantifier, unrolled — 8 parallel sets of value variables, one per possible input, routed through the same connection structure. This is the ∃p ∀i form of program synthesis, and the article briefly names the alternative (CEGIS, two solvers cooperating on the two quantifiers) before choosing brute-force unrolling for a 3-bit input space.

The payoff lines are the ones that separate solvers from heuristics:

> If we ask for all solutions using `solver.solveAll` instead of `solver.solve`, we get that there are 24576 solutions, but many of those solutions are *symmetric* (meaning equivalent), they just have elements swapped around that don't make any difference to the result.

> In just 99 milliseconds on my dying M1 MacBook Pro I'm told in no uncertain terms that nope, you cannot make a Full-Adder with only 8 NAND-gates … this doesn't mean the solver failed to find a solution, it explored the entire design space in 99 milliseconds. *There is no solution*.

That distinction — UNSATISFIABLE as a *proof* over the whole space, not a timeout — is the core value proposition, and it's why the same machinery sits under F* and Lean proof automation. The trick to multi-objective optimization is also the kind of pragmatic modeling wisdom you only get from practitioners: fix the gate count, minimize depth, increment the count when UNSAT.

**Opinionated take.** This is the best kind of tutorial: it teaches the *modeling* skill, which is the actual bottleneck, while treating the solver as a black box — the author explicitly declines to explain CDCL, pointing instead at "Crafting Interpreters"-style future work. Two critiques. First, the article undersells the cliff: 1-bit works in milliseconds, 2-bit is "fair game," and beyond that the exponential wall arrives — the honest framing is that solvers are a scalpel for problems you can *formalize and keep small*, which is a modeling discipline most teams never develop. Second, there's an unstated argument here for the AI era that deserves to be made explicit: while LLM agents degrade as constraints accumulate (see [[Constraint Decay]]), CP-SAT *thrives* on constraints — constraints are its input, not its failure mode. The likely synthesis is not solver-vs-LLM but LLM-proposes-model, solver-verifies-and-searches: the agent writes the CSP, the solver guarantees the answer. That division of labor — stochastic front end, deterministic back end — is exactly the pattern the rest of this wiki keeps rediscovering under different names.

**Key themes:** #concept #tool #pattern

**Related pages:**

- [[The Coming Need for Formal Specification]] — strengthens its argument that verification moves upstream in the AI era; this article shows the deterministic engine that such specifications would ultimately feed, and the `∀` quantifier is precisely what a formal spec expresses.
- [[Constraint Decay]] — the mirror image: LLM agents collapse as constraints pile up, while CP-SAT converts constraints into its entire search strategy. This source sharpens that paper's implicit suggestion that deterministic generation beats agentic rule-following.
- [[We Have Proof Automation Now]] — Langley's SMT-backed F* proof automation is the theorem-proving cousin of the machinery here; both make the same point that predictable, solver-based brittleness beats stochastic, prompt-based brittleness.
- [[Four Provers, One Theorem (blueberrywren)]] — the hands-on experience of driving proof tools; this article explains what the `Grind`/sledgehammer tactics those provers invoke actually are under the hood (SMT search).

---
*Sources: [[raw/the-awesome-power-of-constraint-solvers]], [[summary/the-awesome-power-of-constraint-solvers]]*
*Last updated: 2026-10-11*
