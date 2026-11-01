---
url: https://github.com/owls-on-wires/keplearn
title: "Keplearn — Deterministic Symbolic Regression in Seconds"
author: Chandler Freeman (owls-on-wires)
date_fetched: 2026-11-02
topics:
  - developer-tools
  - databases-and-data
---

Keplearn is a ~7,000-line Rust program that recovers closed-form equations from tabular data: feed it a TSV/CSV on stdin, and it prints candidate laws as JSON — e.g. Newton's inverse-square law from masses and distance. It is a deterministic, single-binary, zero-dependency alternative to the usual genetic-programming/RL symbolic regression stack.

The method is best-first recursive reduction rather than stochastic search. The engine maintains a priority frontier of search nodes, each holding a target column plus a chain of invertible "reductions" already applied to it (dividing out a fitted factor, inverting a shell transform like log/sqrt/arcsin, peeling an affine node). At every node it attempts a terminal fit — an affine monomial whose exponents come from a log-linear regression, a sparse k-term monomial, or composite forms (L2 norms, rationals, trig). Candidates are ranked on an internal 25% holdout with an MDL complexity penalty, so the search prefers compact laws over overfit monsters.

Two details stand out. First, constants are snapped to nearby rationals and to multiples of π, e, and √2 — and each snap is kept only if a verification pass shows the holdout and full-data scores still hold, so pretty numbers can't break the fit. Second, the final `r2` in the JSON comes from evaluating the printed model string, not from any internal fitting stage, which keeps the reported score honest.

On SRBench (60s per-fit budget, one core), Keplearn recovers the ground-truth form on 64.7% of noise-free Feynman datasets — the best published Feynman symbolic rate (AIFeynman's 55.8%) is beaten at a fraction of the compute, and median fit time at noise 0 is 1.3 s with half the suite under a second via an exact-exit path. The trade-off is explicit: on the 138 black-box datasets with no closed-form law, its median test R² is near zero, because Keplearn targets exact analytic laws rather than curve fitting.
