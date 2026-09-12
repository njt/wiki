---
url: https://www.devleader.ca/2026/07/24/realworld-roslyn-analyzer-examples-in-c-naming-null-checks-and-log-validators
title: "Real-World Roslyn Analyzer Examples in C# — Naming, Null Checks, and Log Validators"
author: Nick Cosentino
date_fetched: 2026-09-13
date_published: 2026-07-24
topics:
  - software-engineering-craft
  - developer-tools
---

A production-grade tutorial on writing custom Roslyn analyzers for .NET, filling the gap between the documentation's "hello, diagnostic" examples and an analyzer a team can actually ship on a deadline. Nick Cosentino (Dev Leader) provides three complete, copy-pasteable analyzers, each with full `DiagnosticAnalyzer` code, a "How It Works" walkthrough, edge-case and false-positive handling, and test patterns.

The three examples enforce different layers of code quality at compile time. TEAM001 enforces the `Async` suffix convention on methods returning `Task`/`ValueTask` types, with a `CodeFixProvider` that renames across the whole solution via `Renamer.RenameSymbolAsync` rather than text substitution. TEAM002 uses `IOperation`-based analysis to flag public API parameters that are nullable but lack an `ArgumentNullException.ThrowIfNull` guard. TEAM003 bans string interpolation in `ILogger` calls and fires as an **error**, because interpolation silently destroys the message-template consistency that structured-logging aggregators (Seq, Datadog, Elastic) depend on for queryability.

The article then covers packaging all three into one internal NuGet package — a shared rule-ID registry, centralised category constants, `.csproj` configuration, and a `Directory.Build.props` rollout — plus a decision matrix for choosing which analyzer to adopt first. The through-line is that naming, null-guard, and logging conventions are unenforceable by code review alone, but the compiler will enforce them if you write the analyzer.
