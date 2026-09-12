---
url: https://radan.dev/articles/frozen-test-fixtures
title: "Why frozen test fixtures are a problem on large projects and how to avoid them"
author: Radan Skorić
date_fetched: 2026-05-14
date_published: 2025-12-09
tags: [rails, testing, fixtures]
topics:
  - software-engineering-craft
---

## The Core Problem (Act 1)

Fixtures are fast, structured, and reusable, but that reusability backfires on large projects. Changing fixtures risks "falsely breaking" tests — tests that fail even though the feature still works. Every test makes implicit assumptions about fixture state.

As the author puts it, "This is why sufficiently complex data model fixtures tend to become frozen after a certain number of tests." With thousands of tests, changing fixtures can break "10s or even 100s of unrelated tests," so teams avoid modifying them altogether — hence the term **frozen fixtures**.

## Two Bad Solutions (Act 2)

1. **Creating new fixtures instead of reusing existing ones** — common in multi-tenant apps. Each new test gets its own tenant. This leads to an ever-growing, incomprehensible fixture database where "reviewing existing fixtures for reuse becomes harder," creating a vicious cycle.

2. **Modifying fixture records inline within test code** — the author calls this "re-discover[ing] factories, except you're doing it ad-hoc." The suggestion: consider using both fixtures and factories intentionally rather than this improvised approach.

## The Right Solution (Act 3)

The guiding principle: "a test should test only that which it is meant to test, no more and no less."

The solution is turning that principle "to 11" — writing tests that directly target exactly one property.

### Example 1: Testing Collection Content

**Bad approach** — brittle assertion using `assert_equal`:

```ruby
test "active scope returns active projects" do
  assert_equal [projects(:active1), projects(:active2)], Project.active
end
```

This breaks if any new active project is added to fixtures, even if the scope still works correctly.

**Good approach** — using `assert_includes` / `refute_includes`:

```ruby
test "active scope returns active projects" do
  active_projects = Project.active
  assert_includes active_projects, projects(:active1)
  assert_includes active_projects, projects(:active2)

  refute_includes active_projects, projects(:inactive)
end
```

Now the test: fails if scope misses active projects or includes inactive ones, but is "not be affected when new projects are added to fixtures."

### Example 2: Testing Collection Order

**Bad approach** — hardcoding the expected ordered array:

```ruby
test "ordered sorts by project name" do
  assert_equal Project.ordered, [projects(:aardvark), projects(:active1), projects(:inactive)]
end
```

**Good approach** — test that it *is* sorted:

```ruby
test "ordered sorts by project name" do
  names = Project.ordered.map(&:name)
  assert_equal names, names.sort
end
```

This fails only if the collection isn't sorted and is "not be affected by any other change."

### Example 3: Regression Test for Non-Latin Sorting

Testing a fix for Croatian alphabet characters (Č, Ć):

```ruby
test "ordered correctly sorts non latin characters" do
  # Č and Ć are non latin letters of the Croatian alphabet and unfortunately
  # their unicode code points are not in the same order as they are in the
  # alphabet, leading vanilla Ruby to sort them incorrectly.
  assert_equal [projectĆ, projectČ], Topic.ordered & [projectČ, projectĆ]
end
```

The intersection operator (`&`) preserves array order from the left side, so this checks that `Ć` comes before `Č` in the results.

## Fixtures vs. Factories (Act 4)

The author refuses to declare a winner: "I like to irritate people by being pragmatic and not picking a side." They use both depending on the project's tradeoffs, and sometimes use both simultaneously "to annoy everyone at once."

A link is included to a related article on "the principle of minimal defaults for factories."

## Key Takeaways

| Principle | Practice |
|-----------|----------|
| Test only one property per test | Narrow assertions to what's actually being verified |
| Avoid exact collection equality | Use inclusion/exclusion checks instead |
| Test the property, not the data | For sorting, test that it *is sorted* |
| Don't freeze fixtures | Write tests that survive fixture additions/changes |
| Both tools have value | Fixtures and factories can coexist |
