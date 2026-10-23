# Petal — Screensharing as Native Windows

Petal (Kiruna Labs, Apache-2.0) is a pre-1.0 screensharing app for small engineering syncs whose whole design is organized around one bet: a shared window should arrive on your desktop as a **real, independently movable native window**, not a tile in a meeting grid. Under the hood it is a ~247K-line Rust monorepo — Tauri 2 desktop client, LiveKit SFU transport, Vercel token backend, browser client, and a signed plugin system — with an unusually honest documentation culture (a trust model, a validation page listing what *hasn't* been verified, engineering notes on crash classes). It matters to this wiki less as a meeting app than as the most thorough public treatment yet of what "an agent acting on a shared GUI with human consent gating" looks like when taken seriously.

---

## Architecture

The defining component is the **receiver-side compositor** (`compositor.rs` + `native_display.rs`): one borderless `NSPanel` per subscribed remote window, keyed by `(owner_identity, window_id)`, hosting an `AVSampleBufferDisplayLayer` fed directly from decoded `CVPixelBuffer`s — **no CPU copy** from decode to display. A small SvelteKit webview renders the window header, and a transparent click-through overlay carries telepointers. The layering is strict: `platform/` (raw FFI, no app logic) → `capture.rs`/`transport/` (ScreenCaptureKit, LiveKit publish/subscribe) → `session/` (orchestration, `SessionState`) → chrome (hover tabs, borders, consent prompts) → diagnostics.

- **Transport**: LiveKit SFU with `auto_subscribe = false` — a `NativeSubscriptionCoordinator` owns admission explicitly, accepting only remote audio and video named `petal-window-<u32>`. A camera or malformed track reaching the native compositor is an *invariant violation*, not routing. Measured cost of replacing SDK auto-subscribe: +53–104 ms to first frame, within noise.
- **Token flow**: the app never holds the LiveKit API secret; `backend/api/token.ts` mints scoped room tokens. Deliberately, a build with no `PETAL_BACKEND_URL` has *no* token backend — a from-source build can never quietly use someone else's service.
- **Windows client**: WGC capture → D3D11 compositor with a per-window **latest-frame mailbox** (frame bursts can never fill the lifecycle/geometry command queue or block a Tauri move/resize callback).
- **Plugin system** (`shared/plugin-host/`, `contracts/plugin-registry/`): sandboxed iframe plugins speak a pinned postMessage envelope (`pluginBridge` protocol, version 1) with permission scopes, rate limits, and minisign-signed registry fixtures in `contracts/`.

## Key techniques

- **Zero-copy native display path**: ScreenCaptureKit BGRA frames → LiveKit H.264 → decode to `CVPixelBuffer` → `CMSampleBuffer` → display layer, never touching the CPU. This is what makes per-window native panels viable at low latency.
- **Capability-token remote control**: each authorization gets a fresh 128-bit token bound to `targetKind` and a live `shareInstanceId`; the controller echoes it on every packet. Replay of a stale or re-granted stream stops at the host. Disable calls `revoke_all` synchronously, synthesizing key/button releases *at the moment of disabling* — no injection window after shutoff.
- **Honest result semantics**: `applied` is reserved for an observed application effect; global OS submission without one reports `submitted`. A successful `SendMessageTimeoutW` wheel delivery proves only that the window proc received the message.
- **Chromium-aware wheel injection** (`windows_remote_control.rs`): window-share wheel routes `WM_MOUSEWHEEL` via `SendMessageTimeoutW` to the shared window's own *scrollable descendant* under the cursor — resolving the actual scrollable child (not just any child) is what makes it land in editors and browsers, and synchronous delivery (not `PostMessageW`) is required because Chromium ignores posted wheels to its render-widget HWND.
- **The AI control gate** (`ai_chat/control_gate.rs`): three unit-pinned, pure rules — (1) "allow once" grants are consumed by the attempt, success or failure; (2) a stale click can't authorize a newer request (both `session_id` and `request_id` checked, so a human's yes to a harmless card can't silently approve a replaced one); (3) **refusal is sticky** — a model cannot wear the user down by repetition; only a human-initiated reset lifts it. The whole execution path is dark unless `PETAL_AI_CONTROL=1` is set exactly — a typo must not arm it — and with the switch off the tools aren't even *declared* to the model.
- **Gemini Live design choices** (`ai_chat/mod.rs`): the session runs on the *sharer's* machine (it has pixel frames, the accessibility tree, and the replay machinery); push-to-talk activity brackets keep the model from hearing humans talk amongst themselves; the model id comes from the backend so rotation is a server change.

## Design decisions

- **Trust model: "trust every authenticated meeting peer, gated by host-side checks."** No per-user allow-list; anyone in the room is a potential controller. The trade is explicit and documented rather than hidden.
- **Escalation is user-initiated, never automatic.** A failed cursor-preserving injection returns `notInjectible`; the controller may *request* full control, the sharer approves in a non-activating consent panel. Petal never auto-escalates or silently falls back — one failed operation can never acquire stronger control.
- **Fail-closed everywhere**: elevated or integrity-unknown Windows sources get only a passive unowned telepointer while WGC keeps its system border; unknown UIA text providers fail closed; a `WDA_EXCLUDEFROMCAPTURE` lease that can't be acquired falls back to the system indicator.
- **Crash-class documentation as engineering artifact**: `docs/ENGINEERING.md` documents native-window crash classes "easy to reintroduce" (e.g. never destroy a panel — hide and retire; pause display enqueue while the display sleeps). The architecture doc cites measured A/B latencies for its own design changes.
- **Honest gaps**: a legacy tokenless remote-control path "meant to last one release has outlived that" — tracked in the trust model doc, not buried in an issue.

## Comparison notes

- Compared to [[Tessera]], the consent-gated remote-access broker, Petal shares the same design instinct — human approval as the gate, fail-closed refusal, capability tokens — but applies it to *continuous GUI input* inside a multi-party room rather than one-shot TCP tunnels. Tessera approves a session; Petal approves an escalation mode and binds every packet to a rotating grant.
- Against [[RustDesk]], the incumbent open-source remote desktop, Petal's remote control is deliberately narrower and more paranoid: no silent fallback between injection routes, no auto-escalation, `submitted` vs `applied` honesty, and per-share host-side mode authority — RustDesk optimizes for "it works everywhere," Petal for "it never does something you didn't approve."
- The [[WebRTC Is the Problem]] critique targets WebRTC's meeting-grid UX assumptions; Petal keeps WebRTC media (via LiveKit) but rejects its interface model entirely — the SFU is dumb plumbing, and the product lives in the native compositor. It's a concrete answer to that essay's complaint.
- The AI control gate is a production-grade instance of what [[Cua — Computer Use Agent Platform]] builds generically: computer-use agents acting on real GUIs, but here scoped to a shared window, with the sharer's consent UI as the authorization layer and sticky refusal instead of per-action prompts.
- Its engineering culture (measured latency A/Bs for architecture changes, validation docs admitting what's untested) rhymes with the verification-first stance of [[A New Era for Software Testing]] and [[Agentic Testing]] — testing as an honest ledger, not a checkbox.

---
*Sources: [[raw/petal]], [[summary/petal]]*
*Last updated: 2026-10-24*
