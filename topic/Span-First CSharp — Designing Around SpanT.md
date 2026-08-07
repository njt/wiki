# Span-First C#: Designing Around Span<T>

A practical field guide to C#'s `Span<T>` and the span-first design philosophy: zero-allocation views over contiguous memory that invert the traditional API design pattern, with the canonical overload taking a span and array/string overloads becoming free convenience wrappers.

---

## Key Quotes

> Span<T> is a ref struct that wraps a pointer and a length, offering a type-safe, bounds-checked view over contiguous memory — array, stack, or unmanaged.

The core definition, worth returning to because so much confusion comes from treating spans as owning containers rather than views. A span is a window, not a warehouse.

> A span over a managed array does not pin it. The GC can still relocate the underlying array between accesses.

One of those implementation details that quietly makes or breaks production code. The safety comes from the ref struct constraint (stack-only lifetime), not from pinning.

> Memory<T> serves as the heap-storable counterpart, able to live in fields, cross await, or be captured in lambdas. You call .Span on it to get a Span<T> for synchronous work.

The two-type layering (`Memory<T>` at async boundaries, `Span<T>` at synchronous leaves) is the structural insight the whole design depends on. It is also the discipline most commonly violated in code written by people who learned spans from blog posts.

> Starting from the span means the array overload is free, as byte[] implicitly converts to Span<byte>.

This is the whole "span-first" thesis compressed into one sentence. Write the span overload first, get the array overload for free. The inversion of instinct is the entire design payoff.

> Reach for span-first where allocation count or GC pressure is measured and matters — parsers, serializers, protocol codecs, hot request paths. Everywhere else, a normal string/array-based method that's easier to read is the right default; profile before converting it.

The closing advice is the most honest and useful part. The article earns credibility by telling you when *not* to use its own pattern.

## Critical Analysis

This is a genuinely useful technical guide that does something rare in the genre: it connects the language feature (`Span<T>`) to the *design pattern* (span-first API design) rather than just cataloguing the API surface. The progression from "what is it" through "how the type hierarchy works" to "how to design APIs around it" to "when *not* to bother" is the right pedagogical arc.

The strongest section is the async boundary problem and the `Memory<T>`/`Span<T>` layering pattern. This is where real-world span use most commonly breaks — developers reach for `Span<T>` in an async method, hit CS4012, and either give up or start playing whack-a-mole with compiler errors. The article diagnoses the structural reason (state machines are heap objects, spans can't be heap fields) and gives the structural fix (split the async orchestration from the synchronous transformation).

The weakest section is the `stackalloc` discussion. It mentions the conventional guidance (1–2 KB max, avoid recursion) but doesn't dig into *why* those limits exist — stack overflow on Linux vs. Windows has very different failure modes, and the guard page behavior differs. For an article otherwise rigorous about *why* things work, this section is surprisingly hand-wavy.

The `MemoryMarshal` section is correctly cautionary but undersells the risk. `MemoryMarshal.Cast` doesn't just "bypass type safety" — it bypasses the entire type system, and the resulting bugs are the worst kind: silent data corruption that won't surface until the wrong bytes happen to land in the wrong field. The warning about `[StructLayout]` is one sentence when it deserves a paragraph.

The missing section is **string interop ergonomics**. The article shows `ReadOnlySpan<char>` slicing for CSV parsing but never discusses the practical pain point: most downstream APIs still take `string`, and calling `.ToString()` on every span slice defeats the purpose. The `GetAlternateLookup` mention (for dictionaries) is good, but the broader "how do I actually *use* these spans" question goes unaddressed.

The BenchmarkDotNet comparison (410 ns → 95 ns, 192 B → 0 B) is useful but the article doesn't clarify whether this is per-operation or per-1M-iterations. The 4.3× speedup is plausible; the claim of zero Gen0 collections is the real headline and is structurally credible — no allocations means nothing to collect.

**Verdict:** A strong technical reference that earns its place by being honest about limits. Best consumed alongside a profiler — the closing advice to "profile before converting" isn't boilerplate, it's the thesis.

Steve Gordon's [[Performance Optimization Loop]] provides the methodology that tells you *when* to reach for span-first design: start with production monitoring, profile hotspots, benchmark a baseline, and apply small targeted changes (including span conversion) one at a time with re-benchmarking between each. The span-first guide is the *how*; Gordon's loop is the *whether* and *when*.

## Related Pages

- [[Software Engineering Craft]] — Hub page for engineering patterns and practice
- [[CQRS Pattern in C# and Clean Architecture]] — Another C# design pattern guide; complements span-first as a different layer of the stack
- [[Dapper Performance Trap]] — C# performance analysis at the ORM level; spans operate one layer below
- [[dotnet Slopwatch]] — .NET performance measurement tooling; the profiler you'd use before taking this article's advice
- [[Redb Ecosystem]] — .NET stack overview; the ecosystem spans live within
- [[Celly — Native .NET CEL Implementation]] — A zero-allocation C# library that likely uses spans internally for its expression parsing
- [[C# DateTimeOffset Format Selection]] — Another single-concept C# design guide; where span-first covers the memory layer, this covers the wire-format layer at API boundaries
- [[Observer Pattern to Event-Driven Architecture in Dart]] — Design pattern handbook in a different language but the same structural thinking
- [[Command Line Interface Guidelines]] — Shares the "design the API surface first, implementation follows" mindset

---
*Sources: [[raw/span-first-csharp]]*
*Last updated: 2026-07-25*
