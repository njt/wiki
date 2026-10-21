# Archie — An Architecture Agent for Repositories

Archie (don-smith, v0.3.0) is an open-source architecture agent that installs into a repository as agent skills plus a pinned, hash-verified runtime and works inside your existing coding agent. Its defining move is the refusal to produce an architecture verdict: seven narrow capabilities — assessment, module review, docs with a claims ledger, C4 authoring, conformance onboarding, deterministic dependency contracts, HTML output — each with explicitly bounded result semantics, and an authority stance that keeps every decision with the maintainers.

---

## Key quotes

> "A finding is not a pass … No set of findings adds up to an architecture approval."

The clearest anti-hallucination device in the design is semantic, not technical: results are typed so that none of them *can* be read as sign-off. This is the same insight behind the wiki's verification-first sources, applied to architecture instead of code.

> "Observed code never promotes itself into an active rule; a maintainer writes the rule."

The conformance loop is split so that what-is and what-should-be are separated by a human act. An agent observing the import graph cannot silently legislate — the determinism of the check is only trustworthy because its premise was human-approved.

> "When a capability or one of its dependencies is unavailable, Archie reports that gap rather than simulating the workflow."

A rare explicit honesty boundary: fail by reporting the gap instead of improvising a plausible-looking result. Contrast the pattern this wiki keeps cataloguing, where agents fill silence with confident output.

> "There is no `curl | bash` one-liner, deliberately."

Supply-chain hygiene as product design: you clone, read the commented installer, get hash-pinned tarballs, and the tool never touches your `package.json` or pushes anywhere. Archie is written for an audience that reads installers — its users are its reviewers.

## Themes

- #tool — a concrete open-source project (v0.3.0, MIT, developed in the open)
- #concept — authority and result semantics: findings, rules, and compiled models each carry their own epistemic weight
- #pattern — deterministic conformance checks wrapped around agentic observation
- #pattern — pinned, journal-replayable installation as a trust mechanism

## Analysis

What makes Archie interesting is not that it reviews architecture — it's that it is architected *against* the failure mode of architecture agents: the confident verdict. Every capability's output is designed to be consumed by a human who retains the decision, and the epistemic labels (fact / inference / unknown) plus the named-owning-capability routing make the agent's own uncertainty a first-class output. This is agent UX as epistemology.

The deterministic spine deserves note: dependency contracts are checked by an analyzer, not an LLM, which means the conformance gate is cheap enough to run on every pass and immune to model drift. The agentic layer (assessment, review, docs) sits above it, doing the fuzzy recovery-of-intent work. This is exactly the split the wiki keeps converging on.

The honesty about maturity is also a model for project write-ups: "verified by installing it into real repositories and using it there, not by a formal review gate" — a v0.3.0 that names its own limits.

## Relations

- [[Architecture as Code for Agents]] — Archie strengthens that essay's thesis: its contracts capability is CALM-style machine-readable intent made operational, with the same rule that intent must be stated by a human before it can be enforced.
- [[Claude Is Not Your Architect]] — Holland's polemic against AI as architectural decision-maker gets a constructive counterexample here: Archie routes around the problem by making "approval" structurally impossible for the agent to issue.
- [[Architectural Guardrails for AI-Generated Code]] — Archie's deterministic conformance checks are a concrete instantiation of guardrails-as-code, with the observed-graph→rule promotion gap as the governance detail most guardrail schemes omit.
- [[Deterministic Spine, Agentic Leaves]] — the capability split (analyzer-run contracts under LLM-run assessment and review) is a working instance of that deterministic-core/agentic-shell pattern.

---
*Sources: [[raw/archie]], [[summary/archie]]*
*Last updated: 2026-10-21*
