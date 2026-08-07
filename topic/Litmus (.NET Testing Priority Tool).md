# Litmus (.NET Testing Priority Tool)

Litmus is a free, MIT-licensed .NET CLI tool that tells you which files to test first in a codebase with zero test coverage. It's the automated answer to the problem every .NET developer has faced: staring at an empty test project with no idea where to start. Two commands (`dotnet tool install --global dotnet-litmus && dotnet-litmus scan`) produce a priority-ranked list bucketed into Act Now, Next Sprint, and Monitor — no server, no dashboard, no config file.

---

## Key Quotes

> "Strategy A — alphabetical order — feels like progress but doesn't address the actual risk."

The seduction of visible progress over real progress. Testing `StringExtensions.cs` before `OrderProcessor.cs` feels productive because you're writing tests, but you're optimizing the wrong thing. This is the same category error as measuring developer productivity by lines of code.

> "Strategy B is not a strategy — it's a reflex."

Writing tests only for bugs you just fixed is cargo-cult TDD. You're not building a safety net; you're closing barn doors after horses. The author's dismissiveness here is earned — reactive testing doesn't compound, it just accumulates.

> "There's literally no substitution point for `DateTime.Now`."

The six-category seam taxonomy is Litmus's real intellectual contribution. It's not just counting coverage — it's evaluating *testability* by scanning for hard dependencies via Roslyn. Infrastructure calls get a 2.0× weight because they're the most resistant to unit testing without refactoring. This is Michael Feathers' *Working Effectively with Legacy Code* operationalized as static analysis.

> "Every score is reproducible."

The author's transparency about the scoring formula (`RiskScore = Churn × (1 - Coverage) × (1 + Complexity)`, then `StartingPriority = RiskScore × (1 - Coupling)`) is a feature, not documentation. It means you can argue with the ranking on first principles rather than trusting a black box. That's the right call for a tool developers need to trust with their sprint planning.

---

## Key Themes

- **#tool** — Litmus CLI: `dotnet-litmus` as a NuGet global tool for .NET 8/9/10
- **#pattern** — Risk-prioritized testing: churn-weighted, coverage-discounted, complexity-amplified scoring as a formal alternative to gut-feel prioritization
- **#pattern** — Coupling as priority discount: the insight that entangled files should be deferred not because they're safe but because testing them first wastes the sprint
- **#concept** — Seam detection via static analysis: using Roslyn to find the six categories of hard dependencies that make code untestable without refactoring
- **#concept** — Testability vs. coverage: existing tools measure what's covered; Litmus measures what's *coverable*
- **#comparison** — Litmus vs. SonarQube: Litmus is narrow and deep (one question, no server), SonarQube is broad (code quality platform, requires infrastructure). Complementary, not competitive

---

## Critical Analysis

**What's genuinely new here:** The coupling discount. Most testing-priority tools stop at "high churn + low coverage = test this." Litmus adds a second pass that discounts risk by entanglement — a file that's high-risk but deeply coupled to everything else is a bad first target because you'll spend the sprint refactoring just to write the first test. This is the kind of practical wisdom that comes from actually doing the work across multiple engagements, not from a white paper.

**What's understated:** The seam taxonomy is doing more work than the scoring formula. Anyone could derive `RiskScore = Churn × (1 - Coverage) × (1 + Complexity)` — it's the six categories of unseamed dependencies that make the tool useful, because they answer "why is this file hard to test?" not just "which file should I test?" The author buries the lede here.

**What's missing:** The article doesn't address whether the tool itself has tests. For a testing-priority tool built by someone who's obviously thoughtful about testing, this is a conspicuous omission. Also: no discussion of what happens when you clear the Act Now bucket — does the tool help you plan the refactoring needed to promote Next Sprint items, or does it just re-rank?

**The business case is stronger than the technical one:** The real audience for this tool isn't the developer who knows they need tests — it's the engineering manager who needs to justify *which* tests to the product manager. "We're testing files alphabetically" is indefensible; "we're testing files ranked by churn × complexity × testability" is a strategy. Litmus turns testing prioritization from a craft intuition into a spreadsheet you can show in a sprint review.

**The `--fail-on-threshold` CI gate is the sleeper feature:** Tools that rank files are useful. Tools that *block merges* when high-risk files go untested change behavior. This is where Litmus crosses from nice-to-have to infrastructure — a quality gate that enforces the priority model at the PR level.

**What comes after prioritization:** Litmus tells you which files to test first. Microsoft's [[Polyglot Unit Testing Agent]] (`code-testing-generator`) can write the tests once you've picked the targets — it detects the repo's language, framework, and conventions, generates idiomatic tests, and verifies they're discoverable by CI. The two tools together form a pipeline: Litmus identifies the riskiest untested code, and the polyglot agent generates the tests. One answers "what to test"; the other answers "how to test it."

---

*Sources: [[raw/litmus-dotnet-testing-priority]]*
*Last updated: 2026-07-18*

## See Also

- [[dotnet Slopwatch]] — another .NET static analysis CLI tool; Slopwatch detects AI-generated shortcuts, Litmus ranks by testing priority. They occupy adjacent slots in the .NET code-quality toolchain
- [[Before Reading Code]] — the five-command git archaeology protocol (churn hotspots, contributor ranking, bug clusters) that Litmus automates and systematizes into a single ranking
- [[Accordant]] — Microsoft's model-based testing framework for .NET using Roslyn source generators; Accordant generates tests for things you know matter, Litmus finds things you didn't know were risky
- [[Auditing Legacy Rails Codebases]] — Piechowski's nine diagnostic questions ("What's the one area you're afraid to touch?") are the human-judgment equivalent of Litmus's automated ranking
- [[brooks-lint]] — cites Michael Feathers' *Working Effectively with Legacy Code* directly; its T5 "Coverage Illusion" decay risk is what Litmus explicitly fights
- [[Code Cleanliness and Coding Agents]] — the SonarSource study showing pre-commit static analysis has measurable ROI; Litmus answers *which* files to clean before an agent touches them
- [[A Practical Guide to Brownfield AI Development]] — "agent autonomy requires structure, but legacy systems don't have it"; Litmus identifies where to add structure first
- [[CI Forge (ciforge)]] — the pattern of a CLI tool replacing SonarQube for solo devs; Litmus could fit as one scanner in a similar pipeline
