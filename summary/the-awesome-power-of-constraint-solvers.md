---
url: https://btmc.substack.com/p/the-awesome-power-of-constraint-solvers
title: "The Awesome Power of Constraint Solvers"
author: Bruno Turgeon-MacDonald (btmc)
date_fetched: 2026-10-11
date_published: unknown
topics:
  - software-engineering-craft
  - ai-research-and-models
---

A hands-on tutorial on constraint solving by an author whose entire career has been SMT, SAT, and constraint programming. After listing the real problems solvers have cracked for him (verifying distributed protocols, generating aircraft power systems, tuning boiler aerodynamics), he walks through a complete worked example: synthesizing an optimal 1-bit full-adder circuit out of NAND gates with the CPMpy Python API backed by Google's CP-SAT solver.

The modeling lesson is the heart of the piece: variables, domains, constraints, and the crucial distinction between finding *a* solution that works for one input versus one that works for all inputs — a ∀ quantifier that he handles by unrolling all 8 input combinations into parallel variable arrays. Symmetry breaking (only forward connections between gates) shrinks the search space, and the objective is split into two single-objective steps: fix the gate count, minimize depth.

The punchline is what makes solvers different from search-as-usual: in 99 milliseconds the solver proves UNSATISFIABLE for 8 gates — not "failed to find," but exhaustively proved no solution exists. It then confirms depth 6 is optimal (OPTIMAL, 0.47s) among all 9-gate circuits, of which there turn out to be 24,576 solutions that are all isomorphic to the same circuit. Despite worst-case exponential complexity, state-of-the-art solvers eat most real problems; they only strain past 2-bit adders.

He closes by pointing at the algorithms behind the magic (CDCL for SAT, AC-3 for CP, Simplex for LP) and noting the same SMT techniques power the `Grind` tactic in Lean, which is doing the heavy lifting in the LLM-generated math proofs of the moment.
