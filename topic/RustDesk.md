# RustDesk

Open-source, self-hostable remote desktop application written in Rust with a Flutter UI. The leading open-source alternative to TeamViewer and AnyDesk, RustDesk provides encrypted remote desktop access with NAT traversal, file transfer, TCP tunneling, and multi-platform support — all without depending on any third-party infrastructure if you choose to self-host.

---

## Architecture

RustDesk is a **peer-to-peer remote desktop** with an optional **rendezvous/relay server** for NAT traversal and fallback routing. Each RustDesk instance is both client and server simultaneously.

### Three-tier network model

1. **Rendezvous server (hbbs)**: ID-based registry and connection broker. Devices register their ID + Ed25519 public key. When Alice wants to connect to Bob, the rendezvous server exchanges their NAT addresses to enable hole punching, but never relays data.

2. **Relay server (hbbr)**: Encrypted relay fallback. Used when direct P2P fails (symmetric NAT, restrictive firewalls). Traffic is end-to-end encrypted; the relay cannot decrypt it.

3. **Direct P2P**: The preferred path. Multiple transports are **raced simultaneously** via `select_ok` (`src/client.rs:695-709`):
   - TCP direct (NAT hole-punched address)
   - UDP direct (STUN-style port prediction)
   - IPv6 direct (when both peers have IPv6)
   - KCP reliable UDP (layered on UDP connections)

### Process model

```
┌──────────────────────────────────────────────────┐
│  Flutter UI (Dart)                                │
│  flutter/lib/models/ ← state, peers, input        │
├──────────────────────────────────────────────────┤
│  flutter_rust_bridge FFI                          │
├──────────────────────────────────────────────────┤
│  Rust Core (librustdesk)                          │
│  ┌─────────────┐  ┌──────────────────────────────┐│
│  │ client.rs   │  │ server/                      ││
│  │ connection  │  │ connection.rs (7129 lines)    ││
│  │ NAT traversal│  │ video_service.rs             ││
│  │ audio/video │  │ audio_service.rs              ││
│  │ decode      │  │ input_service.rs              ││
│  └─────────────┘  │ clipboard_service.rs          ││
│                    │ terminal_service.rs           ││
│  ┌──────────────┐ │ display_service.rs            ││
│  │ rendezvous   │ │ video_qos.rs                  ││
│  │ mediator.rs  │ └──────────────────────────────┘│
│  └──────────────┘                                  │
│  ┌──────────────────────────────────────────────┐ │
│  │ libs/scrap    — screen capture (5 backends)   │ │
│  │ libs/enigo    — keyboard/mouse simulation     │ │
│  │ libs/clipboard — clipboard (text + files)     │ │
│  │ libs/hbb_common — protobuf, net, crypto, fs  │ │
│  └──────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

### Connection lifecycle

1. **Registration** (`rendezvous_mediator.rs`): Device connects to rendezvous server, registers ID + public key, sends periodic keep-alive pings (default every 60s).

2. **Connection request** (`client.rs:462-473`): Client sends `PunchHoleRequest` with peer ID, NAT type, licence key, and optionally a pre-established UDP port.

3. **Hole punch** (`client.rs:484-533`): Rendezvous responds with `PunchHoleResponse` containing peer's address, NAT type, relay server, and the peer's signed public key.

4. **Transport race** (`client.rs:695-709`): TCP, UDP, IPv6, and KCP connections are all attempted concurrently. The first to succeed wins.

5. **Secure handshake** (`client.rs:760-836`): NaCl `box_` key exchange using ephemeral Curve25519, authenticated via Ed25519 signatures verified against the public key from rendezvous. Result: symmetric session key.

6. **Session** (`server/connection.rs`): Authentication (password, 2FA, trusted device check), then video/audio/clipboard/input streaming via protobuf messages over the encrypted channel.

## Key Techniques

### Multi-transport NAT traversal (client.rs)

The core innovation is **racing all transports simultaneously** rather than trying them sequentially. `select_ok` from futures picks the first successful connection. Timeouts are dynamically adjusted based on detected NAT types (`client.rs:660-691`):

- **Symmetric NAT both sides**: 1 second timeout (direct P2P nearly impossible, fall through to relay fast)
- **Asymmetric to asymmetric**: Full `CONNECT_TIMEOUT` (18 seconds) — good chance of success
- **Unknown NAT**: Adaptive — `punch_time_used × 6` multiplier, informed by historical direct failure counts

Direct failures are tracked per peer in `PeerConfig.direct_failures` and persisted. Future connections use this history to adjust strategies.

### Adaptive video encoding (video_service.rs)

The video service supports multiple codec backends via Cargo feature flags:

| Feature | Codec | Acceleration |
|---------|-------|-------------|
| `hwcodec` | H264/H265 | DXGI (Win), VAAPI (Linux), VideoToolbox (macOS) |
| `vram` | GPU direct | AMD/NVIDIA VRAM capture |
| (default) | VP8/VP9/AV1 | Software via libvpx/libaom |

`VideoQoS` (`video_qos.rs`) dynamically adjusts quality based on network conditions. Frame fetched notifiers (`FRAME_FETCHED_NOTIFIERS`) provide backpressure from decoder → encoder.

### Statistical audio jitter buffer (client.rs:1199-1319)

The `AudioBuffer` maintains a ring buffer (3 seconds at 48kHz stereo by default). Rather than a fixed water mark, it:

1. Tracks buffer occupancy in 30 buckets over 12-second windows
2. Identifies the "safe water mark" — longest consecutive near-empty period
3. Drops samples at `min(zero, max/N)` of capacity when overfull
4. Protects against underruns by scheduling callback delays when the scheduler starves the audio thread

This is more sophisticated than the typical fixed-threshold approach and was developed empirically to handle real-world jitter patterns.

### Secure handshake protocol (client.rs:760-836, common.rs)

```
Client                                    Server
  |                                          |
  |<---- SignedId(peer_id, signature) -------|  (Ed25519)
  |                                          |
  |-- PublicKey(asym, sym_encrypted) ------->|  (NaCl box_)
  |                                          |
  |<======== symmetric session key =========>|
