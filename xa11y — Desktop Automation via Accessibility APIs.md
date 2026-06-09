# xa11y — Desktop Automation via Accessibility APIs

xa11y is a Rust library (with Python/JS bindings) that provides a Playwright-style API for driving desktop applications through their accessibility trees on Windows, macOS, and Linux. It's a testing tool, an automation platform, and — most interestingly — a blueprint for computer-use agents that don't burn tokens on screenshots. The library wrangles three fundamentally different accessibility APIs (Windows UIA, macOS AXUIElement, Linux AT-SPI2) behind a single `locator("button[name='7']").press()` interface, with lazily-evaluated selectors and built-in retry/waits borrowed from Playwright.

---

## Key Quotes

> "These accessibility APIs are a more robust mechanism for building computer use agents, which until recently have relied primarily on running vision models on screenshots (flaky, slow, token heavy)."

The thesis statement. This is the structural alternative to the 45x vision tax. Accessibility APIs give you structured element data and typed actions — the desktop equivalent of an API endpoint — instead of burning tokens rendering pixels and asking a vision model to interpret them.

> "No fallbacks. With the flexibility of macOS and Linux APIs, it's tempting to take a best-effort approach to matching actions or roles to canonical values. But each of these convenience fallbacks can fail for some cases and obscure what the library is actually doing."

A design philosophy that more agent frameworks should learn from. "Best effort" convenience abstractions are the source of silent failures in production. When an action fails, you want to know exactly what was attempted — not guess which of three possible fallback paths was taken.

> "Earlier iterations read an app's entire accessibility tree first then ran a simple, platform-agnostic filter on the tree. It worked great on Windows, had passable performance on macOS, but was unbearably slow on Linux."

The cross-platform performance story in microcosm: the naive approach works on one platform, degrades on another, and fails catastrophically on the third. The fix — platform-specific query evaluators that fetch only needed attributes during traversal — is the kind of unglamorous engineering that separates real tools from demos.

> "A locator describes how to find an element, but the specific element itself is re-resolved every time we take an action."

The Playwright pattern applied to desktop UIs. Elements are ephemeral — especially with immediate-mode frameworks like egui that replace the entire tree every frame. Re-resolving on every action is the only honest approach.

---

## Key Themes

#tool #pattern #concept

- **Structured access beats vision for automation** — Same insight as [[Computer Use is 45x More Expensive Than Structured APIs]], applied to desktop apps instead of web apps. Accessibility trees give you the same structural advantage that APIs give you over screenshots: typed elements, named actions, programmatic state. The cost gap is architectural, not model-dependent.
- **Cross-platform is three separate libraries wearing a trench coat** — Windows UIA, macOS AXUIElement, and Linux AT-SPI2 diverge on data model rigidity, action semantics, query performance, and even whether you can batch-read. xa11y's architecture (per-platform crates, shared core, escape hatches for irreconcilable differences) is the honest way to do it.
- **The locator pattern is universal** — Playwright's lazily-evaluated selectors with built-in retry/waits work for web UIs because the DOM is volatile. Desktop accessibility trees are even more volatile (immediate-mode frameworks replace the tree every frame). Same solution, same reason.
- **Linux is the hard case** — AT-SPI2 over D-Bus is the performance bottleneck. Multi-millisecond latency per property call means reading a full tree takes seconds. Platform-specific query optimization isn't a nice-to-have; it's the difference between usable and broken.

---

## Critical Analysis

This is the right architecture for desktop automation in the age of computer-use agents, and it's embarrassing that we're still doing vision-based screenshots when these accessibility APIs have existed for decades. The 45x cost gap [[Computer Use is 45x More Expensive Than Structured APIs]] demonstrated for web apps applies even more strongly to desktop: you're not just burning tokens on pixels, you're training vision models to reverse-engineer button boundaries from bitmaps when the OS already knows exactly where every button is, what it's called, and what actions it accepts.

The "no fallbacks" design decision is quietly radical. Most cross-platform libraries paper over differences with heuristic matching and best-effort translations that work 90% of the time and fail silently the other 10%. xa11y refuses to guess. If a macOS app uses a non-standard action name, the library surfaces the raw platform data rather than trying to map it to a canonical equivalent that might be wrong. This is harder to use but impossible to silently break — exactly the tradeoff you want in testing and automation.

The comparison to [[Webwright]] is instructive. Webwright gives coding models a terminal with Playwright and lets them write imperative scripts. xa11y gives agents a structured element tree with selector-based interaction. Both are reactions against vision-based computer use, but from opposite directions: Webwright says "let the model write code," xa11y says "give the model structured data." The xa11y approach is more efficient per-step; the Webwright approach is more flexible for open-ended exploration. The best computer-use agents will likely combine both — structured access for known UI patterns, script-based fallback for everything else.

The Linux performance story is a warning. AT-SPI2 over D-Bus with per-property round trips is architecturally slow in a way that no amount of optimization can fix — you can mitigate it with smarter traversal (as xa11y does) but you can't eliminate it. This is the same class of problem as the vision tax: the interface imposes a structural cost floor. If Linux desktop automation takes off, AT-SPI2 will need a bulk-read analog or a more efficient transport than D-Bus.

What's missing from the article is the agent integration story. The library is described as a foundation for computer-use agents, and agent-desktop.dev is mentioned as an example — but the actual agent architecture (how an LLM consumes the accessibility tree, how it decides which selectors to write, how it recovers from failures) is left as an exercise. That's the hard part, and xa11y is just the substrate. A good substrate, but still just the substrate.

---

## Related

- [[Computer Use is 45x More Expensive Than Structured APIs]] — The economic case for structured interfaces over vision. xa11y is the desktop equivalent of the API approach
- [[Webwright]] — Microsoft's browser agent that gives coding models a terminal with Playwright. Different approach (code-as-action) to the same problem (bypassing vision)
- [[Browser Use]] — The vision-based approach that xa11y and Webwright are alternatives to
- [[FlaUInspect]] — Windows UI Automation inspector for browsing UIA trees. Complements xa11y's Windows backend
- [[Maestro (UI Testing)]] — YAML-based mobile/web UI testing. xa11y brings the same locator pattern to desktop
- [[surf-cli]] — Browser automation via CLI. Same "structured access over screenshots" philosophy for web

---

*Sources: [[raw/xa11y-cross-platform-desktop-automation]]*
*Last updated: 2026-06-09*
