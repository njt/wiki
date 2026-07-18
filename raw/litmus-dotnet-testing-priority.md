---
url: https://dev.to/ebrahimsayed/you-inherited-a-net-codebase-with-zero-tests-now-what-46ca
title: "You inherited a .NET codebase with zero tests. Now what?"
author: Ebrahim Sayed Ebrahim
date_fetched: 2026-07-18
date_published: 2025-03-27
---

# You inherited a .NET codebase with zero tests. Now what?

Every .NET developer eventually faces a codebase with no tests — an empty test project or one with "three tests from 2019 that no longer compile." The challenge: where to begin?

## Two Wrong Strategies

The author identifies two common but inadequate approaches:

**Strategy A – Alphabetical order:** Testing files in sequence regardless of actual risk. The author notes this "feels like progress but doesn't address the actual risk," leading to outdated files getting coverage while high-churn files stay unprotected.

**Strategy B – Reactive testing:** Writing tests only for bugs just fixed. The author calls this "not a strategy — it's a reflex."

## The Third Way: Build a Priority List

The author references Roy Osherove's *The Art of Unit Testing*, which recommends ranking files by risk and testability. But since manual scoring across hundreds of files isn't practical, the author automated it.

## Introducing Litmus

Litmus is a .NET CLI tool that ranks files by testing priority. Setup requires two commands:

```
dotnet tool install --global dotnet-litmus
dotnet-litmus scan
```

No server, dashboard, or config file needed. It outputs files grouped into three buckets: **Act Now**, **Next Sprint**, and **Monitor** — ranked by commits, coverage, complexity, coupling, risk, and priority.

## The Key Insight Competitors Miss

Existing tools like SonarQube and coverage platforms don't evaluate whether a file is *testable right now* or needs refactoring first. The author draws on Michael Feathers' concept of "seams" from *Working Effectively with Legacy Code* — points where you can substitute dependencies during testing without modifying production code.

Litmus uses Roslyn to scan for six categories of "unseamed dependencies":

1. **Infrastructure calls** — e.g., `DateTime.Now`, `File.ReadAllText()`, raw `new HttpClient()`
2. **Direct instantiation in methods** — concrete classes created inside method bodies
3. **Concrete constructor parameters** — e.g., `OrderService` instead of `IOrderService`
4. **Static calls** — methods with no instance to substitute
5. **Async I/O seam calls** — `await _httpClient.GetAsync()`, `await _db.SaveChangesAsync()`
6. **Concrete downcasts** — `(ConcreteType)expr` that bypasses interface abstractions

Each signal has its own weight. Infrastructure calls (2.0×) are weighted most heavily because "there's literally no substitution point for `DateTime.Now`."

## The Two-Phase Scoring Formula

**Phase 1 – Risk:** `RiskScore = Churn × (1 - Coverage) × (1 + Complexity)`. Frequently changed files with low coverage and high complexity score highest.

**Phase 2 – Starting Priority:** `StartingPriority = RiskScore × (1 - Coupling)`. Risk is discounted based on how entangled the file is. A decoupled file keeps its full score; a maximally entangled one drops to zero priority — not because it's safe, but because testing it first would waste the sprint.

The three output buckets derive from this: Act Now (high risk, low coupling), Next Sprint (high risk, high coupling), and Monitor (lower risk or too entangled).

## No Tests Yet? That's the Point

The `--no-coverage` flag skips test runs entirely for codebases starting from zero. Files are still ranked by churn, complexity, and coupling.

## Drill-Down and Progress Tracking

The `--detailed` flag shows method-level data within files, revealing, for example, that `ValidateInput` has "zero coverage and non-trivial complexity" while `ProcessOrder` is partially covered. Users can export baselines with `--output baseline.json` and compare later with `--baseline baseline.json`, generating a Delta column for sprint retrospectives.

## CI Integration

Litmus works in GitHub Actions with `fetch-depth: 0` (required for full git history). A `--fail-on-threshold` flag acts as a quality gate — any file exceeding the risk threshold fails the build.

## How It Differs from SonarQube

The author positions them as complementary: "SonarQube is a broad code quality monitoring platform" requiring a server, while Litmus answers one specific question about where to start testing, runs without a server, and is free.

## The Author's Motivation

The author explains building the tool after repeatedly performing the same manual exercise — "scanning git logs, cross-referencing with coverage reports, eyeballing the source for testability" — across engagements. The scoring model is intentionally transparent: "Every score is reproducible."

## Practical Details

- **Supported runtimes:** .NET 8, 9, and 10
- **License:** MIT
- **NuGet package:** `dotnet-litmus`
- **Docs:** ebrahim-s-ebrahim.github.io/litmus

*Fetched from DEV Community on 2026-07-18*
