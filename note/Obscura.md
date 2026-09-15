# Obscura

Obscura is a headless browser engine built from scratch in Rust for web scraping and AI-agent automation — not a Chromium fork, not a wrapper around one. It embeds real V8 (via `deno_core`) over a real DOM (`html5ever`), speaks the Chrome DevTools Protocol so Puppeteer and Playwright connect to it unchanged, and implements its own CSS layout and paint pipeline so screenshots, screencasts, and PDFs render without ever launching a browser binary. Around that core it layers the things agents actually need: an MCP server with 38 browser tools, an SSRF guard that is on by default, and an optional stealth build that makes the engine's TLS and JS surfaces present a coherent Chrome fingerprint. The pitch is the whole point: a browser redesigned as infrastructure for machines that browse at fleet scale, which is why Cloudflare's Kitesurf prototype reportedly began as a port of it.

---

## Architecture

A Cargo workspace of nine crates, one per layer, with cross-crate calls flowing downward only (`docs/Architecture-overview.md`):

```
obscura-cli       CLI: fetch / scrape / serve / mcp (3.9K lines)
obscura-cdp       CDP WebSocket server, dispatch, domain handlers (19.1K)
obscura-browser   Page type, navigation, lifecycle (10.8K; page.rs alone is 9K)
obscura-js        V8 runtime via deno_core + bootstrap.js shim (30K Rust + 16K JS)
obscura-dom       DOM tree, shadow roots, selectors (5.3K)
obscura-net       HTTP client, stealth client, cookie jar, blocklist, robots (5.7K)
obscura-render    CSS cascade, retained layout, shaping, CPU paint (72.9K)
obscura-mcp       MCP server over stdio or HTTP (3.2K)
obscura           Embeddable Rust library (Browser, Page, Element) (2.2K)
```

Roughly 187K lines of Rust including ~33K of vendored `taffy` and `cosmic-text` (patched in-tree, see below).

**Request flow.** A Puppeteer `page.goto` arrives as a WebSocket frame in `obscura-cdp/server.rs`, is routed by session id through `dispatch.rs`, and lands in `obscura-cdp/domains/page.rs` → `obscura-browser/page.rs::navigate_with_wait`, which fans out to `obscura-net/client.rs` (HTTP), `obscura-dom/tree.rs` (parse), and `obscura-js/runtime.rs` (inline scripts via the `bootstrap.js` + `ops.rs` bridge). Lifecycle events (`init → commit → domcontentloaded → load → networkidle2 → networkidle0`) are emitted back over the same socket, and `waitUntil` semantics resolve client-side in Puppeteer/Playwright on the matching `Page.lifecycleEvent`.

**The JS boundary is deliberately narrow.** Everything a page script can do goes through one stringly-typed op: `Deno.core.ops.op_dom('insert_before', parentNid, refNid, newNid)`. `obscura-js/js/bootstrap.js` (16K lines) provides `document`, `window`, `navigator`, observers, `fetch`, IndexedDB; `ops.rs` (7K lines) registers the Rust side effects. Adding a Web API is a documented three-step: JS shim, Rust op, register in `build_extension()`. Classic Web Workers are emulated *inside* the page runtime — one source execution, retained handlers, pre-load message queueing — not separate isolates or threads.

**Concurrency model.** After issue #430 the CDP server runs thread-per-connection, each connection owning its own V8 isolate; all pages within a connection share it, serialized behind a `tokio::sync::Mutex` (`v8_lock`). Long operations (navigate, eval) are spawned onto the tokio `LocalSet` because V8 is `!Send`, which is why `Target.createTarget` returns immediately under concurrent clients.

**Parallel scraping is process-level.** `obscura scrape --concurrency 25` spawns 25 copies of a separate `obscura-worker` binary; each reads NDJSON commands on stdin (`navigate`, `evaluate`, `title`, `dump_html`, `dump_text`, `shutdown`) and answers in NDJSON on stdout. Proxy/stealth/robots settings propagate by environment (`OBSCURA_PROXY`, `OBSCURA_STEALTH`, `OBSCURA_OBEY_ROBOTS`) — OS-scheduled isolation instead of in-process pools, at the cost of one V8 compile-warm isolate per worker.

