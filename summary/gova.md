---
url: https://github.com/NV404/gova
title: "Gova: Declarative GUI Framework for Go"
author: NV404
date_fetched: 2026-05-22
date_published: 2025
---

# Gova — Declarative Native GUI Framework for Go

Gova is a declarative reactive GUI framework for building native desktop apps (macOS, Windows, Linux) from Go. It wraps Fyne internally but exposes its own SwiftUI-inspired API: typed components, reactive state identified by call site, a chainable modifier system, and platform-native integrations via cgo. The project ships its own `gova` CLI with hot-reload dev server.

Repository: https://github.com/NV404/gova
License: MIT
Language: Go 1.26+
Lines: ~9,700 Go (core lib + CLI + devserver + internal Fyne bridge)
Dependencies: Fyne v2.7.3, fsnotify v1.9.0

## Architecture

Gova is a three-layer system:

**Layer 1: Declarative API (public)**
- `view.go` — The `View` interface and `viewNode` tree. A `viewNode` is a union type (enum kind + typed data fields). Every widget constructor returns a `*viewNode` with chainable modifier methods.
- `layout.go` — VStack, HStack, ZStack, Group, Spacer, Scaffold (border layout with Top/Bottom/Leading/Trailing).
- `component.go` — `Define(fn func(*Scope) View)` for functional components and `Viewable` interface for struct-based components. `asView()` converts View/Viewable/nil into View.
- `modifier.go` — SwiftUI-style modifier chain: Padding, Font, Color, Background, Opacity, Frame, Grow, Shadow, Stroke, CornerRadius, OnTap, AccessibilityLabel/Role. Plus semantic color constants (Primary, Secondary, Accent, etc.) and spacing tokens (SpaceXS/SM/MD/LG/XL).
- Style: `style.go` — Theme system with semantic colors, dark/light variants, accent, customizable per-theme/size-name. Hex() string parser. FadeColor/MixColor utilities.

**Layer 2: Reactive Runtime**
- `scope.go` — The `Scope` type: carries states, refs, stores, and child scopes. `callerKey()` uses `runtime.Caller` to derive file:line keys for automatic state identity. `childScopeFor(id)` creates/caches per-slot child scopes.
- `state.go` — `StateValue[T]` with Get/Set/SetSilent/Update. Internal subscriber system for signals. Signal cache memoized by call site + format string. `Format()` derives `Signal[string]`, `Len()` gets collection length via reflection. `RefValue[T]` for non-reactive mutable values. `State()`, `StateKey()`, `Ref()` factory functions.
- `signal.go` — `Signal[T]` interface (Value + subscribe). `formatSignal` for fmt.Sprintf derivations, `derivedSignal` for arbitrary transforms. Both memoized by call site to prevent subscription leaks on re-render.
- `state_slice.go` — Slice mutation helpers: Append, Prepend, RemoveWhere, UpdateWhere. Both method form (reflection-based, any element type) and free-function form (generics, compile-time safe).
- `effect.go` — `UseEffect` (runs once per component mount, with cleanup) and `UseAsync` (runs async function, returns data/error/loading, re-renders on completion).
- `store.go` — `StoreKey[T]` with `Provide`/`UseStore` for scoped dependency injection. Walks the scope chain to find nearest ancestor provider.
- `persist.go` — `PersistedState[T]` saves state to JSON on disk on every change. Survives `gova dev` reloads via GOVA_DEV_STATE env var. Uses atomic write (tmp + rename). Defensive against re-subscription on repeated renders.

**Layer 3: Fyne Bridge (internal)**
- `internal/fyne/spec.go` — `ViewSpec`: a flat, serializable intermediate representation. 21 ViewKind constants. All widget data (text, buttons, sliders, lists, forms, nav, etc.), all modifier data (padding, colors, shadows, opacity), all layout data (spacing, alignment, grow, min size). Has Cleanup() for unsubscribe chains.
- `internal/fyne/bridge.go` — `FyneBridge`: Mount(spec) creates Fyne CanvasObjects from ViewSpecs, wrapping with padding/visual modifiers/min-size layers. Reconcile(mounted, newSpec) updates existing widgets in-place when possible (color changes, text changes, handler swaps), falling back to remount on kind/key mismatch. Container reconciliation handles add/remove/reorder by index. Visual modifiers (BgRect, StrokeRect, FadeOverlay) have handles for in-place color updates without remounting.
- `internal/fyne/coloredtext.go` — Custom canvas object for colored text at specific sizes.
- `internal/fyne/tappable.go` — Makes any CanvasObject respond to tap events via hit testing.
- `internal/fyne/overlay.go` — ShowAlert (native NSAlert on macOS, Fyne fallback) and ShowSheet (modal overlay).
- `internal/fyne/theme.go` — Maps Gova theme config onto Fyne's theme interface. 30+ ThemeColorName constants, 15 ThemeSizeName constants.
- `internal/fyne/fonts/` — Embedded Inter font family (Regular, Medium, SemiBold, Bold).

