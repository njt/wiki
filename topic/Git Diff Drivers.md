# Git Diff Drivers

How to wire an external tool into `git diff` so complex file formats get human-readable diffs instead of raw JSON churn. Jamie Tanna's guide fills a documentation gap: the interface exists but was never well-documented.

---

## The interface

Git passes **7 arguments** to your external tool when it invokes it as a diff driver:

1. Filename in the repo
2. Path to the "before" file (or `/dev/null` if new)
3. SHA-1 hash of the "before" file (`.` if new)
4. Octal mode of the "before" file (`.` if new)
5. Path to the "after" file (or `/dev/null` if deleted)
6. SHA-1 hash of the "after" file (`.` if deleted)
7. Octal mode of the "after" file (`.` if deleted)

This is more complex than the two-argument `tool [before] [after]` interface many tools expect, but a thin shell wrapper bridges the gap.

## Key quotes

> "There seemed to be a lack of documentation around how to do it."

Tanna started this post in November 2024 and finally published it April 2026. The git-diffs man page technically covers this, but discoverability is near zero.

> "In a lot of cases, using `textconv` is likely sufficient."

For simple format conversion (e.g., rendering a binary format to text before diffing), `textconv` is the lighter-weight option. The full diff driver is for when you need richer output — structured changelogs, semantic diffs, or domain-specific comparisons.

## Worked example: OpenAPI specs

The `oasdiff` tool compares OpenAPI specs and produces a human-readable changelog. Tanna's wrapper script checks for `/dev/null` to handle file creation/deletion, then delegates to `oasdiff changelog "$2" "$5" --color always`.

The script doesn't handle permission changes. Tanna notes the SHA-1 hashes could be used to cache diffs — an optimization worth considering for large repos.

## Key themes

- `#tool` — git's diff driver is a well-designed extension point hiding in plain sight
- `#concept` — the `/dev/null` sentinel pattern for signaling resource lifecycle events
- `#pattern` — thin shell wrappers as the universal adapter between git's interface and opinionated tools
- `#person` — Jamie Tanna, prolific #blogumentation author documenting tools and workflows

## Critical analysis

This is a classic #blogumentation piece: not novel, but fills a real gap. The git diff driver interface has existed for years; what was missing was someone writing down the 7-argument contract in plain English with a worked example. Tanna's post does exactly that.

The deeper pattern here is that git is full of extension points that are *documented* but not *explained*. The man page lists arguments; it doesn't tell you *why* you'd use this, *when* `textconv` is insufficient, or *how* to handle the lifecycle edge cases. This is the documentation genre that matters most for tools with long histories — not reference, but translation.

The caching suggestion (using SHA-1 hashes to skip re-diffing unchanged content) is underdeveloped. It's a one-sentence aside that could be its own post. For repos with many structured files, this is the difference between a diff driver that's pleasant to use and one that's a CPU hog.

Tools like [[Hunk]] sit above this plumbing layer: where git diff drivers produce the raw diff, Hunk renders it as a navigable continuous stream with agent annotations overlaid inline. The diff driver defines *what* the diff contains; Hunk defines *how* a human reads it. The emerging terminal-native review stack stacks these layers: git diff drivers (plumbing) → Hunk (rendering + agent annotations) → [[Local Review]] / [[Subspace]] (human feedback back to agents).

---

*Sources: [[summary/jvt-me-git-diff-driver]]*
*Last updated: 2026-05-15*
