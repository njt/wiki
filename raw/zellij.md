---
url: https://github.com/zellij-org/zellij/
title: Zellij
author: Aram Drevekenin and contributors
date_fetched: 2026-07-18
date_published: 2021-04-06
---

# Zellij — Terminal Workspace with Batteries Included

Zellij is a terminal multiplexer (like tmux) written in Rust, but with a fundamentally different architecture: it replaces tmux's monolithic C server with a multi-threaded Rust server daemon, adds a WASM-based plugin system for extensible UI, and includes its own built-in terminal emulator. As of v0.45.0, it totals ~296,550 lines of Rust across a workspace of 18+ crates.

## Architecture

### Process Model

The system splits into two processes: a **server daemon** and a **client**. Communication between them runs over Unix domain sockets (or Windows named pipes) using protobuf-serialized messages. The client handles terminal I/O and rendering; the server owns all session state, PTY management, and plugin execution.

### Server Thread Topology

The server runs six dedicated long-lived threads, each a separate event loop, communicating via multi-producer channels (a custom `Bus<T>` abstraction wrapping `crossbeam`-style channels):

1. **Main server loop** (`zellij-server/src/lib.rs` `start_server_impl`): Accepts IPC connections from clients, spawns one `route` thread per client, and dispatches `ServerInstruction` messages to coordinate the other threads. Also manages session state (`SessionState` with client tracking, pipe associations, watchers, host-query forwarding tokens).

2. **Screen thread** (`zellij-server/src/screen.rs`, 10,698 lines): The largest module. Manages the tab/pane hierarchy, layout application, rendering pipeline, mouse event routing, copy/paste, search, session resurrection, and mobile mode. Holds `BTreeMap<usize, Tab>` keyed by stable tab IDs with position-based display ordering.

3. **PTY thread** (`zellij-server/src/pty.rs`): Manages pseudo-terminal lifecycle — spawning commands, reading PTY output, forwarding bytes to the screen thread. Uses the OS's native PTY facilities.

4. **Plugin thread** (`zellij-server/src/plugins/mod.rs`): Runs the WASM runtime (wasmi interpreter). Loads, updates, renders, and unloads plugins. Communicates with plugins via protobuf messages over stdin/stdout through a `WasmBridge` abstraction.

5. **PTY writer thread** (`zellij-server/src/pty_writer.rs`): Dedicated to writing input bytes into PTYs. Separated from reading to avoid blocking.

6. **Background jobs thread** (`zellij-server/src/background_jobs.rs`): Handles session serialization (periodic snapshots for resurrection), web server management, and other async tasks.

Each thread gets a `Bus<T>` — a container holding its channel receivers plus cloned senders to all other threads. The `ThreadSenders` struct (`zellij-server/src/thread_bus.rs`) provides typed `send_to_screen()`, `send_to_pty()`, `send_to_plugin()`, etc. methods with error propagation.

### Route Thread (per-client)

Each connected client gets a dedicated `route` thread (`zellij-server/src/route.rs`) that reads `ClientToServerMsg` protobuf messages from the IPC stream, translates user actions into the appropriate thread instructions, and dispatches them. The route thread handles action completion timeouts (1 second default) and coordinates between the client and server threads.

### Client Architecture

The client (`zellij-client/src/lib.rs`, 1,517 lines) runs in a separate process. It handles:
- **Terminal I/O**: Reading from stdin (with ANSI/vt parsing), writing to stdout
- **Kitty keyboard protocol**: Full implementation for enhanced key event reporting (`ENTER_KITTY_KEYBOARD_MODE` / `EXIT_KITTY_KEYBOARD_MODE`)
- **Host query forwarding**: OSC 99 notification protocol — the client subscribes to host terminal theme notifications (CSI 2031) and queries the host theme (DSR 996), forwarding responses to the server
- **Web client support**: WebSocket-based client for browser access to sessions
- **Remote attach**: Connects to sessions over HTTP/WebSocket

### Rendering Pipeline

1. Terminal panes receive PTY output bytes → fed to a `vte::Parser` which advances a `Grid` state machine
2. The `Grid` (`zellij-server/src/panes/grid.rs`, 5,044 lines) is Zellij's built-in terminal emulator — maintains character cells, cursor position, scrollback, selection state, and Sixel image support
3. Plugin panes render via WASM — the plugin's `render()` function returns bytes that get composited into the output
4. The `Tab` (`zellij-server/src/tab/mod.rs`, 7,547 lines) composes pane grids, adds UI chrome (pane frames, tab bar), and produces per-client render output
5. The `Output` module (`zellij-server/src/output/mod.rs`) serializes rendered content as `CharacterChunk` and `SixelImageChunk` structures
6. Rendered output flows: Screen → Server → Client → terminal stdout

## Key Techniques

### Custom Terminal Emulator (Grid)

