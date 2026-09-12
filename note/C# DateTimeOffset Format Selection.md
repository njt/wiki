# C# DateTimeOffset Format Selection

A field guide to picking the right timestamp format at API boundaries: which formats survive the round trip, which silently discard information, and why "we use ISO 8601" isn't a contract.

---

## Key Quotes

> In modern .NET code, use `DateTimeOffset` for timestamps. Unlike `DateTime`, it stores both the clock time and its UTC offset, which is the information most easily lost at a service boundary.

The article's central premise, and correct. `DateTime` with `DateTimeKind.Utc` carries the same instant but loses the offset the moment you call `.ToString()` without also threading `DateTimeStyles` through every call site. `DateTimeOffset` bakes the offset into the type.

> ISO 8601 is a large standard that covers dates, times, UTC, local time with an offset, durations, intervals, and more. It's broad enough that "we use ISO 8601" isn't really a contract on its own. For timestamps exchanged between applications, RFC 3339 defines a much narrower profile of it, and that profile is what most APIs mean when they say ISO 8601.

This distinction is the most valuable sentence in the article. Every team that writes "ISO 8601" in an API spec and then gets confused about `2026-07-14T09:11:30` vs `2026-07-14T09:11:30.123+02:00` vs `20260714T071130Z` needs to read this. RFC 3339 is the subset that actually works as an interoperability contract.

> Use one format for an explicit offset and another when the contract requires UTC with `Z`... Only append `Z` after converting the clock time to UTC. Adding it directly to the example's `09:11:30` moves the timestamp two hours into the future without raising an error.

This is the trap that makes the format-string approach dangerous: `'Z'` is a literal, not a conversion instruction. Appending it to a local time produces a value that parses as two hours later — and no exception fires. The code looks right.

> `System.Text.Json` writes a `DateTimeOffset` in UTC with `+00:00`, but a `DateTime` with `DateTimeKind.Utc` with `Z`.

A surprising behavior that causes real bugs when someone compares timestamp strings for equality or when a test asserts on the exact JSON output. Both are valid RFC 3339, both parse identically in .NET and JavaScript, but they're different strings — and that's enough to break string-based equality checks.

> `new Date("2026-07-14T09:11:30")` means something different for every visitor, which is exactly what the `s` format produces. `new Date("2026-07-14")` is midnight UTC, so the same string is inconsistent with the rule above. `new Date(1784013090)` is `1970-01-21`, because the constructor expects milliseconds.

Three traps at the JavaScript boundary, each silent. The `s` format (`"yyyy-MM-ddTHH:mm:ss"`) drops the offset, so the browser fills in local time — every visitor sees a different instant. Date-only strings parse as midnight UTC, so users west of Greenwich see the day before. And Unix seconds fed to a millisecond constructor produce January 1970 instead of July 2026.

> When the input has no offset, `TryParse` fills in the offset of the machine that runs it, so the same request parses differently on a developer's laptop and on a server in another region.

The environmental-dependency trap. `TryParseExact` with explicit `DateTimeStyles` is the fix, but only if you know the trap exists. The article makes it visible.

## Key Themes

- `#pattern` — Format selection as API contract design. The format is part of the interface, not an implementation detail.
- `#pattern` — Round-trip fidelity as the selection criterion. Judge a format by what it preserves when parsed back, not by what it looks like.
- `#tool` — C# `DateTimeOffset` as the correct type for timestamps. The type system can enforce what conventions can't.
- `#tool` — `System.Text.Json` default behavior as a deliberate design choice (RFC 3339, variable precision, UTC as `+00:00`), not a bug.
- `#concept` — RFC 3339 as the narrow, interoperable profile of ISO 8601. The distinction most API docs fail to make.

## Critical Analysis

This is a reference-quality article. The single-timestamp comparison table, the format string pair for offset vs UTC, and the TypeScript/JavaScript trap trio are each worth the read on their own. Together they form the kind of dense, example-driven technical writing that actually changes how someone writes code tomorrow.

**What it gets right:** The round-trip-as-criterion framing is the correct one. Developers spend too much time asking "what does this format look like?" and not enough asking "what survives when I parse it back?" Nilsson makes this the organizing principle and every recommendation follows from it. The `O` vs RFC 3339 distinction (seven fractional digits for .NET-to-.NET pipe, three for browser-facing APIs) is the kind of practical detail that blog posts usually skip.

**What's missing:** Time zone IDs get a single sentence ("send the IANA time zone ID separately"), which is correct but undersells the complexity. An offset tells you where the clock was, not which time zone it was in — CET and CEST share `+02:00` at different times of year, and no offset can disambiguate them. A follow-up article on the offset-vs-zone distinction would complement this one well. The article also doesn't address `NodaTime`, which is the community-standard alternative to `DateTimeOffset` for anything involving time zones, arithmetic across DST boundaries, or multiple calendar systems.

**Where it fits in the wiki:** This is one of a small but growing cluster of C#/.NET pages (alongside [[Span-First CSharp — Designing Around SpanT]], [[CQRS Pattern in C# and Clean Architecture]], [[Document Generation in .NET]], and [[Celly — Native .NET CEL Implementation]]). Like [[Span-First CSharp — Designing Around SpanT]], it's a single-concept design guide that earns its keep by telling you not just how but *when* — and when not to.

**The deeper pattern:** This article is fundamentally about API contract design. The format you choose is a promise to every consumer, and the mistakes Nilsson catalogues (forgetting the offset, appending `Z` as a literal, trusting `TryParse` with no offset) are all examples of contracts that are underspecified in ways that produce valid-but-wrong output. The same pattern shows up in [[HTTP API Design Guide (Heroku)]] (the immutability contract: once published, an API version is a promise), in [[Designing a Passively Safe API]] (idempotency as a contract), and in every page that argues explicit beats implicit at system boundaries.

The article's closing advice — "a wrong timestamp is still valid, which is why these mistakes rarely raise an error" — is a concise restatement of why API design is hard in general. The system fails silently, and you only find out when someone in another time zone sees the wrong date.

---
*Sources: [[raw/csharp-datetimeoffset-formats-iso-8601-rfc-3339-json-and-unix-time]], [[summary/csharp-datetimeoffset-formats-iso-8601-rfc-3339-json-and-unix-time]]*
*Last updated: 2026-08-07*
