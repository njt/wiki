# Wails

Wails is a Go framework for building cross-platform desktop applications whose UI is written with web technologies — the Go answer to Electron and Tauri. Version 3.0.0 is its stable desktop release for Windows, macOS and Linux, with Android/iOS explicitly experimental. The codebase is worth reading not for what it does (many webview wrappers exist) but for how carefully it avoids the classic sins of the category: open local ports, cgo everywhere, and reflection-heavy IPC hot paths.

---

## Architecture

The repo is a monorepo with v2 and v3 side by side; v3 (`/v3`) is the live product. The heart is `v3/pkg/application` (~92,000 lines of Go), which contains:

- **Per-platform application/window drivers.** `application_windows.go`, `application_linux_gtk3.go`/`gtk4`, and two macOS paths: `application_darwin.go` (cgo) and `application_darwin_purego.go`, which binds Cocoa through `ebitengine/purego`'s ObjC bridge behind a `purego` build tag. Same trick on Linux: `application_linux_purego_gtk3_generated.go` holds generated purego bindings for GTK. A blank import of `internal/buildguard` exists purely to produce one clear error when no native backend build tag was selected — a nice fail-fast touch.
- **A message-processor IPC core.** `messageprocessor.go` defines `RuntimeRequest` — an integer `Object` (Call=0, Events=3, Dialog=5, Window=6, …) plus integer `Method` and args — dispatched across thirteen `messageprocessor_*.go` files. The JS side (`internal/runtime/desktop/@wailsio/runtime/src/runtime.ts` and friends) mirrors the same numbering, so every runtime call is a tiny two-number dispatch, not a stringly-typed RPC.
- **Bindings via reflection with stable IDs.** `bindings.go` reflects bound service structs into `BoundMethod` records (inputs/outputs as `*Parameter`, cached variadic flags, struct fields reordered to minimise padding — the comments brag about saving 8 bytes). Method IDs are pre-registered via `RegisterBindingMethodID` so calls resolve by `uint32` even when binary names are obfuscated. Generated TypeScript comes from `internal/generator`, a static analyser built on `golang.org/x/tools/go/packages` that parses the project source rather than introspecting a running binary.
- **An asset server without a listener.** `internal/assetserver` serves the frontend through the webview's custom URL scheme (with per-window headers `x-wails-window-id`/`x-wails-window-name`), dev-mode and production variants behind the same handler chain, plus a content-type sniffer and ring-queue internals.
- **A CLI.** `cmd/wails3` (clir-based): init, build, dev, doctor, `mcp`, and a `run` that falls back to `go run .` when no `wails.yaml` exists.

## Key techniques

- **Streams: WebSockets without a port** (`pkg/application/stream.go`). A WebSocket can't be spoken over a custom URL scheme, and the only real alternative is a listener any local process could reach. So Streams implement the WebSocket programming model over the already origin-bound asset server: Go→JS via a held long-poll, JS→Go via POST. The file's opening comment is a mini-incident report: event emits used to run `evaluateJavaScript` inline from the main thread and invert 4.4% of events; splicing payloads into eval source blew host memory past platform-specific knees (11.6 GB macOS, 6.2 GB WebKitGTK at 100 events/sec of 1 MB). The fix: a mutex-append plus single drainer for ordering, bounded buffers dropped when a window dies, and connection lifetime = handler goroutine lifetime.
- **Opt-in MCP for agents** (`pkg/application/mcp_enabled.go`, build tag `mcp`). A compiled-in MCP HTTP server (default 127.0.0.1:9099, bearer token required for non-loopback binds) that exposes tools for evaluating JavaScript in the running app. A desktop app framework treating an AI agent as a first-class client of the app's runtime is the most 2026 thing in the codebase — QA and automation agents can drive the real UI without any Electron-style remote-debugging port.
- **Pluggable transport.** `Options.Transport` lets you replace the default HTTP IPC entirely; if the transport implements `WailsEventListener` it gets events wired automatically, otherwise an `EventIPCTransport` fallback kicks in.
- **Structured shutdown as a pattern.** The event payload store, stream manager, and transport each register `OnShutdown` hooks at construction; cleanup policy is declared where the resource is created, not scattered across window-close handlers.

## Design decisions

Wails has consistently chosen *no open ports and no cgo where avoidable* over the easy path — the purego drivers cost them generated-binding maintenance (there are literally `*_generated.go` GTK files) to gain cgo-free builds and cross-compilation. The bindings layer is optimised for the hot path (integer dispatch, pre-registered IDs, cached reflect results, struct-padding tuning) while pushing the expensive static analysis to build time in the generator. The price: a large, complex build-tag matrix (`purego`, `server`, `mcp`, `android`, `ios`, `dev` variants of nearly every driver) that makes the codebase hard to hold in your head — 92k lines in one package is also a smell, however disciplined the internals. Mobile support is quarantined from the desktop compatibility contract, an honest acknowledgment that webview-on-mobile is a different product.

## Comparison notes

Compared with [[Zed]], which rejected webviews entirely for a GPU-native Rust UI, Wails makes the opposite bet: the web frontend is the productivity surface and Go is the system language — Zed optimises for rendering control, Wails for frontend ecosystem reuse. Against Electron the whole design is a rebuttal: no bundled Chromium, no Node main process, no open debug port. And the built-in MCP server is the same move as [[Ghidra MCP Server (bethington)]] — a mature desktop tool deciding that agent drivability is a shipped feature, not a hack — but done at the framework level, so every app built with Wails gets it for free. The no-open-port discipline in `stream.go` reads like the same lesson the sandbox papers in this wiki keep drawing ([[Figma's Nimbus — Sandboxes as a Platform Contract]]): anything bound to localhost is reachable by everything else on the machine.

---
*Sources: [[raw/wails]], [[summary/wails]]*
*Last updated: 2026-11-06*