Unlike tmux which largely passes through terminal escape sequences, Zellij has its own full terminal emulator. The `Grid` struct maintains a 2D character buffer with attributes (colors, bold, italic, underline, etc.), scrollback, cursor state, and handles the full VT/xterm escape sequence repertoire via the `vte` crate. This enables:
- Reliable copy/paste from scrollback across all panes
- Per-pane scrollback with search
- Sixel image rendering (inline graphics in the terminal)
- Hyperlink tracking with OSC 8 support
- Pane content diffing for efficient rendering

### OSC 99 Notification Forwarding (Host Query System)

Zellij implements a clever multiplexer-aware notification system using OSC 99 escape sequences:

1. When a terminal application inside a pane sends an OSC 99 notification, the `Grid` **namespaces** the `i=` (notification ID) field with the pane ID: `i=p<pane_id>[r][q].<original_id>`
2. The namespaced notification is forwarded through the client to the **host terminal** (the actual terminal emulator running Zellij)
3. When the host terminal responds, the response is **denormalized** back — the pane ID prefix is stripped, the original ID is restored, and the response is routed to the correct pane's PTY
4. The `forward_paused` flag on `TerminalPane` suspends processing of further PTY output until the host reply arrives, ensuring correct stream ordering
5. Stuck forwards (clients disconnecting mid-query) are cleaned up by synthesizing empty replies, preventing the system from deadlocking

This is more sophisticated than tmux's approach to terminal queries and enables proper integration with modern terminal features.

### WASM Plugin System (wasmi-based)

Plugins compile to WASM and run in a sandboxed interpreter (wasmi, NOT a JIT). Communication between the host and plugin uses protobuf messages through stdin/stdout:

**Plugin API** (`zellij-tile/src/lib.rs`):
- `ZellijPlugin` trait: `load()`, `update(Event) -> bool`, `pipe(PipeMessage) -> bool`, `render(rows, cols)`
- `ZellijWorker` trait: Background workers with message passing — `on_message(message, payload)`
- `register_plugin!()` macro: Generates WASM export functions (`load`, `update`, `pipe`, `render`, `plugin_version`) that deserialize protobuf from stdin and call the trait methods
- `shim` module: Host function calls (`subscribe`, `unsubscribe`, `set_selectable`, `open_file`, `switch_tab`, etc.) serialized as protobuf and written to stdout

**Plugin loading** (`zellij-server/src/plugins/wasm_bridge.rs`):
- `PluginLoader`: Handles downloading and caching plugins from URLs/filesystem
- `PluginMap`: Tracks running plugins, their subscriptions, and client associations
- `WasmBridge`: Orchestrates plugin lifecycle — load, update, render, resize, unload
- `zellij_exports.rs`: WASI host functions exposed to plugins (file I/O, stdin/stdout)
- `pipes.rs`: CLI pipe mechanism — external processes can pipe data to plugins

**Built-in plugins** (in `default-plugins/`): tab-bar, status-bar, compact-bar, session-manager, configuration (setup wizard), plugin-manager, layout-manager, strider (file browser), about, share, multiple-select, link, mobile.

### Per-Client Configuration Model

Zellij supports per-client runtime configuration that overrides the saved config. The `SessionConfiguration` struct maintains:
- `saved_config`: The config as it exists on disk
- `runtime_config: HashMap<ClientId, Config>`: Per-client overrides

When a client reconfigures (e.g., changes theme), only that client's view changes. Writing to disk propagates the change to all clients. This enables the "configuration" plugin to act as a visual settings editor.

### Session Resurrection

The background jobs thread periodically serializes session state (tab layout, pane positions, working directories, running commands) to disk. On restart, Zellij can reconstruct the session layout. The serialization uses a custom format in `zellij-utils/src/session_serialization.rs`.

### Layout System

Layouts are defined in KDL (KDL Document Language) — a document-oriented configuration language. Zellij parses KDL config files for both session layouts and user configuration. The layout system supports:
- Tiled layouts (split panes in various directions)
- Floating panes (overlay windows with x/y/width/height)
- Swap layouts (predefined alternative arrangements)
- Template-based layouts (parameterized configurations)
- Tab groups (multiple tabs in a single layout definition)

### Mobile Mode

Zellij adapts to small screens via a mobile mode (`zellij-server/src/mobile_mode.rs`). When the viewport falls below configurable threshold dimensions (default 60 columns × 30 rows), Zellij can enter a simplified single-pane-at-a-time view. The `MobileLayoutConfiguration` routes different client types (terminal vs. web) to mobile mode based on configurable rules.

### Web Server and Session Sharing

Zellij can serve sessions over HTTP/WebSocket, enabling browser-based terminal access. Features include:
- Authentication tokens (read-only and read-write)
- Session sharing (multiple clients viewing/interacting with the same session)
- Web client connection management
- HTTPS with optional certificate enforcement

## Design Decisions

### Multi-Threaded over Async

Zellij uses OS threads with blocking channel operations rather than an async runtime for its core server components. Each major subsystem (screen, PTY, plugins, PTY writer, background jobs) gets its own dedicated thread with an event loop. This is a deliberate choice: terminal multiplexing is I/O-bound but CPU-light, and threads provide simpler reasoning about state ownership and backpressure than async tasks. The async runtime (tokio) is used only for auxiliary concerns: file watching, web server, and the client's async runtime.

