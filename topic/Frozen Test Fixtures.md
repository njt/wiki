# Frozen Test Fixtures

Radan Skorić diagnoses a universal testing pathology: the larger the project, the more fixture changes break unrelated tests, until the fixture set becomes untouchable — "frozen." The fix isn't switching to factories or adding more fixtures; it's writing assertions that test the property, not the data.

---

## Key Quotes

> This is why sufficiently complex data model fixtures tend to become frozen after a certain number of tests.

The term itself is the insight. "Frozen" captures the social dynamic, not just the technical one: nobody dares touch the fixtures because the blast radius is unknowable. This is [[Write Only Code]] for test data — code nobody reads because the cost of comprehension exceeds the benefit.

> A test should test only that which it is meant to test, no more and no less.

Turned "to 11," as Skorić puts it. The practical translation: `assert_includes` and `refute_includes` instead of `assert_equal` on collections; `assert_equal names, names.sort` instead of hardcoding expected order. Each test pins exactly one property. New data can't break it.

> I like to irritate people by being pragmatic and not picking a side.

On the fixtures-vs-factories holy war. Skorić uses both, sometimes in the same project, "to annoy everyone at once." The real enemy isn't the tool — it's the frozen state where neither tool can evolve.

## Key Themes

- #pattern — Property-based assertions: test the invariant, not the instance data
- #concept — Frozen fixtures as social-technical debt: the team stops touching what they can't safely change
- #pattern — Inclusion/exclusion checks over exact-match assertions for collections
- #concept — Fixtures and factories as complementary, not competing, tools

## Critical Analysis

Skorić's core insight — test the property, not the data — is correct and transferable well beyond Rails. The `assert_equal names, names.sort` pattern is elegant: it tests that sorting happened without caring what was sorted. This is a special case of a deeper principle: **assertions should be stable under valid changes to the system state.**

The article's weakness is that it stops at assertion patterns without addressing the organizational problem. Frozen fixtures don't happen because engineers don't know about `assert_includes` — they happen because nobody feels safe touching the test suite. The fix is as much cultural (making test breakage visible and fixable) as technical. This is where [[Feedback Loop is All You Need]] applies: frozen fixtures are a broken feedback loop. When changing fixtures breaks 100 tests, the correct response isn't "stop changing fixtures" — it's "make the feedback fast enough that fixing those 100 tests is cheap."

The fixtures-vs-factories pragmatism is refreshing but undersold. The real argument for factories isn't that they're "better" — it's that they make the "test only one property" discipline easier by letting you build minimal data per test. Fixtures make that discipline harder because they tempt you to reuse data that carries assumptions you've forgotten. But Skorić is right that you can write correct tests with either tool. The tool matters less than the assertion discipline.

One gap: the article never addresses what to do with an already-frozen fixture set. The patterns work for new tests but give no migration path for a 50,000-line test suite with frozen fixtures. That's arguably a different article, but it's the one most teams actually need.

---

## See Also

- [[Spec-Driven Development]] — same principle at the specification level: test what the system should do, not what it currently does
- [[Feedback Loop is All You Need]] — frozen fixtures are a dead feedback loop; the fix is making the loop live
- [[Harness Engineering]] — tests as computational feedback; brittle fixtures degrade that signal
- [[Elements of Code]] — "wrong in correctable ways"; these assertion patterns make test failures correctable
- [[Compound Engineering]] — frozen fixtures compound silently; the fix adds a system (property-based assertions)
- [[dotnet Slopwatch]] — the inverse problem: tests that pass without testing anything vs. tests that fail without anything breaking
- [[Software Engineering Craft]] — testing craft as a durable engineering practice

---
*Sources: [[summary/frozen-test-fixtures]]*
*Last updated: 2026-05-14*
