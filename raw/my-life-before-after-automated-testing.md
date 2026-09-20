---
url: https://blog.danlew.net/2026/09/15/my-life-before-after-automated-testing/
date_fetched: 2026-09-20
---

# My life before & after automated testing

It’s hard for me to overstate how much my coding has improved since embracing automated testing. And yet, I used to think of it as a waste of time! This post is for my former self, demonstrating why I embrace the practice now.

Allow me to demonstrate with an example. Let’s say I’m adding job cancellation to a work queue. Jobs could be canceled at any time (including mid-execution), so it’s complex and error prone. How do I make sure the code I wrote **works**?

# Before Automated Testing

I develop the feature piece-by-piece, manually testing it every once in a while.

Manually testing is a pain. Each time, I have to reset the database and spin up the server; that takes a couple minutes, so I only fully test every time I’ve hit a major checkpoint in the project. It’s especially tricky to test cancellation during execution; I temporarily modify the code so that I can guarantee the system will be in the right state when I click the “cancel” button.

The feature is almost done but… uh oh! I forgot to handle concurrency. During the refactor, I do an even more slapdash job of testing this time around, resulting in bugs that QA catches later.

We eventually ship. Because there are *always* bugs, users run into one. I fix it, but only manually tested the fix itself. I don’t regression test the rest of the feature, resulting in a new bug appearing in previously functional code. Oops!

A few months later, a coworker writes a seemingly unrelated change to the database layer. It subtly breaks the work queue, and we spend a day trying to figure out why jobs are no longer cancelling.

A year later, we add a “pause” feature to the job queue. This code lives side-by-side with cancellation. It’s been a while, so I’ve forgotten how to properly test the job queue, meaning I have to redevelop the manual tests from scratch. Inevitably, I forget to test some obscure corner cases and introduce subtle bugs that will bite me later.

# After Automated Testing

I write the tests first, then develop the feature while repeatedly verifying code using those tests.

To start, I write empty function definitions for job cancellation. Next, I write automated tests that anticipate how job cancellation will work (targeting those empty functions). It’s a slog, but it gives me a chance to think through the happy path + all corner cases I eventually want to handle. It’s especially good for testing mid-execution cancellation - I can easily simulate any situation in code.

Finally, I get around to implementing the actual feature. As I code, I constantly run my automated tests - at first, not even to verify it’s correct, but to run a debugger through the half-finished code, to check that random snippets of code are doing what I expect. The tests run quickly - I don’t have to spin up a whole server to verify the feature, I just run a small portion of the codebase at a time.

Eventually, tests start passing. Even after a test passes, I realize I can make the coder cleaner or faster - small tweaks that can quickly be verified by my automed tests.

Uh oh! I forgot about concurrency here as well. I add a few more tests relating to concurrency, then begin refactoring. As I refactor, all the old tests are still good, helping me check I didn’t break any old behavior while handling concurrency.

We ship the feature. Automated tests don’t guarantee bug free code, so a customer still runs into a bug. I add a new test that reproduces the exact steps the customer ran into, then refactor the codebase until the test passes (as well as all my old tests).

A few months later, a coworker is working on a seemingly unrelated database change. When she runs all the tests, she discovers that she’s accidentally broken job cancellation. She fixes the problem before it even hits code review.

A year later, we decide to add a job *pausing* feature. I write a bunch more tests, but feel relatively confident that I won’t accidentally break existing cancellation code because all the old tests still run regularly.

# Benefits

From these stories, there are two key benefits of automated testing I want to highlight:

**Tight feedback loops during development** - Notice the time difference between “writing code” and “testing.” When I’m doing manual testing, the cost is so high to test that I only do it every once in a while. With automated testing, I am **constantly** running tests on new code.

That makes a huge difference on feedback timing. Tight feedback loops can be all the difference between strong and weak code because it’s much easier to course correct if you’re constantly getting nudges in the right direction. If your feedback loop is large (e.g. every few hours/days) then it’s much harder to turn the ship around once you find out you’re doing something wrong.

