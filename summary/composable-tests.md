---
url: https://newsletter.kentbeck.com/p/composable-tests
title: "Composable Tests"
author: Kent Beck
site: Software Design: Tidy First? (newsletter.kentbeck.com)
date_fetched: 2026-08-21
date_published: unknown (not present in fetched source)
tags: [testing, tdd, test-desiderata, kent-beck, software-design]
topics:
  - guardrails-and-feedback-loops
---

## The Distinction

Kent Beck's [Test Desiderata](https://kentbeck.github.io/TestDesiderata/) lists 12 desirable properties of tests. Two of them look like the same thing, and this post is Beck finally finding an example that separates them:

- **Isolation** — one test's result is independent of any other test's result.
- **Composition** — a suite of tests, run together, gives confidence that no individual test gives alone.

Isolation is a property of a *single* test relative to its neighbours. A test that builds its own fixture from scratch is isolated by construction — Beck calls this "the same property as referential transparency in functional programming." xUnit frameworks enforce it by constructing a fresh test object per test and running `setUp()` first. He notes NUnit as an exception that reuses instances, "opening the door to breaking isolation."

Composition is a property of the *suite*. The whole should be predictive even though no individual part is comprehensive.

## The Example: Copy, Paste, Extend

The canonical growth pattern. You write `test1()`: construct, call `doSomething()`, assert. Then you need the next behaviour, so you copy `test1`, paste, and append a call to `nowSomethingElse()` with a second assertion. Beck reports seeing this repeated "6 or 7 times" — "That last test is pretty hard to read."

The key observation: **`test2` cannot pass if `test1` fails.** Every non-compliant program caught by `test1` is also caught by `test2`. So there are three options that preserve identical coverage:

1. Leave both tests.
2. Delete `test1`.
3. Simplify `test2`.

## Pruning

Beck rejects (1) on aesthetic grounds — "& don't discount aesthetics" — because the tests are visibly redundant and "Something *must* be wrong."

He rejects (2) because deleting `test1` costs another Desiderata property: tests should be **specific**, meaning that when one fails you know where the problem is.

He chooses (3): trim `test2` down to call `doSomething()` without asserting on it, and assert only on `nowSomethingElse()`. The *composition* of `test1 + test2` loses no predictive power and no specificity. In fact he argues it may gain specificity, since `test1` can now fail while `test2` passes.

## N × M

The scaling argument, and the part Beck's own commenters singled out. Given 4 ways of computing interest and 5 ways of reporting it, brute force is 20 tests. If the two dimensions are genuinely separated "in a functional programming sense," composition needs 10:

- 4 tests for computation
- 5 tests for reporting
- 1 test that combines them, to demonstrate they are wired together

Beck is explicit that this isn't free: it "requires some thought, some inference, some design (to make the orthogonal dimensions demonstrably orthogonal)." The return is tests that are faster, more readable, easier to change, more specific, and less sensitive to structure changes.

## The Critique Section

Beck pre-empts the reaction he says he reliably gets from experienced testing-developers: *"I would never reduce the assertions in a test."*

> This seems to me to be a reaction based in fear, not in principle. We worked *so hard* to get to write tests at all. We can't make them *worse*.

His answer: composition isn't making tests worse, it's optimising the suite as a whole against several valuable properties at once rather than optimising each test in isolation.

## Reader Responses (included in the source)

Two comments are appended to the fetched text:

- One reader links their own post, [verify-only-what-you-need](https://henko.net/blog/verify-only-what-you-need/), reaching a similar conclusion and connecting it to Separation of Concerns, DRY, and SRP. Their ideal: "only a single test would fail whenever an error occurred."
- A sharper objection: if `doSomething()` has a bug, the state in which `doSomethingElse()` runs is *unknown*, so failures in the second call may not be bugs in the second call. Their two proposed patches: construct the object directly into the required state (`new Whatever(readyForDoSomethingElse)`), or assert the precondition with `assumeTrue(object.isReadyToDoSomethingElse())` — since some frameworks give failed *assumptions* a distinct status from failed assertions, preserving the diagnostic signal without restoring the redundant assertion.

## Note on the Source

The fetched text contains an unmarked CodeRabbit advertisement block (four sections of vendor copy) between the Critique section and the reader comments. It is sponsor content, not Beck's argument, and carries a `*Sponsored by CodeRabbit.*` line at its end.