### Rendering pipeline

1. Component function is called with Scope → returns a View (tree of *viewNode)
2. `toSpecWithScope(node, scope)` walks the viewNode tree, resolving components, creating child scopes for composed components, and building a flat ViewSpec tree
3. `bridge.Mount(spec)` converts ViewSpec → Fyne CanvasObjects, wrapping with padding, backgrounds, strokes, opacity, tap handlers
4. On state change: `onStateChange` callback triggers re-render of the owning component → new viewNode tree → `toSpecWithScope` → `bridge.Reconcile(mounted, newSpec)` updates widgets in-place

Key files:
- `gova.go` — `Run()`, `RunWithConfig()` entry points. Boots Fyne app, creates scope, mounts initial view, runs event loop.
- `render.go` — `toSpecWithScope()`, `childSpecsWithScope()`, `renderSlot()`, `navStackSpec()`. The conversion from viewNode tree to ViewSpec tree.
- `dialog.go` — Native dialog wrappers (NSAlert/NSOpenPanel/NSSavePanel on macOS).
- `dock.go` — Dock/taskbar API: SetBadge, SetProgress, Bounce, SetMenu. cgo implementation on macOS.
- `nav.go` — NavStack with Push/Pop/PopToRoot/Replace. NavLink declarative component. NavTitle/NavToolbar modifiers.

### Dev server

`internal/devserver/` — Hot reload:
- Watcher (fsnotify) watches .go files, debounces, triggers rebuild
- Builder compiles the Go package
- Supervisor manages the child process (start, kill on rebuild, signal forwarding)
- Server orchestrates: watch → build → restart loop
- `cmd/gova/` — CLI with `dev`, `build`, `run` commands

### Examples

7 runnable examples: counter (minimal), todo (state+lists+forms), fancytodo (derived state), notes (nav+multi-view+stores), themed (dark/light), components (Viewable composition), dialogs (native dialogs+dock).

### Tests

~25 test functions across `gova_test.go`, `harness_test.go`, `persist_test.go`, `overlay_test.go`, `dock_test.go`, plus internal tests for bridge, devserver components. Tests use explicit string keys (not runtime.Caller) for deterministic state identity. The dock has a `recordingDock` test double.

## Key Techniques

**Call-site state identity via runtime.Caller**: Instead of React's hook-ordering rules, Gova uses `runtime.Caller(skip)` to generate "file:line" keys. `State(s, initial)` and `Ref(s, initial)` pin identity to the source location of the call, not the order of calls. This means you can add conditional state without worrying about hook-ordering violations. `StateKey(s, key, initial)` provides an explicit-key escape hatch for dynamic/conditional state.

**Signal memoization by call site**: `Format()` and `Derived()` cache derived signals keyed by call site + format string / transform name. Without this, every re-render would create a new signal (and a new subscription), leaking subscribers. The cache lives on the source `StateValue` itself, so it's scoped to the component's lifetime.

**Flat ViewSpec as protocol boundary**: The `ViewSpec` struct in `internal/fyne/spec.go` is a flat data structure — no pointers back to the view tree, no Fyne objects. It's a serialization boundary. Every widget kind packs its data into ViewSpec fields (all nullable/primitives). This means the bridge never touches `viewNode` or `Scope` directly — it only sees the spec. The API layer can evolve independently of the rendering layer.

**Reconciliation with modifier handles**: When a component re-renders and only modifier values change (background color, stroke width, opacity), the bridge updates the affected Fyne objects in-place via handles (`BgRect`, `StrokeRect`, `FadeOverlay`) stored on `MountedNode`. This avoids remounting the subtree. Only kind changes or key mismatches trigger remounts.

**Child scopes keyed by slot**: Each child position in a parent gets a stable "slot ID" — either from an explicit `Key()` on the node, or from its positional index. The scope creates/caches child scopes by this ID, so sibling components of the same type hold independent state. This mirrors React's key-based reconciliation but operates at the scope level.

**Semantic colors as sentinel values**: Theme colors (Primary, Secondary, Background, etc.) are `themeColor` structs that implement `color.Color` (with zero RGBA). They pass through the `any`-typed modifier fields and are detected by `resolveColor()` at render time, which resolves them against the active theme. This lets you write `Text("foo").Color(Accent)` without importing Fyne's color types.