## Key techniques

- **Watchdogs that can actually preempt V8.** `tokio::time::timeout` cannot interrupt synchronous JS, so `runtime.rs` arms a real termination watchdog (`arm_watchdog` / `run_event_loop_bounded`) that fires `IsolateHandle::terminate` from another thread when a budget overruns. `cdp_watchdog.rs` shares one long-lived watchdog thread across all connections — `arm`/`disarm` are a mutex + condvar in the low microseconds versus ~240 µs for thread-per-command — tracking armed slots keyed by a monotonic generation so one connection's arm never clobbers another's (tunable via `OBSCURA_CDP_COMMAND_TIMEOUT_MS`).
- **The anti-panic protocol.** `op_dom` wraps its body in `std::panic::catch_unwind` so a malformed selector degrades to `null` instead of unwinding through deno_core into V8's FFI frame, where `V8_Fatal` aborts the whole process. This is why the release profile explicitly sets `panic = "unwind"` — the `Cargo.toml` comment warns that `panic="abort"` would turn every catchable op panic into a hard crash. `obscura-dom/tree.rs` also rejects cyclic reparenting so tree walks cannot loop forever.
- **SSRF defense at two layers, shared across transports.** `obscura-net/client.rs` keeps one deny-set (loopback, RFC1918, link-local including the 169.254.169.254 metadata endpoint, IANA special-purpose ranges). `validate_url` checks the host *string*; `SsrfGuardResolver` is implemented twice — once as a `reqwest` DNS resolver, once as a `wreq` resolver — so *any* resolved address is checked at dial time, closing the DNS-rebinding hole. The `wreq_client.rs` comment is refreshingly honest: without the stealth-side resolver, `--stealth` was "a downgrade in protection rather than only a change of fingerprint."
- **Consistent, not randomized, at the transport layer.** The stealth client (`wreq`, `Profile::Chrome145`, `Platform::Windows`) presents a real Chrome's TLS ClientHello, ALPN, and cipher order — the *same* one every time — and the constants (`STEALTH_USER_AGENT`, `STEALTH_NAVIGATOR_PLATFORM = "Win32"`) exist so `navigator` reports an identity that matches the wire fingerprint; the code comments note a site cross-checks the mismatch as a bot signal. Randomization is reserved for the JS-visible surface: per-session GPU, screen, canvas, audio, and battery values, `event.isTrusted = true`, `Function.prototype.toString()` returning `[native code]`, `navigator.webdriver = undefined`.
- **Retained layout, one geometry source.** `obscura-render` keeps layout alive between captures and invalidates it on DOM, style, viewport, scroll, animation, font, and resource changes. The same geometry drives `getBoundingClientRect`, `elementFromPoint`, and IntersectionObserver *and* the paint step — there is no separate measurement model to drift from the screenshot model. Taffy provides flex/grid; Obscura layers browser formatting behavior, text shaping, and intrinsic replaced-element sizing on top. The two vendored patches are telling: a local Taffy grid shrink-to-fit fix kept out of upstream, and cosmic-text patched so canonical variable-font coordinates survive from shaping into rasterization.
- **Feature-flagged binaries, not feature-gated features.** Release archives come in four variants (render, render+stealth, no-render, no-render-stealth) sharing one codebase; `obscura-mcp` literally refuses to advertise `browser_screenshot`/`browser_pdf` when the render feature is absent. Stealth builds pull BoringSSL (CMake/Clang); plain builds stay on rustls with no extra toolchain.
- **A repo maintained *by* agents.** `AGENTS.md` encodes non-obvious operating doctrine: build with `-p obscura-cli` scoped targets because the V8 compile dominates, test with `cargo nextest` only (the single-isolate-per-process design makes `cargo test` fail), keep the obstacle course at 33/33, and never bulk-run `cargo fmt` because the tree is not rustfmt-clean. A `skills/obscura/SKILL.md` ships the same knowledge to any agent harness, and `render-repros/` holds one HTML fixture per rendering bug.

