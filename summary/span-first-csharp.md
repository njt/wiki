---
url: https://slicker.me/c_sharp/span-first.html
title: "Span-First C#: Designing Around Span<T>"
author: unknown
date_fetched: 2026-07-25
date_published: unknown
topics:
  - software-engineering-craft
---

An in-depth guide to `Span<T>` and its ecosystem in modern C#, advocating for
"span-first" design in performance-sensitive code paths.

`Span<T>` is a `ref struct` wrapping a pointer and a length — a type-safe,
bounds-checked view over contiguous memory (arrays, stack, or unmanaged) that
never allocates a copy. Its heap-storable counterpart, `Memory<T>`, enables
the same zero-allocation pattern across `await` boundaries. The article
covers the full type hierarchy (`Span<T>`, `ReadOnlySpan<T>`, `Memory<T>`,
`ReadOnlyMemory<T>`), ref struct constraints (no fields in classes, no
closures, no iterators), and the escape hatches in `MemoryMarshal` for
reinterpreting spans without copying.

The core argument is that parsers, serializers, and protocol codecs should
accept `ReadOnlySpan<T>` as their canonical parameter — array and string
overloads then become thin wrappers, since both implicitly convert to spans.
A worked CSV parsing example shows the span-first approach running ~4× faster
with zero allocations versus `string.Split`, eliminating Gen0 GC pressure on
hot paths. The article also covers common pitfalls (assuming slices are free
of bounds checks, treating `ReadOnlySpan<char>` as a dictionary key,
forgetting ref structs can't implement `IEnumerable<T>`) and is clear about
when span-first does *not* pay off: object graphs, low-volume call paths,
broad public library surfaces, and code that must cross `await` inline.

---
*Sources: [[raw/span-first-csharp]]*
*Last updated: 2026-08-01*
