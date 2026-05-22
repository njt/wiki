# Gova

A declarative reactive GUI framework for building native desktop apps (macOS, Windows, Linux) from Go. Wraps Fyne internally but exposes its own SwiftUI-inspired API: typed components, call-site-identified reactive state, chainable modifiers, and native macOS integrations via cgo. Ships with a hot-reload dev server.

---

## Architecture

Gova has three layers: a declarative Go API, a reactive runtime, and an internal Fyne bridge.

**Declarative API** — Views are `*viewNode` trees built from constructor functions (`gova/view.go:6-31`). Every widget (Text, Button, TextField, Toggle, Slider, Picker, List, Grid, Form, ScrollView, Image, TabView, NavStack) returns a `*viewNode`. Layout primitives (VStack, HStack, ZStack, Scaffold) compose children into trees. Components come in two forms: functional (`Define(func(s *Scope) View)`) and struct-based (`Viewable` interface with `Body(s *Scope) View`). A `asView()` converter accepts View, Viewable, or nil at layout boundaries (`gova/component.go:64-76`).

**Reactive runtime** — The `Scope` (`gova/scope.go:12-28`) carries per-component state (`StateValue[T]`), refs (`RefValue[T]`), stores, effects, and child scopes. State identity is automatic: `runtime.Caller` produces "file:line" keys so `State(s, initial)` pins to the call site rather than execution order (`gova/state.go:176-179`). Derived signals (`Format()`, `Derived()`) are memoized by call site to prevent subscription leaks on re-render (`gova/state.go:90-99`, `gova/signal.go:152-161`). Slice helpers (Append, Prepend, RemoveWhere, UpdateWhere) exist as both reflection-based methods and generic free functions (`gova/state_slice.go`). Effects (`UseEffect`, `UseAsync`) manage side effects with context-based cancellation (`gova/effect.go`). Stores (`Provide`/`UseStore`) provide scoped dependency injection with scope-chain walking (`gova/store.go`).

**Fyne bridge (internal)** — The `ViewSpec` (`internal/fyne/spec.go`) is a flat, serializable intermediate representation. `FyneBridge.Mount()` converts ViewSpecs to Fyne CanvasObjects, wrapping them with visual modifier layers (background, stroke, opacity) in box-model order: padding inside the visible box (`internal/fyne/bridge.go:62-81`). `Reconcile()` updates mounted widgets in-place for same-kind, same-key nodes — color changes, text changes, handler swaps — and only remounts on kind/key mismatch. Visual modifiers are kept as handles (`BgRect`, `StrokeRect`, `FadeOverlay`) for in-place color updates without remounting (`internal/fyne/bridge.go:97-133`).

The rendering pipeline: component function → viewNode tree → `toSpecWithScope()` converts to ViewSpec tree → `bridge.Mount()` creates Fyne objects → state change triggers `onStateChange` callback → re-render → `bridge.Reconcile()` updates in place.

The `gova` CLI (`cmd/gova/main.go`) provides `dev` (hot reload via fsnotify watcher + build + restart), `build` (static binary to `./bin/`), and `run` (build + launch once).

## Key Techniques

**Call-site state identity** — Unlike React's hook-ordering rules, Gova uses `runtime.Caller` to derive "file:line" keys for state. Adding conditional state doesn't violate any ordering contract; identity follows the source location. An explicit `StateKey()` escape hatch exists for dynamic cases (`gova/state.go:176-185`).

**Signal memoization by call site** — `Format()` and `Derived()` cache derived signals keyed by "call_site:format_string" to prevent subscription leaks. Without this, every re-render would create a new signal with a new subscription, accumulating listeners on every frame (`gova/state.go:147-161`).

**Flat ViewSpec as protocol boundary** — The ViewSpec struct has no pointers back to viewNode or Scope. It's a pure-data contract between the declarative API and the imperative widget toolkit. The bridge never touches Gova internals; the API layer can evolve independently.

**Reconciliation with modifier handles** — When only modifier values change (background color, stroke width, opacity), the bridge updates Fyne objects in-place via handles stored on MountedNode, avoiding subtree remounts. Remounts only happen on kind change or key mismatch.

**Child scopes keyed by slot** — Each child position gets a stable "slot ID" (explicit `Key()` or positional index). Scopes cache child scopes by this ID, so sibling components of the same type hold independent state — equivalent to React's key system operating at the scope level (`gova/render.go:367-375`).

**Semantic colors as sentinels** — Theme colors (`Primary`, `Accent`, `Background`, etc.) are `themeColor` structs implementing `color.Color` with zero RGBA. They pass through `any`-typed modifier fields and are resolved at render time via `resolveColor()`. Lets you write `Text("foo").Color(Accent)` without importing Fyne types (`gova/style.go:69-71,86-111`).

**Unicode strikethrough** — Instead of renderer-specific strikethrough, inserts U+0336 (combining long stroke overlay) after each rune. Works in any font, no renderer changes (`gova/render.go:322-331`).

## Design Decisions

**Fyne stays internal** — The user never imports Fyne. All rendering is behind `internal/`. This means the backend could be swapped without breaking the public API. The cost: Gova reimplements widget wrappers, theme mapping, and layout logic that Fyne already provides.

**Explicit Scope over implicit hooks** — React uses call-order tracking and a hidden scheduler; SwiftUI uses property wrappers. Gova makes `Scope` an explicit parameter. More typing per component, but zero surprises about re-render triggers. The user can see where scope flows.

**Rebuild-and-reconcile, no virtual DOM diff** — State change rebuilds the entire viewNode tree from the component function. Reconciliation compares by positional index, not structural hashing. Simpler than React but less efficient for large trees where insertions shift all subsequent indices.

**Pre-1.0 with honest disclosure** — README explicitly states API instability and recommends pinning tags. Rare directness in the Go ecosystem.

**Binary size trade-off** — 32MB minimum (Fyne's cost: GLFW, font rendering, SVG renderer, platform backends). Larger than pure-Go TUIs (~5MB) but smaller than Electron (~150MB+). Justified for native widgets.

**Go 1.26 minimum** — Requires the bleeding-edge Go release, limiting adoption in conservative environments.

## Comparison Notes

Unlike **Fyne** directly — Gova provides declarative composition with automatic reactive re-render instead of imperative widget construction with manual state synchronization. Fyne says `widget.NewLabel(x); label.SetText(y)`; Gova says `Text(state.Format(...))` and handles the update automatically.

Unlike **Wails** — Gova produces native widgets via Fyne, not a webview. No JavaScript, HTML, or CSS. Larger binary but genuinely native feel.

Unlike **vibes-cli** (`[[vibes-cli]]`) — vibes-cli targets non-coders with AI-generated single-file HTML apps. Gova targets Go developers building real desktop applications with a framework. vibes-cli is a code generator; Gova is a framework.

Like **SwiftUI** in design — modifier chains (`.Padding()`, `.Color()`, `.Background()`, `.Frame()`, `.Shadow()`, `.OnTap()`), the `Viewable` protocol, and scoped state are direct SwiftUI parallels. The key difference: Gova uses call-site identity instead of property wrappers for state.

Like **React** in architecture — functional components, hooks, context/providers, key-based identity, reconciliation. Unlike React: no JSX (Go function calls), no virtual DOM (viewNode tree), no hidden scheduler (synchronous callbacks).

---

*Sources: [[raw/gova]]*
*Last updated: 2026-05-22*
