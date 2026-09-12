---
url: https://reubenbrooks.dev/blog/structural-backpressure-beats-smarter-agents/
title: Structural Backpressure Beats Smarter Agents
author: Reuben Brooks
date_fetched: 2026-05-22
date_published: 2026-05-18
topics:
  - guardrails-and-feedback-loops
---

# Structural Backpressure Beats Smarter Agents

Reuben Brooks argues that the most reliable way to enforce critical invariants in AI-generated code is not to make models smarter or write better prompts, but to embed those invariants as structural constraints in the language and tooling the model writes against.

## Behavioral Gates And Structural Gates

A behavioral gate depends on the model remembering the rule. A structural gate is deterministic — a compiler, a type checker, a test runner, a linter, a proof checker. The distinction: behavioral gates are "soft" and live in the model's attention; structural gates are "hard" and produce a concrete pass/fail on the artifact.

> "A behavioral gate depends on the model remembering the rule"

> "Structural gates are different. A compiler, a type checker, a test runner, a linter, a proof checker."

> "deterministic gate gives the loop something firmer than vibes to push against"

> "English is simply the wrong medium in which to enforce it."

## The Substrate Move

Brooks describes moving invariant enforcement out of prompts and into the code's type system and compilation surface. This is the "substrate move" — embedding rules in the substrate (the language, the type system, the toolchain) rather than in instructions to the model.

## Shen-Backpressure

The author's tool combining the Shen language's sequent-calculus type system with code generation for guard types in Go and TypeScript. The tool, shengen, generates sealed constructors from Shen datatype rules. Five default gates: shengen (drift detection), test, build, shen tc+ (spec consistency), tcb audit (no hand-edits to generated code).

> "The generated guard code is sacred; edit it by hand and the audit gate rejects it."

## Proof Chain For Multi-Tenant Auth

Types like `tenant-access` and `resource-access` whose constructors require discharging premises — the proof travels with the value. A constructor won't compile if the prerequisite proofs aren't satisfied.

> "makes the specified invariants practically impossible to bypass by accident"

## From Spec To Guard Types

Shen's sequent-calculus type system allows expressing rules like "to construct a `resource-access`, you must first discharge `tenant-access`" as formal sequents. shengen lowers these into target-language types with sealed constructors that enforce the proof chain at compile time.

## Authorization Without The Hand-Written Check

Once guard types are generated, authorization becomes structurally enforced — code that tries to use a resource-access without first proving tenant-access simply doesn't compile. No hand-written `if` check, no prompt-level reminder, no hoping the model remembers.

## The Ralph Loop

The iterative AI coding loop where the model bounces off failing gates until the artifact satisfies all checks.

> "that short, mechanical 'no' is the backpressure"

## Costs And Limits

Brooks is honest: these invariants are not categorically unbypassable. A determined adversary could work around them. But making the wrong path "structurally difficult and expensive to introduce by accident" is the high-leverage intervention for shipping LLM-generated production code.

## The Thesis

> "you need better backpressure more than you need a better model"

> "capability and certainty are different things"

The closing argument: better deterministic signals about the artifact matter more than incremental model capability improvements, since "this artifact upholds the invariant" is a concrete claim about a single object, not a probabilistic claim about a writer.
