---
url: https://github.com/rustdesk/rustdesk
title: "RustDesk — Open Source Remote Desktop"
author: "rustdesk (Purslane Tech Pte. Ltd.)"
date_fetched: 2026-07-25
date_published: 2021
---

# RustDesk — Ingest Analysis

RustDesk is an open-source, self-hostable remote desktop application written primarily in Rust with a Flutter UI frontend. It is the leading open-source alternative to TeamViewer and AnyDesk, with ~80K GitHub stars. The project is maintained by Purslane Tech Pte. Ltd. and a large open-source community.

## Architecture

### Language and Build System

- **Rust** (~23K+ lines in src/ alone, plus ~5 shared library crates) — the core, compiled as both a binary (`rustdesk`) and a library (`librustdesk`) in cdylib, staticlib, and rlib formats
- **Dart/Flutter** (flutter/lib) — the desktop and mobile UI
- **C++** (build.rs, scrap lib) — screen capture and platform interop
- **Python** (build.py, 27K lines) — build automation
- **Protobuf** — the wire protocol for all network communication

The Cargo.toml defines three binaries: `rustdesk` (default), `naming` (src/naming.rs), and `service` (src/service.rs). The library is compiled as `librustdesk` for FFI linking with Flutter via `flutter_rust_bridge`.

### Module Map

```
src/
├── main.rs              — entry point: desktop (core_main+ui) vs mobile/flutter (test+init)
├── lib.rs               — crate root, re-exports all public modules
├── client.rs            — (4327 lines) client-side connection, NAT traversal, audio/video decode
│   ├── io_loop.rs       — main I/O event loop for connected sessions
│   ├── helper.rs        — connection helpers
│   ├── file_trait.rs    — file transfer trait
│   └── screenshot.rs    — screenshot handling
├── server/              — server-side services
│   ├── connection.rs    — (7129 lines) incoming connection handler, auth, session mgmt
│   ├── video_service.rs — screen capture + encoding (H264/H265/VP8/VP9/AV1)
│   ├── audio_service.rs — audio capture + encoding (Opus)
│   ├── input_service.rs — keyboard/mouse input processing
│   ├── clipboard_service.rs — clipboard sync
│   ├── display_service.rs   — multi-monitor display mgmt
│   ├── terminal_service.rs  — remote terminal access
│   ├── printer_service.rs   — remote printing
│   ├── portable_service.rs  — portable (no-install) mode
│   └── video_qos.rs     — adaptive video quality
├── rendezvous_mediator.rs — (1002 lines) registration + connection brokering with hbbs
├── common.rs            — (3047 lines) shared state, crypto, NAT testing, config
├── flutter_ffi.rs       — (3170 lines) FFI bridge to Dart/Flutter
├── flutter.rs           — Flutter-specific session management
├── ipc.rs               — (2227 lines) inter-process communication
├── port_forward.rs      — TCP tunnel / RDP forwarding
├── platform/            — OS-specific implementations
│   ├── mod.rs           — (248 lines) trait definitions
│   ├── windows.rs       — Windows: service install, UAC, Winlogon, screen capture
│   ├── macos.rs         — macOS: accessibility, TCC permissions, retina
│   └── linux.rs         — Linux: X11/Wayland detection, display mgmt, headless
├── ui/                  — DEPRECATED Sciter UI (replaced by Flutter)
├── plugin/              — plugin framework (opt-in via feature flag)
├── whiteboard/          — cross-platform whiteboard/annotation overlay
├── hbbs_http/           — HTTP client for hbbs server API (sync, account, download)
├── keyboard.rs          — keyboard layout handling
├── clipboard.rs         — clipboard state machine
├── clipboard_file.rs    — file copy/paste between platforms
├── lang.rs              — Rust-side translation strings
├── lang/                — per-language translation modules (sq.rs, ko.rs, ru.rs, ...)
├── privacy_mode.rs      — privacy mode (blank remote screen)
├── virtual_display_manager.rs — Windows virtual display driver
├── kcp_stream.rs        — KCP reliable UDP transport
└── updater.rs           — auto-update logic
```

### Shared Libraries (libs/)

| Library | Purpose |
|---------|---------|
| `hbb_common` | Core shared: protobuf messages, TCP/UDP wrappers, config, file transfer, crypto, TLS |
| `scrap` | Screen capture: DXGI (Windows), CoreGraphics (macOS), X11/PipeWire/Wayland (Linux), MediaProjection (Android) |
| `enigo` | Cross-platform keyboard/mouse simulation |
| `clipboard` | Cross-platform clipboard (text + files) |
| `virtual_display` | Virtual display driver for headless Windows |
| `portable` | Portable app packaging |
| `remote_printer` | Remote printer redirection |

