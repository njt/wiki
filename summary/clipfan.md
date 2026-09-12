---
url: https://github.com/prime-radiant-inc/clipfan
title: Clipfan
author: Prime Radiant, Inc. (Jesse Vincent)
date_fetched: 2026-06-11
date_published: 2026-05-28
topics:
  - misc
---

# Clipfan — Fleet Clipboard Sync with Headless Image Paste

A Go daemon (1.26, ~47K LoC excluding tests) that syncs the clipboard across a fleet of macOS + Linux hosts over authenticated SSH streams. The standout feature: image paste into Claude Code and Codex CLI on headless remotes "just works" without OSC 52, Xvfb, or per-SSH session state.

## Architecture

### Module layout
```
cmd/
  clipfan/             Daemon entrypoint + subcommand dispatch (main.go, 207 lines)
  clipfan-shim/        xclip / wl-paste replacement for Linux remotes
  generate-ssh-release-gates/  Build tool for release-gate constants
internal/
  cli/                 Subcommands: copy, paste, ssh-gateway, mesh-heal, provisioning (430 lines mesh-heal)
  clipboard/           Per-platform OS clipboard read/write (clipboard.go + _darwin.go + _linux.go + selection.go)
  config/              JSON config v2, SSH peer config, listener repair, fleet reset (1991 lines ssh_peer_config.go)
  daemon/              Core loop: poll, onReceive, publish, seenSet, fleet aggregation (862 lines daemon.go)
  discovery/           Pluggable: Tailscale status --json or static peer list
  localdaemon/         Local daemon discovery, startup, and recovery helpers
  releaseflags/        Build-time release gates (config v2, SSH runtime/transport)
  sshprovision/        SSH peer provisioning: known_hosts/authorized_keys management, pair plans
  store/               XDG state dir: state.json, current.txt, images/<sha>.png, history.json
  transport/           Signed HTTP API + SSH sync stream framing + auth + crypto (960 lines server.go, 887 lines ssh_stream.go)
  tmux/                load-buffer-all across tmux sockets
  version/             Build version stamp
apps/
  mac/Clipfan/         SwiftUI menubar app (Clipfan.app)
```

### Core loop (daemon.go:536-545)
1. HTTP server starts on `127.0.0.1:7853`
2. SSH sync sessions connect to configured peers
3. 250ms poll ticker reads local OS clipboard
4. On change: mint random 128-bit clip ID, add to seenSet, save to history + image store, publish over SSH
5. On receive: dedup check (seenSet + echo suppression + image-path guard), write OS clipboard + tmux load-buffer + relay to other peers

### Recirculation prevention — three independent layers
1. **Clip-ID dedup** (seenSet, seen.go:9-34): Bounded LRU set of 256 recent clip IDs. Map + slice, evicts oldest on overflow. Simple O(n) eviction that's bounded at 256 — explicit correctness over probabilistic structures.
2. **Content echo suppression** (daemon.go:625-641): Records currentClip {id, kind, hash, imagePath} on every write. A subsequent clipboard read matching hash or imagePath string is suppressed — catches re-represented content that clip-ID dedup structurally cannot see (a path has a fresh hash, tmux re-submits under a new ID).
3. **Image-path guard** (daemon.go:583-584): Never broadcast text whose bytes match a content-addressed image store path — prevents image demotion to path string.

### Image flow — the load-bearing trick (ARCHITECTURE.md:202-228)
1. Image arrives at host → dedup against seenSet
2. Write PNG to `$XDG_STATE_HOME/clipfan/images/<sha256>.png`
3. Record state.json (kind=image, path) + current.txt (absolute path as text)
4. Set OS clipboard: macOS → bundled pasteboard helper writes single NSPasteboardItem carrying BOTH PNG bytes (public.png) AND file path text (public.utf8-plain-text). Linux → text-only (path string)
5. tmux load-buffer the path into every socket

Result: Cmd-V in a terminal sends the path as bracketed paste → Codex/Claude Code detects the path and attaches the image file. On Linux, Claude Code's Ctrl-V calls the xclip shim which serves PNG bytes from the images/ directory.

### SSH sync stream protocol (ssh_stream.go)
Newline-delimited JSON frames over SSH pipe:
- **hello**: purpose, host_id, peer_id, protocols[1], HMAC-signed with nonce replay protection
- **state**: seq, sender, clip envelope (AES-GCM encrypted, base64 body + nonce)
- **ack**: seq, clip_id, status (applied/ignored_seen/ignored_echo/ignored_older/ignored_concealed/rejected)
- **error**: code

Frames validated strictly: known fields only, no duplicates, no trailing JSON, bounded size (90MB frame, 64MB payload).

### Auth model
- Single shared key per fleet (base64, ≥16 bytes decoded)
- HKDF-SHA256 derives sub-keys: request HMAC key, SSH hello HMAC key, body AEAD key
- Canonical request signing: `method\nuri\nts\nnonce\nauth_version=v\nbody`
- Signed responses echo request nonce
- Nonce replay protection with 4-minute bucket retention
- v2 config requires versioned `clipfan-v1/request-hmac` auth header; mixed fleets fail closed

