---
url: https://slicker.me/c_sharp/span-first.html
title: "Span-First C#: Designing Around Span<T>"
author: unknown
date_fetched: 2026-07-25
date_published: unknown
site: slicker.me
---

# Span-First C#: Designing Around Span<T>

## What Span<T> Actually Is

`Span<T>` is a `ref struct` that wraps a pointer and a length, offering "a type-safe, bounds-checked view over contiguous memory — array, stack, or unmanaged." It does not own the memory it views and does not allocate a copy. Reads and writes through a span go directly to the underlying storage.

```csharp
int[] source = { 1, 2, 3, 4, 5 };
Span<int> view = source.AsSpan(1, 3); // { 2, 3, 4 }, no copy
view[0] = 99;
// source is now { 1, 99, 3, 4, 5 }
```

Three sources spans can wrap: managed arrays (including `string` via `AsSpan()`, any `T[]`), stack memory via `stackalloc` blocks, and unmanaged memory from native interop, accessed through `MemoryMarshal`.

A key clarification: a span over a managed array does not pin it. The GC can still relocate the underlying array between accesses. However, because `Span<T>` is stack-only, "the JIT guarantees no GC-relocating operation can happen while a span referencing movable memory is live across it."

## The Span/Memory Type Hierarchy

| Type | Mutability | Storage | Heap-storable | Typical Use |
|------|-----------|---------|---------------|-------------|
| `Span<T>` | read/write | stack, array, unmanaged | no | synchronous hot-path parameter |
| `ReadOnlySpan<T>` | read-only | stack, array, unmanaged, string data | no | parsing, string slicing |
| `Memory<T>` | read/write | array, unmanaged only | yes | async methods, fields, closures |
| `ReadOnlyMemory<T>` | read-only | array, unmanaged only | yes | async parsing, cached buffers |

`Memory<T>` serves as the heap-storable counterpart, able to live in fields, cross `await`, or be captured in lambdas. You call `.Span` on it to get a `Span<T>` for synchronous work. It cannot wrap `stackalloc` memory because stack memory doesn't persist long enough.

## Ref Struct Constraints

`Span<T>` is declared `ref struct`, which is what makes it safe to hand out pointers to stack and GC-tracked memory without a runtime lifetime tracker. The compiler enforces lifetimes instead by disallowing anything that would let a span escape its current stack frame.

**What's allowed:** local variables, method parameters and return values (tracked via `ref` lifetime rules).

**What's disallowed:** instance fields of a class, instance fields of a non-ref struct, generic type arguments (pre-C# 13 `allows ref struct`), captured in a lambda or local function closure, across an `await`, in an iterator (`yield return`), boxed to `object`.

C# 13 introduced `allows ref struct` as a generic constraint, letting generic code accept ref structs including `Span<T>`. However, it "does not lift the field, closure, or async restrictions."

## Slicing and Indexing

Slicing produces a new `Span<T>` over a subrange of the same backing memory using pointer arithmetic and a length — nothing is copied. Range and index syntax works on spans similarly to arrays and lowers directly to `Slice` calls. Both throw on invalid ranges — bounds checking is performed once per slice rather than per element.

## Stack Allocation with stackalloc

Pairing `stackalloc` with `Span<T>` gives a stack-allocated buffer with array-like indexing and no GC pressure, suitable for small, short-lived buffers. A `stackalloc` span is only valid for the current stack frame. Conventional guidance recommends keeping `stackalloc` under roughly 1–2 KB and avoiding it in recursive code. For size-dependent inputs, the article suggests using `ArrayPool<T>.Shared` instead.

## Span-First Parsing

Parsing is where the span approach shines most visibly. String-based parsing allocates a new string for each substring or split segment, while span-based parsing walks indices over the original buffer.

String-first approach (allocates per field):
```csharp
string line = "2026-07-23,LEWISVILLE,72.4,SUNNY";
string[] parts = line.Split(',');
string date = parts[0];
double temp = double.Parse(parts[2]);
```

