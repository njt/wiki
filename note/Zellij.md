# Zellij

A terminal workspace and multiplexer written in Rust (~296K lines) that replaces tmux's monolithic architecture with a multi-threaded server daemon, a WASM-based plugin system for extensible UI, and its own built-in terminal emulator. The most architecturally ambitious terminal multiplexer — pushing the boundary from "split panes and tabs" toward a programmable terminal application platform.

---

## Architecture

Zellij is a **client/server system** with the server running as a daemon and the client as a separate process. Communication flows over Unix domain sockets (Windows named pipes) using protobuf-serialized messages.

The server is organized into **six dedicated threads**, each an event loop with its own channel receivers and cloned senders to all other threads (`ThreadSenders` in `zellij-server/src/thread_bus.rs`):

| Thread | Source | Role |
|--------|--------|------|
| Main loop | `zellij-server/src/lib.rs` | Accepts IPC connections, dispatches `ServerInstruction`s, manages session state |
| Screen | `zellij-server/src/screen.rs` (10,698 lines) | Tab/pane hierarchy, layout, rendering pipeline, mouse, search, resurrection |
| PTY | `zellij-server/src/pty.rs` | Pseudo-terminal lifecycle — spawn, read output, route to screen |
| Plugin | `zellij-server/src/plugins/mod.rs` | WASM runtime (wasmi), plugin lifecycle, protobuf IPC to plugins |
| PTY writer | `zellij-server/src/pty_writer.rs` | Dedicated thread for writing input to PTYs (separated from reading to avoid deadlocks) |
| Background jobs | `zellij-server/src/background_jobs.rs` | Session serialization, web server, async maintenance |

Each connected client gets a dedicated **route thread** (`zellij-server/src/route.rs`) that reads protobuf messages from the IPC stream, translates user actions into thread instructions, and manages action completion with 1-second default timeouts.

### Client (`zellij-client/`)

The client process handles terminal I/O (raw mode stdin/stdout), implements the **kitty keyboard protocol** for enhanced key reporting, manages host terminal theme subscriptions (CSI 2031), and supports two connection modes: local Unix socket and web WebSocket. The `input_handler.rs` maps parsed input to `ClientToServerMsg` actions; the `os_input_output.rs` handles rendering server output to the terminal.

### Rendering Pipeline

1. PTY output → `vte::Parser` → `Grid` (built-in terminal emulator, 5,044 lines in `zellij-server/src/panes/grid.rs`)
2. Grid maintains character cells, cursor, scrollback, selection, Sixel images
3. `Tab` (`zellij-server/src/tab/mod.rs`, 7,547 lines) composes pane grids with UI chrome (frames, tab bar)
4. `Output` module serializes as `CharacterChunk` and `SixelImageChunk`
5. Rendered per-client (different clients can see different themes/styles)
6. Screen → Server → Client → terminal stdout

## Key Techniques

### Built-in Terminal Emulator (Grid)

Unlike tmux which passes through escape sequences, Zellij's `Grid` is a full VT/xterm emulator. This enables reliable scrollback/search independent of the host terminal, per-pane content diffing for efficient rendering, Sixel image support, and hyperlink tracking (OSC 8). The `namespace_notification_id()` function in `grid.rs` implements the multiplexer-aware OSC 99 notification routing.

### OSC 99 Host Query Forwarding

Zellij's cleverest protocol-level innovation. When a terminal app inside a pane sends OSC 99, the Grid namespaces the notification ID with the pane ID (`i=p42r.mynotif`), forwards it through the client to the host terminal, then denormalizes the response (strips pane ID, restores original ID) and routes it to the correct pane's PTY. The `forward_paused` flag suspends further PTY processing until the reply arrives, ensuring correct stream ordering. Stuck forwards (disconnected clients) are cleaned up with empty synthetic replies to prevent deadlocks.

### WASM Plugin System

Plugins compile to WASM and run in the **wasmi interpreter** (not JIT — deliberate choice for sandboxing and portability). Communication uses protobuf over stdin/stdout:

- `ZellijPlugin` trait (`zellij-tile/src/lib.rs`): `load()`, `update(Event) -> bool`, `pipe(PipeMessage) -> bool`, `render(rows, cols)`
- `register_plugin!()` macro: Generates WASM exports that deserialize protobuf from stdin, call trait methods
- `shim` module: Host function calls serialized as protobuf to stdout — `subscribe()`, `open_file()`, `switch_tab()`, etc.
- `WasmBridge` (`zellij-server/src/plugins/wasm_bridge.rs`): Orchestrates loading, updating, rendering, resizing
- `PluginMap`: Tracks subscriptions, client associations, event routing
- `zellij_exports.rs`: WASI host functions gating filesystem/network access

Built-in plugins include tab-bar, status-bar, compact-bar, session-manager, configuration (setup wizard), plugin-manager, layout-manager, strider (file browser), and mobile.

### Per-Client Configuration

`SessionConfiguration` maintains `saved_config` (disk) + `runtime_config: HashMap<ClientId, Config>` (per-client overrides). A client changing its theme affects only that client. Writing to disk propagates the change to all clients. This enables the visual configuration editor plugin.

### Layout System (KDL)

Layouts defined in KDL (a document-oriented language replacing the earlier YAML). Supports tiled panes (nested splits), floating panes (overlay windows with absolute coordinates), swap layouts (alternative arrangements), template-based layouts, and tab groups.

### Mobile Mode

Auto-detects small viewports (< 60 cols × 30 rows by default) and switches to simplified single-pane view. Configurable routing rules for web vs. terminal clients.

## Design Decisions

**Multi-threaded over async**: OS threads with blocking channels for core components. Each subsystem owns its state; simpler reasoning about ownership and backpressure than async. Tokio used only for auxiliary concerns (file watching, web server).

**WASM over native plugins**: Sandboxing, portability, crash isolation, hot reloading. Trade-off: interpretation overhead, acceptable for UI plugins.

**Built-in terminal emulator over passthrough**: More work, but enables reliable scrollback/copy/search, Sixel images, and future-proof feature support. Trade-off: correctness burden — must implement the entire VT/xterm escape sequence repertoire.

**Separate PTY write thread**: Eliminates a class of deadlocks where the reader blocks on write while holding state the writer needs. Correctness over minimalism.

## Comparison

- **vs tmux**: Rust vs C, WASM plugin system vs shell scripts, built-in Grid vs passthrough, KDL vs custom config, floating panes, mobile mode, native web server
- **vs WezTerm**: Multiplexer vs terminal emulator with multiplexing, WASM vs Lua plugins, character-cell rendering vs GPU rendering
- **vs Ghostty**: No plugin system (Ghostty) vs WASM plugins (Zellij), GPU-native rendering vs character-cell, terminal emulator first vs multiplexer first

## Innovation Points

1. **WASM plugin system with protobuf IPC**: No other terminal multiplexer has a sandboxed, language-agnostic plugin system
2. **OSC 99 notification namespacing**: Genuinely novel solution to the multiplexer forwarding problem
3. **Per-client rendering**: Different clients see different themes and UI configurations simultaneously
4. **Session resurrection**: Reconstructs layouts, working directories, and running commands
5. **Mobile mode**: Acknowledges that people SSH from phones

---

*Source: [[raw/zellij]]*
*Last updated: 2026-07-18*