## Design decisions

- **The engine is the bet.** 73K lines of render code is the price of "no Chromium": full control over memory (30 MB vs 200+), startup (instant vs ~2 s), and binary size (~70 MiB vs 300+). The README is candid about the cost — long-tail CSS, some Web APIs, media playback, compositor effects, and platform font rasterization may differ from Chromium. Screenshots can therefore *disagree* with Chrome on the long tail, which matters when you're capturing pages for pixel comparison.
- **One isolate per connection, not per page.** Simpler memory and scheduling than Chrome's process-per-tab; the trade is a serialized JS domain within a connection, mitigated by watchdogs and LocalSet spawning rather than by parallelism. Cross-connection isolation is bought with OS threads and separate workers instead.
- **Stringly-typed op bridge.** `op_dom(cmd, arg1, arg2) -> String` is inelegant but keeps the JS/Rust boundary at exactly one chokepoint — easy to guard, easy to trace, cheap on the hot path (the happy path "pays nothing measurable" for the catch_unwind landing pad).
- **Security defaults inverted.** Most scrapers allow everything and add guardrails later; Obscura denies private networks and `file://` by default and makes you *ask* to point it at `127.0.0.1:3000`. For a tool whose primary consumers are agents running unsupervised, that default is the correct one.
- **MCP breadth over README minimalism.** The README documents 14 tools; `tools/list` actually serves 38, including schema-driven `browser_extract` (field→CSS-selector maps with `rows[]` list syntax and `selector@attr` attribute extraction), `browser_detect_forms`, tab and cookie/storage-state management, and token-dense `browser_markdown`. The MCP server is the CLI's richest interface — a sign the author expects agents, not humans, to be the primary driver.

## Comparison notes

- [[Lightpanda Browser]] is the closest peer: also a from-scratch headless browser for agents (in Zig), also embedding V8, also exposing CDP + MCP + a fetch CLI, also claiming order-of-magnitude memory wins. The divergence is instructive: Lightpanda gives each connection its own isolate thread and owns a 28-tool closed enum with comptime-enforced predicates and a native agent REPL with session→script replay, while Obscura keeps rendering as the centerpiece — a real CSS layout/paint engine versus Lightpanda's deliberately text-only rasterizer — and adds stealth transport engineering Lightpanda doesn't attempt. Lightpanda's screenshot is "for spatial layout, not a primary read"; Obscura's is the product.
- [[Browser Use]] bolts stealth onto a hardened Chromium fork (fingerprint randomization, residential proxies, Cloudflare bypass) and caches deterministic replay scripts. Obscura's bet is the opposite: build the stealth into the engine from scratch (consistent Chrome 145 TLS fingerprint, JS-surface randomization, tracker blocklist) rather than patching someone else's browser and fighting its default surfaces.
- [[Chawan]] is the other from-scratch browser in the wiki, but aimed at humans in a terminal: text-mode, opt-in JavaScript via QuickJS, protocol nostalgia (Gopher/Gemini). Obscura is its mirror image — JavaScript always on, rendering always aimed at pixel output, protocols narrowed to HTTP(S) and CDP — and both share the memory-safe-language thesis (Nim and Rust respectively).
- [[Chrome DevTools MCP — Debug Your Browser Session]] wraps Chrome's own DevTools protocol for agents; Obscura implements that same protocol server-side without Chrome, so Puppeteer/Playwright code and CDP-native tooling attach to a 30 MB process instead of a full browser install. Its custom `LP.getMarkdown` domain (DOM→Markdown over CDP) shows how thin the seam is between "CDP compatibility" and "agent-native API."

The honest weaknesses: the headline benchmarks (85 ms loads, 30 MB, 6× memory) are self-reported in a companion repo, the CSS long tail is a permanent maintenance treadmill, and the first build still compiles V8 from source — "lightweight" is a runtime claim, not a build-time one.

#tool #project #agents #browser-automation #rust #stealth #web-scraping

---
*Sources: [[raw/obscura]], [[summary/obscura]]*
*Last updated: 2026-09-15*
