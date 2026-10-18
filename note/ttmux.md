# ttmux — a Terminal Multiplexer Written Entirely by Claude Code

ttmux is a Rust terminal multiplexer (macOS/Linux, MIT, beta) that rethinks tmux: floating panes with real mouse dragging, curved borders, an in-app settings screen, kitty/iTerm2/sixel graphics passthrough, and — its sharpest differentiator — per-pane coding-agent state detection (`◐` busy, `●` wants you, `✓` done) with a "next pane that wants you" keybinding. The README's matter-of-fact note that every line was written by Claude Code makes it as much an agentic-development artifact as a tool: ~22K lines of Rust across 24 modules, with an `ARCHITECTURE.md` that reads like a module contract for a maintainer who never wrote the code.

---

## Architecture

Client-server, in the tmux lineage, but cleaner about who owns what:

- **The server owns the PTYs and the `App`; clients are only viewers** (`src/server.rs`, ~960 lines). Detaching is free because the session never lives in a client, and `ttmux upgrade` (`src/migrate.rs`) moves a live session into a new server binary — picking up a new build without killing shells.
- **Wire protocol** (`src/proto.rs`): length-prefixed JSON over a unix socket. The comment justifies JSON honestly — frames are tiny next to PTY traffic, and a readable wire beats the bytes it costs. Protocol version is negotiated in the handshake, *not* encoded in the socket path; the doc records the failure of the path-versioned design (old servers became stranded and unattachable). `Hello.proto` and `Welcome` are frozen forever so any-year builds can greet each other.
- **Per-client bounded queues** (`QUEUE: usize = 256`) with dedicated writer threads and a 5s write timeout — the app thread must never block on a socket, or one peer that stops reading freezes everyone. Slow clients skip frames until they catch up.
- **Purity discipline** (`ARCHITECTURE.md`): `layout` is geometry with no I/O, `render`/`status` only write into a ratatui `Buffer`, `input` is a pure function from key events to bytes, `agent` is a state machine over text. `src/app.rs` (3,870 lines) is the only impure hub.

## Key techniques

- **Agent detection as a small, defensive state machine** (`src/agent.rs`). A per-pane `Watcher` classifies tail text against configurable `[agents]` patterns, but with guards that show real thought about false positives: a 1.5s `GRACE` holds Busy through spinner-beat quiet gaps; only the last 5 non-empty lines count; `?`/`y/n` patterns are *end-anchored* to the final 40 chars of the last line so mid-prose punctuation can't trigger Attention; Done only fires from a Busy/Attention state. OSC 7501 program-status reports (a new spec where programs self-report `working`/`blocked`/`done`/`error`) override the heuristics — agents that speak the protocol get exact pane titles like `terraform · needs permission: apply plan?`.
- **Graphics pre-lifting** (`src/graphics.rs`): kitty (APC), iTerm2 (OSC 1337) and sixel (DCS) sequences are intercepted from the pane's byte stream *before* vt100 sees them, so they can be replayed to the host terminal — how `mpv --vo=kitty` works inside a pane.
- **A vendored, forked vt100** (`vendor/vt100/TTMUX.md`): grapheme clusters, extended SGR, and scrollback fixes, upstreamed as a documented fork rather than patched in place.
- **Layout as a tested pure module** (`src/layout.rs`, 2,170 lines): `insert` refuses any split that leaves a pane under 3x3; a `Corner` drag kind moves two perpendicular dividers at once so tiled panes resize by corner, not just edge; `snap_target()` computes a docking preview the renderer tints under the dragged pane.
- **Scripting is tmux-compatible** (`src/script.rs`, 1,735 lines): `send-keys`, `split-window`, `capture-pane` etc., targetable by pane id, with `$TTMUX_SESSION` scoping so any pane can drive its own session — and `ttmux list-commands --json` prints the entire API *for an agent to read*, making the mux itself agent-scriptable.

## Design decisions

- **Copy mode off by default** because coding agents copy on their own and want mouse drags — the defaults are tuned for the agentic workflow, not the 1990s workflow.
- **Config never hot-reloads** (`[keys]`, widgets): widget commands run as the user, so a program that writes the config file must not get its commands executed unasked. A deliberate security posture over convenience, with an explicit prefix+r reload instead.
- **Purity over power in layout**: no I/O in geometry means full unit coverage of the hardest logic in a mux, at the cost of an adapter layer (`From<Rect>`) at the ratatui boundary.
- **JSON over a binary protocol**: readable wire beats bytes, an unusual call for a hot path — defensible only because frames are diffs, not full screens.
- **Honest beta**: the README warns the config format will change before 1.0.

## Comparison notes

- **vs [[Zellij]]**: both are Rust tmux challengers, but Zellij bets on a WASM plugin platform and a built-in emulator as an application runtime; ttmux bets on a smaller surface — one crate, no plugin system, vendored emulator — plus agent awareness as the killer feature. Zellij is the platform thesis; ttmux is the workflow thesis.
- **vs [[Warp Agent CLI]]**: Warp puts a coding agent inside a terminal product and multiplexes *agents* across harnesses; ttmux stays a dumb-but-observant mux that watches whatever agents you run in its panes from the outside, via patterns and OSC 7501. Observation over orchestration.
- **vs [[Tmux Resurrect]]**: Resurrect persists tmux state across restarts from outside tmux; ttmux solves continuity structurally with a persistent server and live `upgrade`, so sessions survive binary swaps instead of being snapshot-restore events.
- As a Claude-Code-written codebase, its `ARCHITECTURE.md`-as-contract practice belongs in the same conversation as [[The Harness Is the Company]]-style arguments about what structure agentic codebases need to stay maintainable.

---
*Sources: [[raw/ttmux]], [[summary/ttmux]]*
*Last updated: 2026-10-18*