### Fleet view aggregation (fleet_aggregate.go)
- Parallel SSH gather of fleet-snapshot from each peer (bounded concurrency 6, 10s per-peer timeout)
- Strict JSON decode — any truncation, stderr noise, malformed JSON, trailing data, or missing origin marks host unreachable
- 8-second cache TTL so Mac refreshes don't re-SSH the whole fleet on every poll

## Key techniques

1. **Image-as-path trick**: Images content-addressed as `<sha>.png`; path string propagated as clipboard text. Bracketed paste + file path = image attachment for coding agents. No Xvfb, no x11-bridge needed.
2. **Dual-target NSPasteboardItem**: The `clipfan-pasteboard-helper` Swift tool writes a single item carrying both PNG bytes (Cmd-V into Preview/Slack) AND UTF-8 text path (Cmd-V into terminal). macOS API quirk exploited deliberately.
3. **xclip/wl-paste shim**: Symlinks in `~/.local/bin` intercept Claude Code's `xclip -t image/png -o` calls on headless Linux, serving PNG bytes from the images directory.
4. **Recipient-bound envelopes**: Each envelope carries `recipient` checked on open — prevents cross-peer replay even with shared fleet key. Short-name normalization (`.local` and FQDN normalize to same host).
5. **HKDF key separation**: Shared key never used raw — derived into separate HMAC and AES-GCM sub-keys via labeled HKDF-SHA256.
6. **Strict frame validation**: SSH stream JSON frames validated against known field whitelist; unknown fields, duplicate keys, and trailing data all rejected. This is defensive: a corrupted frame silently misrepresenting clip state would be worse than detecting and surfacing the corruption.
7. **Safe mode**: When listener is non-loopback, server surface reduces to health + status + listener-repair endpoints. All other routes return 409. App can repair via PATCH to move listener back to loopback.
8. **Concealed clip privacy**: macOS transient/concealed pasteboard types detected; sync, history, and relay all skip these. Never leaked.
9. **Mesh-heal**: Discovers fleet by roster-walking from seed endpoints, checks every undirected edge for health (bidirectional accept+connect+keys_ready), provisions missing edges via SSH pair provisioning, restarts only changed hosts. LAN address fallback for cross-tailnet mesh edges.

## Design decisions

- **Poll over push**: 250ms poll interval is deliberately simple. OS-level clipboard change notifications are unreliable across platforms (macOS `NSPasteboard.changeCount` is polling anyway, Linux selection ownership is fragile). The cost is latency — up to 250ms before a copy propagates.
- **Last-write-wins with monotonic timestamps**: No CRDTs, no vector clocks. Deliberate trade-off: correctness sacrificed for simplicity because (a) clipboard conflicts are rare in single-user fleets, (b) the user can always re-copy if the wrong version wins, (c) implementing proper CRDTs for clipboard would be massive over-engineering.
- **Star topology with mesh augmentation**: Mac holds edge to every peer it installs (star); mesh-heal provisions direct edges between peers that can reach each other (mesh). Peers with no edge and no common relay host wait — no DHT, no NAT traversal.
- **Shared key over PKI**: Single fleet-wide pre-shared key rather than per-peer certificates. Simpler setup at cost of weaker per-peer isolation. Mitigated by recipient binding in envelopes.
- **Loopback-only API by default**: Daemon binds 127.0.0.1. The menubar app polls over loopback (exempt from macOS Local Network privacy gate). Safe mode catches and repairs any non-loopback config.
- **Proprietary license**: Closed source. Built by Prime Radiant (Jesse Vincent's company), which also builds Serf (coding agent) and Clearance (Markdown viewer).
- **Bounded seenSet at 256**: Small enough that the O(n) slice-shift eviction is negligible. 256 entries covers ~64 seconds of sustained copying at human speed — plenty for mesh dedup where the window of interest is seconds, not hours.

## Comparison notes

- Unlike **Universal Clipboard** (Apple's Continuity), clipfan works across macOS + Linux, handles images to headless hosts, and integrates with tmux. Apple's solution is seamless but Apple-only and text-focused.
- Unlike **Synergy/Barrier** (KVM sharing), clipfan doesn't require a shared keyboard/mouse — it's clipboard-only and designed for SSH-accessible remote hosts.
- Unlike OSC 52-based solutions (terminal escape sequences for clipboard), clipfan's SSH stream is out-of-band, can carry large binary payloads, and doesn't depend on terminal emulator support. OSC 52 is used only as a fallback in the tmux snippet for non-clipfan terminals.
- Unlike **[[Agentcookie]]** (same ecosystem — session state sync over Tailscale), clipfan syncs clipboard content rather than browser cookies. Both use Tailscale for discovery but clipfan uses SSH for transport.
- Like **[[Prime Radiant (Company)]]**'s other tools ([[Serf]], [[Clearance]]), clipfan is practical infrastructure for the AI-augmented developer. It solves a real problem that emerged from using coding agents on remote machines.
- The image-as-path pattern echoes **[[Agent-Native Architectures (Every)]]**'s "files as universal interface" principle — the file path is the intermediary that lets any tool (tmux, terminal, coding agent) participate in image sharing without special protocols.

## Tags
#tool #clipboard #sync #ssh #tailscale #golang #macos #linux #tmux #developer-tools

Source: https://github.com/prime-radiant-inc/clipfan · Ingested 2026-06-11
