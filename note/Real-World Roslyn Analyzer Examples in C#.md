# Real-World Roslyn Analyzer Examples in C#

Nick Cosentino's field guide to shipping Roslyn analyzers that actually enforce team standards, not the "hello, diagnostic" skeletons the official docs stop at. Three complete analyzers — async-method naming, null-guard detection on public APIs, and a structured-logging validator — each with full `DiagnosticAnalyzer` code, a code fix where relevant, edge-case handling, and `Microsoft.CodeAnalysis.Testing` tests. The argument throughout: the conventions a team cares about most are exactly the ones code review is worst at enforcing, and the compiler can enforce them if someone writes the analyzer.

---

## Key Quotes

> "most of them stop at 'hello, diagnostic.' They show you enough to understand the API and then leave you to figure out how a production-quality analyzer actually looks. That gap is expensive when your team needs something working by Thursday."

The framing that makes this worth a page: the Roslyn API is well documented, but the *gap* between API knowledge and a shipping rule is where teams stall. Every example below is aimed at closing that specific gap.

> "When a developer sees `DoWork()` in a code review, the missing suffix makes it look synchronous -- reviewers routinely overlook the `Task` return type and approve fire-and-forget bugs that surface in production under load."

The quiet insight of TEAM001 is that the bug isn't the naming — it's that the naming *hides* the asyncness from a reviewer scanning at speed. The analyzer doesn't fix a style preference; it removes a class of misread.

> "The key insight here is using `Renamer.RenameSymbolAsync` -- it ensures every reference across the entire solution gets updated atomically, not just the declaration."

This is the production detail a tutorial normally skips. A naive fix appends `Async` to the declaration and silently breaks every call site; `Renamer` walks the solution graph and updates all of them. The hard part of a code fix is never the rename, it's the rename *everywhere*.

> "A single developer who habitually reaches for string interpolation in log calls can silently corrupt an entire service's observability data -- and the problem is completely undetectable at runtime until observability costs spike or a log query returns zero results."

TEAM003's case for existing at all: interpolation bakes values into the message body, so message templates stop being consistent, and every aggregator that groups by template quietly loses the events. A bug with no runtime symptom is precisely the bug only a compile-time check can catch.

> "warnings get deferred; errors force the fix."

The whole article's philosophy in five words. TEAM003 fires as `DiagnosticSeverity.Error` not because interpolation is a crash, but because in any service that depends on structured-log queryability there is no legitimate use of it — and deferring it is how it ships.

## Key Themes

- #concept **Deterministic enforcement over review.** All three rules take a decision that a human reviewer makes inconsistently (does this name end in `Async`? is this parameter guarded? is this log message structured?) and move it into the compiler, where it fires at the exact call site before the code runs.
- #tool **Roslyn analyzers** — the .NET static-analysis API (`DiagnosticAnalyzer`, `DiagnosticDescriptor`, `CodeFixProvider`) and the Workspaces `Renamer`.
- #pattern **Choose the right analysis layer.** TEAM001 uses syntax nodes, TEAM002 uses `IOperation` blocks (semantic intent, not textual form), TEAM003 checks syntax deliberately (an interpolated string is unambiguous in the syntax tree). The article's real lesson is not "how to write an analyzer" but "match the analysis level to the question."
- #pattern **False positives are the product.** The override/interface suppression, the default-`null` exclusion, the `if (param is null)` recognition — a rule that nags compliant code gets disabled, and a disabled rule is worth nothing.
- #person Nick Cosentino, Principal Engineering Manager at Microsoft writing as Dev Leader.

## Critical Analysis

The article's most valuable idea is its least flashy one: severity as an encoding of *cost*. TEAM003 is an error because interpolated logs silently corrupt observability — there is no "warn and move on" version of that failure. The two-phase rollout advice (ship new rules at `Warning`, promote to `Error` after a sprint) is the disciplined counterpoint: severity is a rollout tool, not a statement of permanent moral weight. That pairing — "errors force the fix, warnings buy adoption" — is more useful than any single line of analyzer code here.

What the "copy-pasteable" framing underplays is where the real work sits: the edge cases. The null-guard analyzer only scans top-level statements, so guards buried inside `if` blocks are missed; the logging analyzer needs manual extension to cover `ILogger<T>` via DI and `LoggerFactory` patterns. Every one of these is called out, but a team that grabs the code verbatim will spend more time on those extensions than on the initial drop-in. That's not a flaw — it's the honest truth that a production rule is mostly edge-case handling, and the article is unusually transparent about it.

There is also a through-line of self-promotion (each example points to a companion Dev Leader article), but the embedded cross-references are real prerequisites: testing, packaging, and IOperation analysis are each their own deep topic, and the article correctly defers them rather than skimming.

The notable absence is any mention of AI. This is a pre-agent document through and through — and that's exactly why it belongs in this wiki. It shows that "linters beat prompts" and "deterministic checks over instructions" are not new ideas the agent era invented; they are the .NET team's everyday tooling, predating the agents that now rediscover them. A Roslyn analyzer is a guardrail in the most literal sense: a deterministic check that fires identically no matter whether a human or an agent wrote the offending line.

## Related Pages

- [[Guardrails and Feedback Loops]] — this source is the pre-agent .NET instance of that topic's "linters and deterministic checks" thesis, and it strengthens the case that the principle long predates coding agents.
- [[Feedback Loop is All You Need]] — makes that page's "your CLAUDE.md is a suggestion, your linter isn't" concrete: TEAM003 fires as an error and blocks the merge, the exact deterministic feedback instructions can't supply.
- [[Litmus (.NET Testing Priority Tool)]] — same Roslyn substrate, different use: Litmus uses Roslyn-based seam detection to rank files for testing, while this article shows the more common Roslyn application, diagnostics and code fixes.
- [[CSharp DateTimeOffset Format Selection]] — complicates the C# cluster's "the type system can enforce what conventions can't" claim: Roslyn analyzers enforce what the type system *also* can't — naming, null-guard presence, and log-message shape.

---
*Sources: [[raw/realworld-roslyn-analyzer-examples-in-c-naming-null-checks-and-log-validators]], [[summary/realworld-roslyn-analyzer-examples-in-c-naming-null-checks-and-log-validators]]*
*Last updated: 2026-09-13*