### Connection Architecture

RustDesk uses a three-tier network architecture:

1. **Rendezvous Server (hbbs)**: ID-based device registry and NAT traversal broker. Devices register their ID + public key. Clients request connections by peer ID. The rendezvous server exchanges NAT addresses to enable hole punching but does not relay data.

2. **Relay Server (hbbr)**: Fallback relay for when direct connections fail (symmetric NAT, firewall). Relays encrypted traffic between peers.

3. **Direct P2P**: The preferred path. Three concurrent strategies raced with `select_ok`:
   - **TCP hole punching**: Direct TCP connection using addresses exchanged via rendezvous
   - **UDP hole punching**: UDP NAT traversal with STUN-style port prediction
   - **IPv6 direct**: Direct IPv6 connection when both peers have IPv6
   - **KCP (KCP Stream)**: Reliable UDP transport layered on top for UDP connections

### Connection Flow (Detailed)

From `client.rs:371-631`, the `_start_inner` method:

1. Connect to rendezvous server via TCP
2. Optionally secure the TCP channel with the licence key (encrypts the token)
3. If UDP enabled, concurrently spawn UDP NAT test (measure NAT type + external port)
4. Send `PunchHoleRequest` containing: peer ID, token, NAT type, licence key, conn type, version, UDP port, IPv6 addr, force_relay flag
5. Receive `PunchHoleResponse` containing: peer's socket address, NAT type, is_local flag, signed public key, relay server address, feedback
6. **Race multiple connection attempts** via `select_ok` (`client.rs:695-709`):
   - TCP direct to peer address
   - UDP direct via NAT hole-punched socket
   - IPv6 direct via IPv6 socket
