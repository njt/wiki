# Spec-First (spec-first.dev)

A product site demonstrating a spec-first agent workflow end to end: four slash commands (`create-spec`, `review-spec`, `apply-spec`, `deliver-spec`) take a one-paragraph intent to an open pull request, with the spec, an evidence check, and a decision record as the durable artifacts along the way.

---

The demo is a tiny todo app, but the shape of the session is the argument. The human's first message is a sentence of intent, not a ticket. The agent's first moves are to read memory and ask a clarifying question ("Should todos survive a reload?") — ambiguity resolved *before* the spec exists. The spec lands on its own git worktree branch, named `spec/2026-10-27-7b2d`, so the plan itself is version-controlled.

The review stage is the most interesting design decision. It combines two things usually kept apart: a deterministic checker (`check-evidence.py` verifies the spec's evidence resolves — the machine-checkable part) and a fresh-context agent review that edits the spec with a budget of cuts ("Drag to reorder is out, a check for empty titles is in"). Scope reduction is done at spec time, when it costs nothing, rather than discovered mid-implementation.

Implementation then proceeds with a plan confirm ("the store first, then the list, then the tests. Go?"), 9 passing tests, and — the detail that lingers — a written decision record: "localStorage over IndexedDB, because a few hundred todos fit easily." Delivery is gated on build+test and ends in a PR.

## Key quotes

> "Should todos survive a reload?"

The clarifying question before the spec is written. Contrast with agents that silently resolve underdetermined requirements — this workflow makes the question the first artifact.

> "BUILD WITH CUTS. Drag to reorder is out, a check for empty titles is in."

Review as subtraction plus addition. The reviewer isn't grading the spec; it's trimming scope and tightening acceptance criteria before any code exists, when cuts are free.

> "One decision on record: localStorage over IndexedDB, because a few hundred todos fit easily."

The decision record carries its *reasoning*, not just the choice — exactly the kind of provenance that survives when a later agent session asks why.

## Key themes

#concept #pattern #tool

- **Spec as the unit of work**: the branch, the review, and the PR all key off a spec ID, making the spec the durable spine of the change.
- **Two-layer review**: deterministic evidence check + fresh-context agent judgment, neither trusted alone.
- **Cuts at spec time**: scope reduction before implementation, not during.
- **Decision records with reasons**: `.spec-first/decisions/` as agent-readable memory of *why*.

## Opinionated take

This is the most compressed statement of the spec-driven thesis in the wiki: four commands, one page, everything a [[Spec-Driven Development]] advocate claims, shown rather than argued. What distinguishes it from heavier SDD pipelines is the *evidence check* — the spec must resolve against something checkable before it's approved, which is a guardrail most spec workflows lack. It's also honest about smallness: this is a todo app, and the workflow's value at scale is asserted, not demonstrated. The risk it doesn't address is the one [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] names — here the clarifying question was asked, but nothing in the workflow *forces* one; a less careful agent could have defaulted. The demo also quietly shows the human's job shrinking to three "Go"s — which is either the dream or the problem, depending on which [[Human Judgment Doesnt Leave the Software Factory — It Relocates]] conclusion you believe.

## Related

- [[Spec-Driven Development]] — this source is that page's thesis rendered as a four-command product; strengthens it with the evidence-check and decision-record additions.
- [[SDD Case Study — 13 Apps in 70 Days]] — Fontoura's "correct the spec, not the code" discipline at 70-day scale; spec-first.dev is the same loop shown in miniature.
- [[Aviator Verify]] — both make review evidence-based, but Aviator verifies intent against running code after the fact while spec-first checks the spec's evidence before code exists.
- [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] — spec-first.dev's clarifying question is exactly the tripwire bl00cyb asks for; complicates it by showing the question is a demo choice, not an enforced step.

---
*Sources: [[raw/spec-first-dev]], [[summary/spec-first-dev]]*
*Last updated: 2026-10-28*
