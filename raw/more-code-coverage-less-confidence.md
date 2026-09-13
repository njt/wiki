---
url: https://www.typemock.com/more-code-coverage-less-confidence/
date_fetched: 2026-09-13
---

**Code coverage and test confidence should move together. But they don’t always.**

Here’s a result most development teams would celebrate:

Last month: 78% code coverage This month: 91% code coverage ↑ 13%

Nice.

Surely the test suite is getting better.

Maybe.

Now imagine what happened behind that number:

+ 146 new tests + 13% code coverage + 37 duplicate tests + 18 tests touching external resources + 2 minutes added to every CI run

The code coverage went up.

But did your test confidence go up with it?

## Code Coverage Is Useful

Let’s get one thing out of the way: **code coverage is a valuable engineering metric**.

Typemock has provided code coverage capabilities for years, and I wouldn’t want to work on a large codebase without knowing which parts of the production code our unit tests actually execute.

If an important piece of business logic has zero coverage, that’s useful information.

If a pull request suddenly causes coverage to collapse, that’s useful information too.

The problem starts when we turn code coverage from a **signal** into a **score**.

72% 😟 84% 🙂 91% 😎 97% 🏆

Software quality isn’t quite that cooperative.

## What Code Coverage and Test Confidence Actually Measure

Suppose we have this simple method:

```
public decimal CalculateDiscount(Customer customer)
{
    if (customer.IsPremium)
        return 0.20m;
    return 0;
}
```
And this unit test:

```
[TestMethod]
public void CalculateDiscount_Test()
{
    var customer = new Customer { IsPremium = true };
    var result = CalculateDiscount(customer);
    Assert.IsTrue(result >= 0);
}
```
The test executes the premium path.

Coverage goes up.

But the test doesn’t actually verify that a premium customer receives a 20% discount.

`0.01`, `0.10` or `0.19` would all satisfy that assertion.

Code coverage can tell us:

**This code executed.**

It cannot tell us:

**This behavior was correctly verified.**

That’s not a weakness in code coverage.

It’s simply not what coverage measures.

And that’s where **code coverage and test confidence start to diverge**.

## More Tests Can Make the Dashboard Look Better

Now imagine someone decides the project needs more coverage.

They add another test.

Then another.

Or perhaps an AI coding assistant generates several automatically.

Soon we have:

CalculateDiscount_PremiumCustomer CalculateDiscount_PremiumCustomer_ReturnsDiscount CalculateDiscount_WithPremiumCustomer PremiumCustomerGetsDiscount CalculateDiscount_WhenPremiumIsTrue

Five tests.

Perhaps they use slightly different values or assertions, but fundamentally they exercise the same behavior.

Our numbers look excellent:

Tests: ↑ Coverage: ↑ Build: ✓

But the amount of unique protection may have barely changed.

We’ve improved the dashboard without necessarily improving the safety net.

This problem is becoming more visible as AI makes generating tests increasingly cheap. I discussed the impact of duplicate unit tests in the age of AI with Software Testing Magazine.

Generating more tests is easy.

Knowing whether those tests add something useful is much harder.

## More Tests Also Mean More Code to Maintain

Every additional unit test becomes code we own.

When production code changes, tests may need to change.

When a test fails, somebody has to investigate it.

When a supposedly isolated unit test depends on a file, network connection, process, environment variable or other external resource, that dependency can eventually become somebody’s mysterious CI failure.

And when several tests protect essentially the same behavior, developers may have to update all of them after one legitimate change.

So there are some important numbers that don’t appear on the code coverage report:

Maintenance cost: ??? Unique confidence added: ??? Developer attention: ???

Those are much harder to measure than a percentage.

But they’re important.

## When Code Coverage Goes Up but Test Confidence Goes Down

Consider two test suites.

### Suite A

1,200 tests 95% code coverage Slow Some duplication Several external dependencies Frequent maintenance

### Suite B

750 tests 85% code coverage Fast Isolated Tests distinct behaviors Easy to understand

Which would you rather inherit?

There’s actually not enough information to answer.

And that’s exactly the point.

**Code coverage alone can’t tell us which test suite is better.**

Maybe Suite A protects critical behavior that Suite B misses.

Maybe Suite B gives developers far more reliable feedback.

Maybe both have serious problems.

A percentage can’t tell us.

## Code Coverage Shouldn’t Become the Goal

Imagine optimizing a development team around lines of code written.

Developers would become extremely productive:

if (x == 1) return true; if (x == 2) return true; if (x == 3) return true; if (x == 4) return true; // KPI looking fantastic...

We know that’s absurd because lines of code are an output, not the goal.

Code coverage deserves similar caution.

The goal of unit testing isn’t to maximize a percentage.

The goal is to give developers **confidence to change software**.

Code coverage helps us get there.

But so do:

- meaningful assertions
- proper test isolation
- effective mocking
- fast feedback
- understandable tests
- tests that protect distinct behavior
- tests that fail for the right reasons

That’s a much richer definition of test quality than a single percentage.

## What Happens When AI Optimizes for Coverage?

This becomes particularly interesting with AI-generated tests.

Tell an AI:

Increase code coverage to 90%.


And that’s a very measurable objective.

The AI can generate tests, run them, inspect the coverage result and keep generating until the number reaches the target.

82% ↓ Generate tests ↓ 86% ↓ Generate more tests ↓ 91% ↓ Mission accomplished ✓

Technically, it succeeded.

But did we ask it the right question?

Perhaps a better objective would be:

Increase meaningful protection of important application behavior.


That’s much harder to turn into a percentage.

It’s also much closer to what developers actually want.

## Ask One More Question

When code coverage goes from:

82% → 91%

celebrate it.

Then ask:

**Why did it go up?**

Did we cover previously untested behavior?

Excellent.

Did we add important edge cases?

Great.

Did we protect against a bug that could otherwise return?

Perfect.

Or did we simply add another collection of tests that happen to execute more lines?

That’s a different result.

And that’s where **code coverage and test confidence** need to be considered separately.

## Coverage + Test Quality

This is also one of the reasons we’re working on Typemock Test Review.

Traditional test results tell us whether tests passed.

Code coverage tells us which production code executed.

Test Review looks at tests from another direction, including whether tests duplicate existing behavior and whether supposedly isolated tests access external resources.

These are complementary signals:

Test Result → Did the test pass? Coverage → What code executed? Test Review → What deserves attention?

No single metric can tell you whether a test suite is good.

And that’s probably how it should be.

Software engineering rarely fits neatly into one percentage.

So the next time your code coverage dashboard climbs from 82% to 91%, enjoy the green arrow.

Just don’t confuse it with confidence.

Because sometimes you can have **more coverage and less confidence**.