Did the last few lines of code I wrote help? If it took 5 minutes to manually test it, I probably wouldn’t bother finding out. With automated testing, why not try it! Worst-case scenario, I run the tests in the background and forget about them; they don’t interrupt my flow. And commonly, being able to run the tests constantly lets me move faster because I feel more confident in the code I just wrote.

**Quick, cheap, & consistent regression testing** - One big difference in the two stories is how often I go back to test that something *still* works. The cost for manual regression testing is high, so I stop bothering at some point, opening up a pathway for bugs. It’s also inconsistent - there’s no guarantee I tested the code the same way each time when I do it myself.

By contrast, with automated testing, it costs me almost nothing to repeatedly run regression tests. Sure, there was the high upfront cost of writing the tests, but that cost is amortized over the long development cycle of the feature. Eventually, it’s far *cheaper* for me to have written the tests, because regression testing is always needed.

On top of that, the automated tests make refactoring safer. We’re repeatedly running into bugs - that happens regardless of automated tests - but with automated tests, when I fix them, I can be assured I didn’t break any other existing behavior. That’s why __Martin Fowler’s “Refactoring”__ essentially boils down to “write tests, then refactor.”

Also, these regression tests don’t just help me - they help *everyone*, even people who know nothing about this feature. Notice above how the regression test informed a coworker they’d accidentally broken this feature. Not only does it catch the bug early, but when they go to fix it, the tests themselves act as documentation of how the feature is used.

The two benefits above are what have dramatically changed how I develop code. While working on a feature, I’ve got tight feedback loops, speeding up development. After releasing it, I’ve got regression testing to ensure the code continues to work.

This post is long enough already, but there are other benefits to automated testing I want to call out as well:

**Ability to set up complex situations in tests** - manually testing all corner cases is tricky. Automated testing lets you craft the perfect path to reproduce rare circumstances.

**Backbone of CI/CD** - If you want the benefits of continuous deployment, you’re gonna need automated tests to reduce risk of regressions.

**Better code architecture** - adapting code for automated testing improves code design. This idea could be a long post on its own, so you’ll just have to trust me on this one, but it’s true!

# Warnings / Caveats

Automated testing is a tool, and like all tools, it can be used incorrectly. I could leave food in the oven all day and burn down my house. That doesn’t mean the oven is a bad tool, I just used it poorly!

This lens is how I view most complaints I’ve heard about automated testing. Like, yeah, if you use it in a shitty way, it’ll feel really bad! So… don’t?

Here’s some warnings / caveats:

**You still need manual testing!** A test is only as good as it’s written, and tests can be bad / not cover all situations. You still need to do manual testing to make sure the test was written *correctly*.

**Don’t religiously test everything.** There are some code paths where you get a ton of bang for your buck when testing, but others you might get minimal returns, or even negative returns (if you have to bend over backwards to make testing work). Start strategically testing where it helps the most, and expand the test coverage only as necessary.

**Kill flakey tests!** A test that randomly passes sometimes is worse than no test at all. Flakey tests should be fixed or deleted; they don’t help anyone by sticking around.

**Don’t spend all your time writing tests for existing code.** A lot of people who first start adopting automated tests feel obligated to go back and write tests for existing code. You lose a *lot* of the advantages of automated testing this way (all the “development” parts of it), plus the code probably already works. Instead, only write automated tests for new code OR before you refactor old code.

**Avoid combinatorial explosions. **Ideally, you mostly want to test isolated parts of the codebase. If you include too many code paths in one test, then testing all variations of that code path results in a huge number of tests (see: __Beware the Integrated Tests Scam__).

# Getting Started

If you’re just getting into it, know that automated testing does NOT have to be an all-or-nothing activity. Start small, with whatever gets you the most bang for your buck. Start where it’s easy to test, instead of trying to cover everything. Gradually, as you get more comfortable with writing automated tests, you can expand.

Eventually, you’ll wonder how you ever got by without good automated tests.