7. If all direct attempts fail and relay server is available, request relay via rendezvous server
8. **Secure handshake** (`client.rs:760-836`):
   - Server sends `SignedId` (peer ID signed with server's Ed25519 private key)
   - Client verifies signature against the public key obtained from rendezvous
   - Client generates ephemeral Curve25519 keypair, sends public key encrypted with server's public key
   - Both derive shared symmetric key via NaCl `box_` key exchange
   - All subsequent traffic encrypted with symmetric key

### Key Technical Details

1. **NAT traversal intelligence** (`client.rs:660-691`): Connection timeout is dynamically computed based on detected NAT types. Symmetric NAT → 1s. Asymmetric-to-asymmetric → full CONNECT_TIMEOUT (18s). Unknown NAT → adaptive based on punch time × multiplier. Direct failures are tracked per peer and used to adjust future timeout strategies.

2. **Audio pipeline** (`client.rs:1181-1541`): Opus audio decoding → optional sample rate conversion (via dasp/rubato/samplerate feature flags) → platform-specific output. On non-Linux: ring buffer (3 seconds at 48kHz stereo) with jitter-compensating adaptive drop (`AudioBuffer.try_shrink`). On Linux: direct PulseAudio simple API.

3. **Video pipeline** (`video_service.rs`): Multiple codec backends via feature flags — `hwcodec` (H264/H265 via platform APIs), `vram` (GPU VRAM direct), software VP8/VP9/AV1. Adaptive quality via `VideoQoS` that monitors network conditions.

4. **Plugin framework** (`plugin/`): Opt-in via `plugin_framework` feature. Native plugin DLLs loaded at runtime. Each plugin declares capabilities via a JSON descriptor. Can intercept and modify input events (`src/server/connection.rs:152-177`).

5. **Clipboard sync**: Polling-based with platform-specific listeners (`clipboard_listener`). Supports text, images, and files (via `unix-file-copy-paste` feature flag). File copy/paste uses platform-specific backends (FUSE on Linux, NSFilePromise on macOS, OLE on Windows).

6. **Security model**: End-to-end encryption by default using NaCl/libsodium. Key hierarchy:
   - Long-term: Ed25519 keypair (device identity, registered with rendezvous)
   - Per-session: Ephemeral Curve25519 for forward secrecy
   - Password: Stored as salted SHA-256 hash (`src/server/connection.rs:1716-1722`)
   - 2FA: TOTP support via `totp-rs` crate
   - Trusted devices: Device-based bypass for 2FA

### Release Profile

The release profile (`Cargo.toml:238-244`) is optimized for size:
- `lto = true` (link-time optimization)
- `codegen-units = 1` (single codegen unit for better optimization)
- `panic = 'abort'` (no unwinding)
- `strip = true` (strip symbols)

## Flutter Integration

The Rust core compiles as a cdylib linked by Flutter via `flutter_rust_bridge` v1.80. The bridge generates Dart bindings from Rust function signatures. Key bridge modules:

- `flutter_ffi.rs`: Session management, connection lifecycle, event streaming
- `flutter.rs`: State models, peer management, async task scheduling
- `flutter/lib/models/`: Dart-side models for peers, state, address book, input

The Flutter UI supports desktop (Windows/macOS/Linux), mobile (Android/iOS), and web.

## Platform Abstraction Strategy

Rather than scattering `#[cfg(target_os = ...)]` throughout the codebase, platform-specific functionality is extracted into dedicated crates:

- `scrap` — screen capture (one trait, per-platform implementations)
- `enigo` — keyboard/mouse control
- `clipboard` — clipboard read/write
- `src/platform/` — higher-level platform glue (service management, display control, window management)

This is clean but has costs: scrap and enigo are RustDesk forks (not upstream), maintained as git dependencies in Cargo.toml. Several dependencies are pinned to RustDesk-org forks (parity-tokio-ipc, magnum-opus, rdev, cpal, arboard, clipboard-master, portable-pty, etc.), creating a maintenance burden.

## Key Design Decisions

1. **Race-all-the-things connection strategy**: Instead of trying TCP first and falling back, RustDesk races TCP, UDP, IPv6, and relay simultaneously. This minimizes connection time at the cost of network overhead.

2. **Protobuf everywhere**: All network messages use Protocol Buffers. This provides cross-version compatibility (new fields default to empty) and language-agnostic serialization (the Dart side can parse messages too).

3. **Dual UI — Sciter deprecated, Flutter current**: The original UI was Sciter (a lightweight HTML/CSS engine). The project migrated to Flutter for better cross-platform support (mobile, web), but Sciter code remains in `src/ui/` (deprecated) and can still be compiled.

4. **Self-hostable by design**: The rendezvous/relay servers are a separate open-source project (rustdesk-server). Users can run their own infrastructure. The client has no hard dependency on the public servers.

5. **Plugin framework as opt-in feature**: The plugin system is behind a compile-time feature flag. This prevents accidental plugin loading and keeps the core binary smaller for users who don't need it.

6. **Size-optimized release builds**: The project prioritizes binary size over compilation speed, with aggressive LTO and symbol stripping.

## Non-Obvious Implementation Details

1. **The audio buffer's statistical shrink algorithm** (`client.rs:1231-1291`): Rather than a simple fixed-size ring buffer, the audio buffer tracks occupancy levels over 12-second windows in 30 buckets, identifies the "safe water mark" (longest consecutive near-empty period), and drops samples at a rate of `min(zero, max/N)` of the buffer capacity per adjustment. This is a sophisticated jitter-compensation strategy that avoids both underruns and excessive latency.

2. **The restart grace window** (`client.rs:100`): A 5-minute grace period where `restarting_remote_device` tracking prevents false reconnections during remote reboots. On Windows, the peer may briefly reconnect before the actual reboot disconnect, and this grace window prevents treating that as a failed reconnection.

3. **The "deploy" backoff** (`rendezvous_mediator.rs:70-79`): When a device hasn't been deployed to the server yet, registration attempts are throttled to once every 30 seconds via `deploy_register_throttled()`. This prevents tight-loop reconnection storms when the device is awaiting operator deployment.

4. **The clipboard listener unsubscribe mechanism** (`client.rs:950-952`): Clipboard monitoring uses a named subscription model (`"client-clipboard"`) with reference counting. Only when all Flutter sessions disconnect does the clipboard thread actually stop.

5. **The display connection tracking** (`video_service.rs:76`): A global `DISPLAY_CONN_IDS` map tracks which connections are subscribed to which displays. This allows targeted notifications when frames arrive, rather than broadcasting to all connections.

## Comparison Notes

- **vs TeamViewer**: Similar rendezvous/relay architecture, but RustDesk is fully open-source and self-hostable. TeamViewer uses proprietary protocols; RustDesk uses documented protobuf messages. TeamViewer's NAT traversal is more battle-tested (they've been doing this for 20+ years), but RustDesk's approach is architecturally equivalent.

- **vs AnyDesk**: AnyDesk uses a custom DeskRT video codec optimized for low latency. RustDesk uses standard codecs (H264/H265/VP8/VP9/AV1) with hardware acceleration. AnyDesk's codec is more bandwidth-efficient for text-heavy screens; RustDesk trades some efficiency for codec compatibility.

- **vs Sunshine/Moonlight**: Sunshine/Moonlight are optimized for game streaming (ultra-low latency, high FPS) using NVIDIA/AMD hardware encoders. RustDesk targets general remote desktop use with broader codec and platform support.

- **vs VNC/RDP**: VNC is pixel-based (no codec intelligence), RDP is Windows-centric. RustDesk provides codec-aware streaming with cross-platform support and NAT traversal that neither VNC nor RDP offer.
