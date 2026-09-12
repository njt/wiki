---
url: https://github.com/zellij-org/zellij/
title: "Zellij"
author: Aram Drevekenin and contributors
date_fetched: 2026-07-18
date_published: 2021-04-06
topics:
  - developer-tools
---

Zellij is a terminal multiplexer (like tmux) written in Rust, distinguished by a
WASM-based plugin system, a built-in terminal emulator, and a multi-threaded
server architecture. As of v0.45.0 it spans ~296,550 lines of Rust across 18+
crates.

The server runs six dedicated OS threads — main server loop, screen, PTY, PTY
writer, plugin manager, and background jobs — communicating via typed
multi-producer channels. Each connected client gets its own "route" thread.
Communication between the server daemon and client process uses protobuf over
Unix domain sockets (or Windows named pipes).

Instead of passing escape sequences through to the host terminal as tmux does,
Zellij implements its own terminal emulator (the `Grid` struct) via the `vte`
crate. This enables reliable scrollback, per-pane search, diff-based rendering,
Sixel image support, and sophisticated host-query forwarding using OSC 99
escape-sequence namespacing per pane.

Plugins compile to WASM and run in the wasmi interpreter (no JIT), sandboxed
with explicit resource grants. They communicate with the host via protobuf over
stdin/stdout through a `WasmBridge`. Built-in plugins include tab bars, status
bars, a session manager, a file browser (strider), and a visual config editor.

Other notable features: KDL-based configuration and layout files, session
resurrection (periodic serialization of pane layouts and working directories),
mobile mode for small screens, a built-in web server for browser-based session
access, per-client rendering (different clients see different themes), and
floating panes.

The design favors OS threads over async for core components (async is used only
for auxiliary concerns like the web server). The WASM plugin system trades raw
performance for crash isolation, portability, and hot reloading. The built-in
terminal emulator adds correctness burden but enables features impossible with
passthrough.

Compared to tmux: memory-safe Rust, sandboxed plugins, built-in web access, and
mobile support. Compared to WezTerm: Zellij is a multiplexer that works with any
terminal, not a terminal emulator itself. Compared to Ghostty: Zellij's plugin
system is a core differentiator against Ghostty's GPU-native rendering focus.
