---
url: https://radan.dev/articles/frozen-test-fixtures
date_fetched: 2026-07-05
backfilled: true
---

# Why frozen test fixtures are a problem on large projects and how to avoid them

Tests grow to thousands


All make their claim on fixtures

Frozen by demands

*An ancient Japanese Haiku about a common problem with software test fixtures*

## Act 1: The problem, frozen fixtures

Fixtures have a lot going for them: super fast, clearly structured, reusable across tests …

That last one is also the source of a common problem in large test suites. Every time you change fixtures you risk **falsely breaking** some tests. Meaning: *the test fails even though the feature it tests still works*. This is because every test makes assumptions about the fixtures. This is necessarily part of the test setup, even if it is not explicit in the test code. If the code breaks those assumptions the test itself will no longer work. The more tests there are, the more likely you are to falsely break some of them when you change fixtures.

This is why sufficiently complex data model fixtures tend to become frozen after a certain number of tests. If you aren’t careful, when you get to 1000s of tests, making any change to fixtures can break 10s or even 100s of unrelated tests. It becomes really hard to fix them so you try to avoid directly modifying the fixtures at all. You start to work around it (more on that below) and they stop changing. Hence, **frozen** fixtures.

Thankfully, there are ways to write tests to minimise this effect but it requires discipline.

## Act 2: The bad solutions

First, let me go over 2 approaches I’ve seen on projects and why I think they’re bad:

- **If current fixtures can’t be reused, create new ones.**This is especially prominent in multi-tenant applications: create a brand new tenant in fixtures just for the new tests you’re adding. This is a road of ever increasing fixture size. It becomes really hard to understand which fixtures are for which tests and the testing database starts to become larger and larger. Reviewing existing fixtures for reuse becomes harder. It becomes easier to just add new fixtures for the next test which makes the problem worse.
- **Use code inside the test to modify the fixture records just for this test.**It seems obvious: let’s just modify the DB to match the state we need for the test. Congratulations! You’ve started to re-discover factories, except you’re doing it ad-hoc. If you start going down that road, consider using both fixtures and factories. I’m not being sarcastic, this combination can work really well.

## Act 3: The right solution

First, recognise that every test is written to test a specific property of the code. It should be red if the property breaks and green if satisfied. Diverging from it in any direction is bad, in different ways:

- A test that passes while the property breaks gives us false confidence. That’s obviously bad because we could ship a bug.
- However, a test that breaks while the property holds distracts us with false information. That’s also bad because it is wasting our precious development time and reducing our confidence in our test suite.

To put it zenly1 : **a test should test only that which it is meant to test, no more and no less**.

A great solution to remedy frozen fixtures is turning this principle to **11**2.

### Test only what you want to test

This means getting into the habit of asking yourself what a specific test **actually** tries to test. Then, write the test code to directly test exactly that property. This is not trivial but it becomes effortless with practice. Writing good tests is a skill that needs practice *like any other programming skill*.

I know that this still sounds abstract so here are 2 very concrete examples.

### Example 1: Testing collection content

Testing collections is especially problematic because it has to involve multiple records. This means you’re probably using fixture records that are also used in many other tests. Either that or your fixtures list is crazy long.

Let’s say you are testing a scope on a model. You might be tempted to write something like:

1
2
3

test "active scope returns active projects" do
    assert_equal [projects(:active1), projects(:active2)], Project.active
end


This test has just made it impossible to introduce another active project without breaking it, even if the scope was not actually broken. Add a new variant of an active project for an unrelated test and now you have to also update this test.

Instead, try this:

1
2
3
4
5
6
7

test "active scope returns active projects" do
    active_projects = Project.active
    assert_includes active_projects, projects(:active1)
    assert_includes active_projects, projects(:active2)
    refute_includes active_projects, projects(:inactive)
end


The test will now:

- Fail if the scope no longer includes active projects.
- Fail if the scope now includes inactive projects.
- **Not be affected when new projects are added to fixtures.**

This last one is key. By slightly rewriting the test, we’ve avoided freezing fixtures.

### Example 2: Testing collection order

A related example is checking that a returned collection is in the correct order.

You might be tempted to do something like this:

1
2
3

test "ordered sorts by project name" do
  assert_equal Project.ordered, [projects(:aardvark), projects(:active1), projects(:inactive)]
end


Instead, think like a zen master: to test sorting, *test that it is sorted*:

1
2
3
4

test "ordered sorts by project name" do
  names = Project.ordered.map(&:name)
  assert_equal names, names.sort
end


The test will now:

- Fail if the collection is not sorted.
- **Not be affected by any other change.**

To test a specific case of ordering, focus the test even more and only test that very specific ordering. For example, imagine you just fixed a bug where non latin characters were incorrectly sorted and you want to add a regression test. Do it this way:

1
2
3
4
5
6

test "ordered correctly sorts non latin characters" do
  # Č and Ć are non latin letters of the Croatian alphabet and unfortunately
  # their unicode code points are not in the same order as they are in the
  # alphabet, leading vanilla Ruby to sort them incorrectly.
  assert_equal [projectĆ, projectČ], Topic.ordered & [projectČ, projectĆ]
end


The test will now:

- Fail if the non latin characters are incorrectly sorted.
- **Not be affected by any other change in sorting logic.**

Rewriting the test slightly made it both more precise and not freeze the fixtures.

## Act 4: So … this makes fixtures better than factories?

Now that you know how to minimise fixtures’ downsides without sacrificing any of the benefits, surely, this means they’re better than factories? Right?

Fixtures vs factories is one of those topics that you really wouldn’t expect people to have strong feelings about but somehow they do. I like to irritate people by being pragmatic and not picking a side.

Sometimes I use fixtures sometimes factories. They have different tradeoffs and each could fit a different project better.

Sometimes I decide to go wild and use both, because that way I can annoy everyone at once!

Which is why I didn’t write an article about which one is better, enough digital ink has been spilled on that hill. I did write before about a principle that makes factories easier to use, if that is something you’re interested in.

This article sparked a lively discussion on Hacker News.


## Footnotes

- Bluntly but with a zen overtone. If you don’t like this, tough luck, ever since I became a dad I have an excuse for such jokes. ↩