```

- Server's long-term identity: Ed25519 keypair registered with rendezvous
- Per-session: Ephemeral Curve25519 keypair → forward secrecy
- If key mismatch: falls back to unencrypted (for direct IP connections) or errors
- Password storage: salted SHA-256 (`src/server/connection.rs:1716-1722`)
- 2FA: TOTP via `totp-rs`, with trusted-device bypass

### Plugin framework (src/plugin/)

Opt-in via `plugin_framework` Cargo feature. Native DLLs loaded at runtime, each with a JSON descriptor declaring capabilities. Plugins can intercept and modify input events (`src/server/connection.rs:152-177`) — used for privacy mode enforcement and custom input filtering.

## Design Decisions

### What they optimized for

1. **Connection reliability over elegance**: The "race everything" approach is network-noisy but maximizes the chance of a successful direct connection. Each failed transport is a cost paid in packets, not user time.

2. **Binary size over build speed**: Release builds use LTO, single codegen unit, panic=abort, and symbol stripping. This produces a smaller binary at the cost of longer compile times.

3. **Protobuf over custom binary protocol**: All messages are protobuf-encoded. This costs some wire bytes (field tags) but buys cross-version compatibility (new fields default to empty), language-agnostic parsing (Dart can parse messages too), and self-describing messages for debugging.

4. **Self-hostability over convenience**: The architecture cleanly separates rendezvous/relay from the client, and the server is open-source. Users can run their own infrastructure. The cost: more complex initial setup for self-hosters, and the project maintains two separate repos (rustdesk + rustdesk-server).

5. **Forked dependencies over upstream contributions**: Many critical dependencies (scrap, enigo, cpal, portable-pty, etc.) are pinned to RustDesk-org forks rather than upstream. This gives them control over fixes but creates a maintenance burden — they must rebase on upstream periodically.

### What they sacrificed

1. **Hardware codec integration is complex**: Unlike Sunshine/Moonlight which focus narrowly on game streaming with NVIDIA/AMD hardware encoders, RustDesk's multi-codec approach requires maintaining bindings to DXGI, VAAPI, VideoToolbox, MediaCodec, and software codecs — each with platform-specific quirks.

2. **Sciter UI is abandoned but not removed**: The original Sciter-based UI is deprecated but still in `src/ui/` and can be compiled. This is dead code that complicates the build.

3. **Mobile is secondary**: Android/iOS support exists but many features are `#[cfg(not(any(target_os = "android", target_os = "ios")))]` — clipboard, file transfer, whiteboard, terminal, port forwarding are all desktop-only.

4. **Single-threaded audio on Linux**: Linux audio uses PulseAudio simple API (blocking writes in the audio handler) while other platforms use cpal's callback-based streaming with a ring buffer. The Linux path is simpler but less resilient to scheduling jitter.

## Comparison Notes

- **[[Tunnet]]**: Both are Rust networking tools, but RustDesk is remote desktop (application layer) while Tunnet is mesh VPN (network layer). Complementary: you could use Tunnet to provide the encrypted network and RustDesk for the remote desktop protocol on top.

## Related Pages

- [[Razorback]] — another open-source Rust CLI tool with a focus on scientific benchmarking rigor
- [[Castor]] — Go CLI for streaming; shares the "self-hostable, no cloud dependency" philosophy
- [[Chawan]] — memory-safe (Nim) browser; like RustDesk, chooses memory safety as a design principle

---
*Sources: [[raw/rustdesk]]*
*Last updated: 2026-07-25*
