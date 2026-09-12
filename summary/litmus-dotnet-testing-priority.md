---
url: https://dev.to/ebrahimsayed/you-inherited-a-net-codebase-with-zero-tests-now-what-46ca
title: "You inherited a .NET codebase with zero tests. Now what?"
author: Ebrahim Sayed Ebrahim
date_fetched: 2026-07-18
date_published: 2025-03-27
topics:
  - software-engineering-craft
---

A practical guide for .NET developers facing a legacy codebase with no test
coverage, and an introduction to Litmus, a CLI tool the author built to automate
the "where do I start?" decision.

The author dismisses two common approaches — testing files alphabetically
(activity without risk reduction) and writing tests only for bugs just fixed (a
reflex, not a strategy). Instead, he advocates ranking files by a combination of
risk and testability. Litmus scans a repo's git history, code complexity, and
test coverage to group files into three buckets: **Act Now** (high risk, low
coupling), **Next Sprint** (high risk, high coupling), and **Monitor** (lower
risk or too entangled to test profitably).

The key insight is that existing tools like SonarQube don't evaluate
*testability* — whether a file can be tested now or needs refactoring first.
Litmus uses Roslyn to detect six categories of "unseamed dependencies" (drawn
from Michael Feathers' concept of seams): infrastructure calls like
`DateTime.Now`, direct instantiation in methods, concrete constructor
parameters, static calls, async I/O seams, and concrete downcasts. Each is
weighted, with infrastructure calls at 2.0× because they have no substitution
point.

The two-phase scoring formula multiplies risk (churn × inverse coverage ×
complexity) by testability (1 − coupling). A decoupled file keeps its full risk
score; a maximally entangled one drops to near zero — not because it's safe, but
because testing it first would waste a sprint. The tool runs without a server or
config, supports .NET 8–10, and integrates into CI with a quality-gate flag for
failing builds on high-risk files.

---
*Sources: [[raw/litmus-dotnet-testing-priority]]*
*Last updated: 2026-08-01*