**Unicode strikethrough**: Instead of requiring renderer-specific strikethrough support, Gova inserts Unicode combining long stroke overlay (U+0336) after each rune. Works in any font, no renderer changes needed. Clever hack with a clear limitation: the strikethrough appearance depends on font rendering quality.

## Design Decisions

**Fyne stays internal — the public API is Gova's alone**: Unlike Wails or go-app which expose web views, or Fyne itself which exposes Fyne widgets directly, Gova wraps Fyne entirely within `internal/fyne/`. The user never imports Fyne. This means the rendering backend could theoretically be swapped out without breaking the public API. The cost is that Gova reimplements widget wrappers, theme mapping, and layout logic that Fyne already provides.

**Explicit Scope vs implicit hooks**: React uses call-order tracking and a hidden scheduler; SwiftUI uses property wrappers and a hidden diffing engine. Gova makes the `Scope` an explicit parameter to every component function. State is `State(s, initial)`, effects are `UseEffect(s, fn)`, stores are `UseStore(s, key)`. The user can see where scope flows. The trade-off is more typing per component, but there are no surprises about when or why something re-renders.

**No virtual DOM diff — rebuild and reconcile by index**: Every state change rebuilds the entire viewNode tree from the component function. The renderer compares the new ViewSpec against the mounted tree by positional index (not structural hashing or key diffing). Same-kind, same-index nodes get reconciled in-place; different-kind or key-mismatched nodes get remounted. This is simpler than React's reconciliation but less efficient for large trees where a single insertion shifts all subsequent indices.

**Pre-1.0 with API instability disclosure**: The README explicitly states "The API will shift before v1.0.0. Pin a tag in production." This is rare honesty in the Go ecosystem, where many projects ship v0.x for years without acknowledging API instability.

**Binary size is the cost of a native toolkit**: A minimal counter app is 32MB (23MB stripped). This is the Fyne dependency — Fyne bundles GLFW, font rendering, an SVG renderer, and platform backends. Compare to Electron apps which are often 150MB+, but also compare to pure-Go TUI frameworks which are ~5MB. The trade-off is justified for native GUI features, but worth noting for distribution-conscious projects.

**Go 1.26 as minimum**: Requires the bleeding-edge Go release. This limits adoption in enterprises and CI environments that standardize on older Go versions. The choice likely reflects reliance on new generics features or toolchain improvements.

## Comparison Notes

Unlike **Fyne** itself — Gova is a higher-level framework on top of Fyne. Where Fyne gives you imperative widget construction (`widget.NewLabel("hello")`, `widget.NewButton("OK", handler)`), Gova gives you declarative composition (`VStack(Text("hello"), Button("OK", handler))`) with automatic re-render on state change. Fyne requires manual state-to-UI synchronization; Gova does it reactively.

Unlike **Wails** (Go + web frontend) — Gova produces native widgets, not a webview. No JavaScript runtime, no HTML, no CSS. The binary is larger than a Wails app but the UI feels native rather than like a website in a frame.

Unlike **vibes-cli** (`vibes-cli.md`) — vibes-cli targets non-coders with single-file HTML apps generated by Claude Code. Gova targets Go developers building real desktop applications. vibes-cli is a code generator; Gova is a framework.

Unlike **Zeroclaw** (`Zeroclaw.md`) — Zeroclaw is a Rust-native agent runtime with OS-level sandboxing. Gova is a Go GUI framework. Both are "native" but in different domains — Zeroclaw for agent execution, Gova for user-facing applications.

Like **SwiftUI** — the modifier chain pattern (`.Padding()`, `.Font()`, `.Color()`, `.Background()`, `.Frame()`, `.Shadow()`, `.CornerRadius()`, `.OnTap()`) is directly inspired by SwiftUI's view modifiers. The `Viewable` interface with `Body(s *Scope) View` mirrors SwiftUI's `View` protocol. The `@State` equivalent is `State(s, initial)` with call-site identity instead of property wrappers. The explicit Scope replaces SwiftUI's hidden environment.

Like **React** — functional components (`Define`), hooks (`State`, `Ref`, `UseEffect`, `UseAsync`), context/providers (`Provide`/`UseStore`), key-based identity (`Key()`), and reconciliation. Unlike React, there's no JSX (everything is Go function calls), no virtual DOM (viewNode tree is the intermediate), and no hidden scheduler (re-renders are synchronous callbacks).

Like **Jetpack Compose** — the composable function pattern and modifier chains. Unlike Compose, there's no compiler plugin — `Define` is a plain Go function, not an annotation.

Tags: #tool #project #go #gui #desktop #reactive #declarative-ui
