---
url: https://crowecawcaw.github.io/general/2026/05/30/accessibility-for-computer-use.html
title: "Cross-platform desktop automation through accessibility APIs"
author: crowecawcaw
date_fetched: 2026-06-09
date_published: 2026-05-30
topics:
  - agent-architecture
---

# Cross-platform desktop automation through accessibility APIs

By crowecawcaw, 2026-05-30.

[xa11y](https://xa11y.dev) provides a Playwright-style API for driving desktop applications through their accessibility tree on Windows, macOS, and Linux. It's a Rust library with additional Python and JavaScript bindings.

The library provides a foundation for desktop testing, automation, and accessibility software. Additionally, these accessibility APIs are a more robust mechanism for building computer use agents, which until recently have relied primarily on running vision models on screenshots (flaky, slow, token heavy).

## API Example

```rust
use xa11y::*;
use std::time::Duration;

let calc = App::by_name("Calculator", Duration::from_secs(5))?;
calc.locator("button[name='7']").press()?;
calc.locator("button[name='+']").press()?;
calc.locator("button[name='3']").press()?;
calc.locator("button[name='=']").press()?;

let display = calc.locator("static_text").first().element()?;
assert_eq!(display.data().value.as_deref(), Some("10"));
```

## Architecture

Under the hood, the Cargo workspace splits along platform lines: `xa11y-core` holds the shared types and selector engine; `xa11y-windows`, `xa11y-macos`, and `xa11y-linux` each wrap one platform's FFI (COM via the `windows` crate, Core Foundation via `core-foundation`, and D-Bus via `zbus`); and the top-level `xa11y` crate conditionally compiles the right backend per target. The Python and Node packages are separate crates layered on top via `pyo3` and `napi-rs`. Isolating each FFI surface in its own crate keeps `unsafe` and platform `cfg`s out of `xa11y-core` and lets each backend evolve against its own native idioms.

## Cross-platform differences

### Windows (UIA)
- Most structured data model: fixed role enums, standardized "control patterns" for actions
- Can prefetch entire subtrees in a single call (`FindAllBuildCache` + `CacheRequest`)
- Fastest and most programmatically interpretable

### macOS (AXUIElement)
- Flexible: roles and subroles are string-based with conventions but no enforcement
- Actions split between property updates (e.g., text input) and action invocation (e.g., button press)
- No bulk tree-read API — must read attributes per-element, but supports batched `AXUIElementCopyMultipleAttributeValues`

### Linux (AT-SPI2)
- Middle ground: enum-based roles with custom registration, untyped string-based actions
- Performance bottleneck: individual API calls per property, traversing the system-wide D-Bus interface with multi-millisecond latency
- Reading a full a11y tree for a real app can take multiple seconds

## Design Decisions

1. **Honest abstractions with escape hatches.** Common patterns (reading elements, text input, button clicks) have clean cross-platform abstractions. Irreconcilable platform differences (e.g., macOS subroles) are preserved as raw platform data accessible to library consumers.

2. **Build on existing design.** Windows UIA used as the starting point because it has the most rigid types.

3. **No fallbacks.** No best-effort matching of action names to canonical values — each convenience fallback can fail for some cases and obscure what the library is actually doing.

4. **Platform-specific query evaluation.** Earlier iterations read the entire tree then filtered; worked on Windows, passable on macOS, unbearably slow on Linux. Current approach fetches only needed attributes during tree traversal. Example: for `button[name='7']`, only reads name and role for each element.

## Thrashing trees

Desktop UIs can replace elements on every layout shift or frame (especially immediate-mode frameworks like egui). Each platform has stable element references but they're opt-in/best-effort, not reliable. This is critical for testing/automation where "click Ok" is really two operations (find + click) and the element may be replaced between them.

UIs also have real, observable delays: opening a dropdown, waiting for menu render, clicking an item. Longer sequences become littered with polling loops and retries.

**Solution: lazily evaluated locators with waiting.** Borrowed from Playwright. A locator describes how to find an element, re-resolved on every action/read. Resolution retries for a short period to allow the UI to settle.

## Links

- Rust: `cargo add xa11y`
- Python: `pip install xa11y`
- Node: `npm install @crowecawcaw/xa11y`
- Docs: https://xa11y.dev
- Related: https://agent-desktop.dev