Span-first approach (zero intermediate string allocations):
```csharp
ReadOnlySpan<char> line = "2026-07-23,LEWISVILLE,72.4,SUNNY";
ReadOnlySpan<char> remaining = line;

int comma = remaining.IndexOf(',');
ReadOnlySpan<char> date = remaining[..comma];
remaining = remaining[(comma + 1)..];

comma = remaining.IndexOf(',');
ReadOnlySpan<char> city = remaining[..comma];
remaining = remaining[(comma + 1)..];

comma = remaining.IndexOf(',');
double temp = double.Parse(remaining[..comma]);
```

This works without allocating because numeric parsing methods like `double.Parse` and `int.Parse` have `ReadOnlySpan<char>` overloads since .NET Core 3.0. The article also mentions that `SearchValues<T>` (available since .NET 8) precomputes efficient lookups for repeated `IndexOfAny` calls.

## Span-First API Design

The design principle is that the canonical overload takes a `ReadOnlySpan<T>`, and array/string overloads become thin convenience wrappers over it. Starting from the span means the array overload is free, as `byte[]` implicitly converts to `Span<byte>`. Callers with a `Memory<byte>`, `stackalloc` buffer, or slice get the zero-copy path without a separate implementation to maintain. The same logic applies to string-accepting APIs — a method taking `ReadOnlySpan<char>` accepts a `string` for free via implicit conversion.

## MemoryMarshal and Interop

`System.Runtime.InteropServices.MemoryMarshal` serves as the escape hatch for reinterpreting spans across type boundaries without copying. `MemoryMarshal.Cast<TFrom, TTo>` reinterprets the underlying bytes directly. It requires both types to be unmanaged (no references) and the source length to be a whole multiple of the target element size. The article warns that `Cast` and `CreateSpan` bypass normal type safety — struct layout, padding, and endianness are not validated.

## The Async Boundary Problem

The most common span-first failure mode is reaching for `Span<T>` in a method that also needs to `await`. A span cannot be a field of the compiler-generated state machine. The fix is a layered design: `Memory<T>` at the async layer, `Span<T>` at the synchronous leaf.

## Allocation Comparison

Representative BenchmarkDotNet-style figures for parsing a 40-character CSV row 1,000,000 times:

| Approach | Mean Time | Allocated / op | Gen0 Collections |
|----------|-----------|---------------|------------------|
| `string.Split(',')` + `Parse` | ~410 ns | ~192 B | high |
| `ReadOnlySpan<char>` + manual scan | ~95 ns | 0 B | none |

The gap compounds under load: zero allocations means zero Gen0 pressure, which "means the collector never has to pause a hot request path to reclaim discarded split arrays and substrings."

## Common Pitfalls

1. **Slicing Past the Point of Validity** — The compiler catches direct cases but not every indirect one.
2. **Assuming Slice Is Free of Bounds Checks** — `Slice` and range operators validate `start` and `length` against the source span's bounds on every call.
3. **Treating ReadOnlySpan<char> as Interchangeable with string in Dictionaries** — .NET 9 added `GetAlternateLookup<TAlternateKey>()` to allow `ReadOnlySpan<char>`-keyed lookup against a `string`-keyed dictionary without intermediate allocation.
4. **Forgetting Ref Struct Rules Block LINQ** — `Span<T>` has no `IEnumerable<T>` implementation; a ref struct can't implement interfaces that would require boxing it.

## When Span-First Doesn't Pay Off

1. **Object graphs, not buffers** — It's a memory-layout technique for contiguous primitive or blittable data and offers nothing for code about references between heap objects.
2. **Low call volume** — If a code path runs a handful of times per request rather than in a tight per-element loop, "the allocation avoided is noise next to the rest of the request's cost."
3. **Public library surface for broad consumption** — Ref struct parameters can't be used from `dynamic`, can't cross certain reflection-based or expression-tree-built call sites.
4. **Anything that must cross an await inline** — If the natural shape is "await, touch buffer, await again," the `Memory<T>`/`Span<T>` layering split adds a second method and a seam that may not be worth it for a cold path.

Closing advice: "Reach for span-first where allocation count or GC pressure is measured and matters — parsers, serializers, protocol codecs, hot request paths. Everywhere else, a normal `string`/array-based method that's easier to read is the right default; profile before converting it."
