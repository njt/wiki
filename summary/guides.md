---
url: https://eng-atlas.dev/guides
title: "Engineering Atlas — The Ten Properties of Software Quality"
author: Unknown (site is unsigned)
date_fetched: 2026-08-21
date_published: unknown
topics:
  - software-engineering-craft
---

The Engineering Atlas guide opens with a reframe: stop asking whether a system
is *good* and start asking which property it is missing, because "good" is not
a property. It then walks ten properties — Correctness, Reliability,
Performance, Scalability, Security, Maintainability, Operability, Usability,
Cost, and Compliance — each with its own decades-old body of practice. The
fetched portion covers that framing plus the full first chapter, Correctness.

Correctness is "getting the right answer": the system computes correctly and
keeps the promises it made about its own state. The chapter's through-line is
Dijkstra's observation that testing shows the presence of bugs, never their
absence — read not as despair but as a design constraint. Since you cannot check
every input and interleaving, correctness is built in layers: requirements that
say what "right" means, designs that make wrong states hard to reach, and tests
that pin behaviour so it can't drift.

The lane runs from the keyboard to the distributed system. Core practices:
whole-team quality, the testing pyramid, trunk-based development, ubiquitous
language, TDD, property-based testing, and mutation testing (the chapter's
single recommendation: measure your tests with mutation testing). The strongest
move is designing wrong states out of existence — Make Illegal States
Unrepresentable — so a whole family of tests is never needed. At the
distributed edge, correctness becomes an agreement problem: consistency models,
contract testing, data contracts, and the transactional outbox for publishing
an event without letting a crash split it from the state change.

The recurring trap is trusting the green checkmark: a passing suite proves only
the behaviours you thought to check, and coverage targets make this worse — a
suite can be 90 percent covered and nearly toothless. The pattern across the
chapter is that correctness is earned in small installments, and the
highest-value work removes the possibility of a bug rather than hunting for its
absence.
