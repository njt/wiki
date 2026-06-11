# Clipfan

Clipfan syncs your clipboard across a fleet of macOS + Linux hosts over SSH. Copy on any machine and it lands everywhere: Mac pasteboard, every remote's OS clipboard, every tmux paste buffer. The killer feature: image paste into Claude Code and Codex CLI on headless remotes "just works" — no OSC 52, no Xvfb, no X11 bridge required.

Built by [[Prime Radiant (Company)]] (Jesse Vincent). Proprietary, Go, ~47K LoC.

## Architecture

A daemon runs on every host. It polls the local OS clipboard every 250ms, exposes a signed loopback HTTP API (`127.0.0.1:7853`), and syncs clipboard updates over authenticated SSH streams. A SwiftUI menubar app on the Mac is the control surface: installs the daemon, provisions peers over SSH, and shows fleet health.

The sync model is peer-to-peer with star-topology relay. Every daemon relays received clips onward to its configured peers (except the clip's origin). The Mac typically holds an edge to every peer it installs, so hosts that can't see each other directly still converge through the Mac. Mesh-heal provisions direct edges between peers that can reach each other, so they keep converging when the Mac is down.

Three independent layers prevent recirculation:

1. **Clip-ID dedup**: A bounded LRU set (`seenSet`, cap 256) of recently-seen random 128-bit clip IDs. Clips with no ID are dropped outright — the fleet runs a single version with no ID-less fallback.
2. **Content echo suppression**: The daemon records the content hash of every clipboard write. A subsequent read matching that hash (including an image re-read as its store-path text) is suppressed. This catches re-representations that clip-ID dedup structurally cannot.
3. **Image-path guard**: Text whose bytes match a content-addressed image-store path is never broadcast — prevents image demotion to a path string.

Conflict resolution is last-write-wins by monotonic timestamp. No CRDTs, no vector clocks — the position is that clipboard conflicts are rare in single-user fleets and the user can always re-copy.

## The image trick

This is the load-bearing innovation. When an image arrives at a host:

1. Write PNG bytes to `$XDG_STATE_HOME/clipfan/images/<sha256>.png`
2. Record `state.json` (kind=image, path) and `current.txt` (absolute path as text)
3. On macOS: the pasteboard helper writes a single `NSPasteboardItem` carrying *both* PNG bytes (`public.png`) and the file path as text (`public.utf8-plain-text`). Cmd-V into Preview/Slack pastes the image; Cmd-V into a terminal pastes the path.
4. On Linux: just the path string goes to the clipboard (no display server needed).
5. `tmux load-buffer` the path into every socket so `prefix-]` pastes it.

The result: bracketed paste in a terminal sends the image's absolute path on the remote. Claude Code and Codex CLI detect the path and attach the image file from disk. On Linux, the `clipfan-shim` symlinks in `~/.local/bin` intercept `xclip -t image/png -o` calls from Claude Code's Ctrl-V handler, serving PNG bytes from the images directory. No X server required.

## Security model

- Single shared key per fleet (base64, ≥16 bytes). HKDF-SHA256 derives separate HMAC and AES-GCM sub-keys.
- SSH stream frames are newline-delimited JSON: hello (HMAC-signed handshake), state (AES-GCM encrypted clip), ack/error.
- Envelopes carry a `recipient` field checked on open — prevents cross-peer replay even with shared key.
- Local HTTP API requires canonical request HMAC signatures with nonce replay protection (±2 min skew, 4 min nonce retention).
- Password manager pastes (macOS concealed/transient types) are detected and never synced, recorded, or relayed.
- In "safe mode" (non-loopback listener), server surface reduces to health + status + repair endpoints.

## Key techniques

- **Content-addressed image storage** by SHA-256. Same image never written twice. GC is history-aware: never deletes PNGs still referenced by pinned or retained history entries.
- **Fleet view aggregation**: parallel SSH gather of fleet-snapshot from each peer (concurrency 6, 10s timeout). Strict JSON decode — any truncation, stderr noise, or trailing data marks host unreachable rather than silently misrepresenting edges.
- **Short-name host reconciliation**: the configured peer might be `jesse-paradise-park` (tailnet) while the recv envelope is stamped `paradise-park` (hostname). `hostsMatch()` handles both exact match and the `<user>-<short>` tailnet pattern.
- **Pluggable discovery**: `tailscale status --json` or static peer list. Discovery feeds the fleet view; actual sync uses the provisioned SSH peer config, not discovery.
- **Mesh-heal**: roster-walks from seed endpoints, checks every undirected edge for bidirectional health (accept + connect + keys_ready), provisions missing edges, restarts only changed hosts. LAN address fallback for cross-tailnet edges.
- **Clipboard history**: local per-host, newest-first, content-addressed dedup (re-copying floats to top rather than duplicating). Pinned entries exempt from GC. Capped at 200 by default, configurable 50–5000.

## Design trade-offs

**Optimized for**: correctness (three independent dedup layers, strict frame validation, recipient binding), headless image paste (the whole raison d'être), and macOS/Linux developer experience.

**Sacrificed**: raw latency (250ms poll interval, not push-based), complex conflict resolution (last-write-wins is simple but can lose concurrent writes), multi-user support (explicit non-goal), rich types beyond text + PNG.

**Opinionated simplicity**: The poll loop, the bounded 256-entry seenSet with O(n) eviction, the shared fleet key — these are deliberate choices. This isn't infrastructure that needs to scale to thousands of hosts or sub-millisecond latency. It's a tool for one developer with a handful of machines.

## Comparison notes

Unlike Apple's **Universal Clipboard**, clipfan works across macOS + Linux and handles images to headless hosts. Unlike **Synergy/Barrier**, it doesn't require shared keyboard/mouse — it's clipboard-only, designed for SSH-accessible remotes. Unlike OSC 52-based clipboard sync, clipfan's SSH stream is out-of-band, carries large binary payloads, and doesn't depend on terminal emulator support.

The image-as-path pattern echoes [[Agent-Native Architectures (Every)]]'s "files as universal interface" principle. Like [[Agentcookie]] (also from the Prime Radiant ecosystem), clipfan uses Tailscale for discovery but SSH for the actual transport. See also: [[Tmux Resurrect]] for tmux session persistence.

## Tags
#tool #clipboard #sync #ssh #tailscale #golang #macos #linux #tmux #developer-tools #project

---
Source: https://github.com/prime-radiant-inc/clipfan · Ingested 2026-06-11