### WASM over Native Plugins

By choosing WASM (and the wasmi interpreter specifically, not a JIT), Zellij gets:
- **Sandboxing**: Plugins cannot access the filesystem, network, or OS unless explicitly granted
- **Portability**: Plugins work identically on all platforms
- **Crash isolation**: A crashing plugin cannot bring down the server
- **Hot reloading**: Plugins can be updated without restarting the session

The trade-off is performance — WASM interpretation is slower than native code, but for UI plugins (status bars, tab bars) this is negligible. The `zellij_exports.rs` file gatekeeps all host resources.

### Built-in Terminal Emulator over Passthrough

Tmux mostly passes escape sequences through to the host terminal, adding its own wrapping. Zellij opted to implement a full terminal emulator (the `Grid`). This is more work but enables:
- Reliable scrollback and copy/paste independent of the host terminal's capabilities
- Per-pane rendering that can be diffed (only changed cells are sent to the client)
- Proper Sixel image support
- Future-proof terminal feature support (kitty keyboard protocol, OSC 99 notifications)

The trade-off is correctness burden — Zellij must correctly implement the entire VT/xterm escape sequence repertoire, and bugs in the emulator manifest as visual glitches.

### KDL Configuration over YAML/TOML

Zellij switched from YAML to KDL for configuration files. KDL is a document-oriented language that's more human-writable than JSON/TOML but more structured than YAML. This choice reflects Zellij's design philosophy: terminal tools should have readable, maintainable configuration.

### Separate PTY Write Thread

The PTY writer thread is a dedicated thread for writing input to PTYs, separated from the PTY reader thread. This eliminates a class of deadlocks where the reader thread blocks on a write while holding state the writer needs. It's a concurrency pattern explicitly designed for correctness over minimalism.

## Comparison Notes

### vs tmux
- **Language**: Rust vs C — memory safety and modern tooling
- **Plugin system**: WASM-based with UI primitives vs shell scripts and `display-message` hacks
- **Terminal emulation**: Built-in Grid vs passthrough — enables better scrollback, search, and rendering but adds complexity
- **Configuration**: KDL vs tmux's custom config syntax
- **Layout**: First-class floating panes and swap layouts vs tmux's simpler pane model
- **Mobile**: Built-in mobile adaptation vs none
- **Web access**: Native web server vs third-party solutions (ttyd, gotty)

### vs WezTerm
- **Scope**: Zellij is a multiplexer (uses your existing terminal); WezTerm is a terminal emulator with built-in multiplexing
- **Plugin language**: WASM (any language that compiles to WASM) vs Lua
- **GPU rendering**: WezTerm uses GPU-accelerated rendering; Zellij renders to character cells

### vs Ghostty
- **Rendering**: Ghostty uses GPU-native rendering via platform APIs (Metal, OpenGL); Zellij is character-cell based
- **Plugin model**: Ghostty has no plugin system; Zellij's WASM plugin system is a core differentiator
- **Multiplexing**: Ghostty is a terminal emulator first; Zellij is a multiplexer first

## Innovation Points

1. **WASM plugin system with protobuf IPC**: The most architecturally novel aspect. No other terminal multiplexer has a sandboxed, language-agnostic plugin system. The `register_plugin!()` macro compiles to WASM exports that communicate via protobuf — the host never trusts the plugin's memory.

2. **Host query forwarding with pane routing**: The OSC 99 notification namespacing is a genuinely clever solution to the multiplexer problem — how does a terminal app inside a multiplexer communicate with the host terminal? Zellij's answer (namespace queries with pane IDs, route responses back) is more sophisticated than tmux's approach.

3. **Per-client rendering**: Each attached client gets independently rendered output, enabling different clients to see different themes, pane frame styles, and UI configurations simultaneously.

4. **Session resurrection with layout preservation**: Not just saving pane layouts but attempting to reconstruct working directories and running commands.

5. **Mobile mode with automatic detection**: Terminal multiplexers typically assume large screens. Zellij's mobile mode acknowledges that people SSH from phones and tablets.

## Weaknesses and Trade-offs

1. **Grid complexity**: The built-in terminal emulator means Zellij must reimplement what every terminal already does. Bugs in escape sequence handling (especially edge cases with cursor positioning, scroll regions, or alternate screen) are notoriously hard to fix.

2. **WASM performance ceiling**: wasmi interpretation limits plugin performance. Complex plugins (e.g., a file browser rendering large directory trees) may feel sluggish compared to native implementations.

3. **Thread count**: Six+ dedicated threads per session plus one per client adds up. While threads are cheap on modern OSes, a system running many Zellij sessions could face scheduler pressure.

4. **Protobuf overhead**: Using protobuf for IPC between client and server (rather than a simpler text or binary protocol) adds serialization/deserialization overhead to every keypress and render frame. This is mitigated by the fact that terminal I/O is low-bandwidth.

5. **Single binary coupling**: Despite the crate split, all server components compile into one binary. The WASM plugin system is the only truly isolated extension point.
