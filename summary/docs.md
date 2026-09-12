---
url: https://stryker-mutator.io/docs/
title: "What is Mutation Testing?"
author: Stryker Team
date_fetched: 2026-09-13
topics:
  - software-engineering-craft
  - developer-tools
---

# What is Mutation Testing?

Stryker's documentation landing page explains mutation testing — the practice of injecting small deliberate bugs ("mutants") into production code and checking whether the unit tests catch them. The central claim: line coverage measures whether code was *executed*, not whether a test actually asserted anything about it, so a high-coverage suite can still be toothless.

The page distinguishes the two outcomes that matter. A mutant is *killed* when at least one test fails against the mutated code; it *survived* when every test still passes — evidence that no test actually pins down that behaviour. The ratio of killed to total mutants becomes a measure of test-suite effectiveness, which the page argues coverage alone cannot provide.

Stryker itself is introduced as one tool that "uses one design mentality to implement mutation testing on three platforms" (JavaScript/TypeScript, C#, Scala), promising it is easy to use and fast to run. Its one concrete design claim: it mutates only *source* code, never test code, so that a mutant failing is always a signal about test quality rather than an artifact of the harness.

A worked example walks through `isUserOldEnough(user) { return user.age >= 18; }`, showing two mutators (`BinaryOperator`, which flips `>=` to `>`, and `RemoveConditionals`, which replaces the whole expression with `true`), then shows the clear-text reporter output distinguishing the killed mutant from the survivor — the survivor flagged as "probably a test missing that explicitly tests for age lower than 18."
