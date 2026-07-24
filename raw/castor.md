---
url: https://github.com/stupside/castor
title: Castor
author: stupside
date_fetched: 2026-07-25
date_published: unknown
---

# Castor — Repository Analysis

A Go CLI (~10,640 lines) that casts web video to smart TVs at full quality by extracting the real video stream URL, transcoding it for the target device, and serving it over the local network. Written by stupside. Licensed under MIT.

## Project Structure

```
castor/
  main.go                    — entry point: signal-aware context, structured logging
  cmd/                       — CLI tree (urfave/cli v3):
    cmd.go                   — root command, lazy config loading with sync.Once
    cmd-cast.go              — cast subcommands: interactive, player, url, movie, episode
    cmd-scan.go              — device discovery
  internal/
    config/                  — YAML config loading (koanf), typed composition
    cast/                    — the cast pipeline (planner/executor):
      cast.go                — Play entry point: resolve → plan → run
      plan.go                — BuildPlan: DLNA vs Chromecast decisions
      pipeline.go            — runSpooled: pull → spool → transcode → serve
      source.go              — upstream pull (ffmpeg remux → spool)
      deliver.go             — runDirect, serveToDevice
      subtitle.go            — whisper transcription + drawtext burn-in
      gate.go                — playback gate: waits for whisper lead + spool data
      config.go              — typed config compose-and-forward pattern
      spool/                 — append-only on-disk buffer with blocking tails
      replay/                — HTTP server replaying spool from byte 0 per client
      whisper/               — whisper.cpp Go bindings for live transcription
      cue/                   — subtitle word → cue shaping (timing, wrapping)
      ffmpeg/                — ffmpeg command builders, encoder registry, process mgmt
    device/                  — device discovery (SSDP, mDNS) and control protocols:
      device.go              — Device interface, Discover, Connect, FindInfo
      dlna.go                — DLNA/UPnP AVTransport + ConnectionManager negotiation
      chromecast.go          — Chromecast (experimental)
    source/
      extract/               — headless Chrome stream extraction:
        extractor.go         — CDP-based network traffic capture
        collector.go         — stream URL collection, dedup, ranking
        session.go           — browser lifecycle, navigation, action pipeline
        stealth.go           — anti-detection (WebGL, canvas, font metrics, etc.)
        turnstile.go         — Cloudflare Turnstile bypass
        js/                  — injected JavaScript (17 stealth/modification scripts)
      resolve/               — stream resolution and ranking:
        resolve.go           — HLS variant selection, bandwidth-based ranking
        probe.go             — ffprobe-based stream inspection
        hls.go               — HLS playlist parsing
    media/                   — shared types: Stream, ProbeInfo, Renderer, content types
    browse/                  — Bubble Tea TUI for TMDB browsing:
      model.go               — main model: tabs, search, discover, drilldown
      inspector.go           — poster + metadata panel
      genre.go               — genre filter modal
      drilldown.go           — TV seasons → episodes navigation
      picker.go              — device picker
      discover.go            — TMDB discover with genre/sort filters
      tmdb/client.go         — TMDB API client
  third_party/whisper.cpp/   — vendored whisper.cpp submodule (cgo bindings)
```

## Dependency Map (go.mod)

Key direct dependencies:
- `chromedp/chromedp` + `chromedp/cdproto` — headless Chrome via DevTools Protocol
- `charmbracelet/bubbletea`, `bubbles`, `lipgloss` — TUI framework
- `ggerganov/whisper.cpp/bindings/go` — vendored via `replace` directive (cgo)
- `huin/goupnp` — SSDP discovery + UPnP SOAP actions
- `vishen/go-chromecast` — Chromecast control
- `knadh/koanf` — configuration loading (YAML + env)
- `urfave/cli/v3` — CLI framework
- `golang.org/x/sync` — errgroup for pipeline coordination

Requires Chrome/Chromium, ffmpeg 7.1+, and ffprobe 7.1+ on PATH.

## Architecture

**Pattern**: Single-binary CLI → extraction → resolution → cast pipeline (planner/executor with async device connection).

The cast pipeline is the heart of the project. It uses a **planner/executor split**: `BuildPlan` (cast/plan.go) makes every decision up front based on device type and source properties, producing an immutable `Plan` struct. The executor (cast/pipeline.go) wires stages together with zero branching on device type or codec — it just follows the plan.

**DLNA path** (the primary, well-tested path):
1. `pull` (source.go): One ffmpeg process remuxes the source into MPEG-TS on the spool (codec-copy, cheap). Paced at 2× realtime after a 90-second initial burst for VOD, 1× for live. Optionally tees mono PCM for whisper.
2. `spool` (spool/spool.go): Append-only disk buffer. Writes append, tails block at end-of-data via `sync.Cond` until more bytes arrive. Crucially, ffprobe can probe the growing spool locally — no upstream round-trip.
3. `gate` (gate.go): Blocks until whisper has a 10+ second transcription lead (or is done) AND the spool has data. Also detects upstream stalls via a 60-second progress watchdog.
4. Device discovery runs concurrently with pull+gate via `deviceFuture` — SSDP and connect can take seconds that short-lived signed URLs can't spare.
5. Local spool probe decides copy-vs-encode: if the source codec matches the renderer's capabilities and fits within the height cap, video is stream-copied; otherwise re-encoded through hardware encoder auto-detection (VA-API, VideoToolbox, or software libx264/libx265).
6. `replay` server (replay/server.go): HTTP server spooling the encoder output to disk and replaying it from byte 0 to every client. This is what survives Samsung's HEAD-probe → short-GET → real-GET dance.
7. DLNA device control via SOAP actions: SetAVTransportURI + Play.

**Chromecast path**: Simpler — can pass-through when the device accepts the source container. Otherwise remuxes (video copied, audio to AAC). Subtitles unsupported due to library limitation.

**Stream extraction** (source/extract/):
- Launches headless Chrome via chromedp
- Injects anti-detection JavaScript (17 scripts covering WebGL, canvas, font metrics, WebRTC, notifications, plugins, permissions, etc.)
- Collects stream URLs from CDP network events (requestWillBeSent + ExtraInfo for headers, responseReceived for confirmed MIME types)
- Also scrapes HLS .m3u8 URLs from console.log output
- Runs an action pipeline: click → navigate into largest iframe → bypass Turnstile → click fallback
- URLs are deduplicated, scored (master playlists preferred over variants), and sorted

**Stream resolution** (source/resolve/):
- Probes streams with ffprobe (concurrent, bounded by semaphore)
- For HLS masters: fetches the playlist, parses variants, picks highest-bandwidth within height cap
- Ranks streams: decoys (no video+audio, too short/ad insert) dropped; within-cap preferred; ties broken by bandwidth
- Per-host probe cap (maxProbePerHost=5) prevents rate-limiting on embed proxies

## Key Techniques

### 1. Read-Once Spool Pipeline
The single most important architectural decision. The upstream CDN is touched exactly once: a single ffmpeg process pulls the source, remuxes it codec-copy to MPEG-TS on disk, and optionally tees PCM audio. Everything downstream — encode, whisper, the replay server — reads local data. This means:
- CDN token expiration can't kill playback mid-movie
- Rate limits are irrelevant after the initial fetch
- The puller is paced like a buffering player (2× realtime, 90s initial burst), so the CDN sees a well-behaved client, not a ripper

The spool's blocking tail (spool.go:104-133) uses `sync.Cond.Wait()` to block readers at end-of-data until more bytes arrive — a Unix `tail -f` primitive in Go. Unlike a pipe, multiple independent readers can each start from byte 0.

### 2. async Device Discovery Overlapping the Pull
The device is discovered and connected concurrently with the upstream pull (pipeline.go:46). Device discovery (SSDP + connect + GetProtocolInfo) can take seconds — seconds that signed source URLs can't spare between extraction and first byte. The `deviceFuture` type uses a single-slot channel and errgroup integration: connect failure cancels the group cleanly; an unclaimed device is closed during teardown.

### 3. Live Subtitle Burn-in via drawtext + Atomic File Swap
The subtitle system has three concurrent goroutines:
1. The puller tees mono PCM (16kHz s16le) to whisper
2. whisper transcribes with LocalAgreement-2 streaming policy (commit only word prefixes confirmed by two consecutive hypotheses)
3. The encoder's `-progress` feed drives a cue writer that looks up the active subtitle line at `out_time + 1s` bias and atomically renames a temp file over the cue text file

ffmpeg's `drawtext` filter with `reload=1` re-opens the textfile before every frame. The atomic rename (write tmp, os.Rename) prevents ffmpeg from reading a partially-written file and crashing. This is a remarkably clean way to inject live text into a video pipeline — no IPC protocol, just a filesystem atomic swap.

The encoder is paced at 1.15× realtime via `-readrate` so the cue writer can keep up; the playback gate (gate.go) ensures whisper has a 10-second lead before encoding starts.

### 4. Hardware Encoder Auto-Detection
Rather than guessing from the platform, SelectEncoder (ffmpeg/encoder.go:73-87) runs a real one-frame test encode through each hardware encoder candidate (VA-API, VideoToolbox) and caches the result. A working GPU is proven once; a wedged one isn't retried per cast. The encoder registry is codec-ordered with hardware candidates ahead of software baselines — selection tries HEVC on GPU first, then H.264 on GPU, then software.

### 5. DLNA Capability Negotiation
Rather than assuming per-device-type capabilities, the DLNA path negotiates the renderer's actual capabilities at connect time via UPnP ConnectionManager `GetProtocolInfo` (dlna.go:128-151). The Sink protocolInfo CSV is parsed to extract supported codecs (AVC/HEVC from DLNA.ORG_PN tokens) and containers. The result drives the copy-vs-encode decision. Fallback to conservative H.264+MPEG-TS if negotiation fails.

### 6. Stream Collector Scoring and Dedup
Captured URLs from CDP events are scored (collector.go:298-324): master playlists get +100, "playlist" paths +50, variant/segment paths -50. When a master playlist is detected, the collection window exits immediately — no need to wait for individual variants. Duplicate URLs enrich existing entries (attach request headers from later sightings for hotlink-protected hosts).

### 7. Replay-from-Zero HTTP Server
The replay server (replay/server.go) spools the encoder's output and serves every HTTP client from byte 0 through an independent spool tail. This solves the Samsung-specific problem: the TV sends a HEAD, then a short GET (probe), then a real GET — without replay, the probe would consume the stream head and the real GET would join mid-stream at an undecodable byte offset. With replay, each connection gets its own copy from the start.

### 8. Playback Gate with Stall Detection
The gate (gate.go) blocks playback until the pipeline is ready, with a 60-second stall timeout. When the upstream stalls, it surfaces ffmpeg's stderr ring buffer — capturing "Server returned 404" patterns from expired HLS segments that would otherwise be hidden by context cancellation killing the ffmpeg process before it logs them.

## Design Decisions

**Optimized for DLNA, Chromecast is experimental**: The README is honest about this. The DLNA path has spool, replay server, capability negotiation, and subtitle support. The Chromecast path is a simpler direct-feed path with no subtitles (blocked by a library limitation in vishen/go-chromecast).

**Single upstream connection**: The puller remuxes the source exactly once. No retries, no parallel fetches. This is the right call for short-lived signed URLs — every second between extraction and first byte increases the chance of token expiry.

**Always remux to MPEG-TS for DLNA**: MPEG-TS is strictly append-only (no trailer), making it the right spool format. ffmpeg auto-inserts the correct bitstream filter (h264_mp4toannexb vs hevc_mp4toannexb) — no hardcoded assumptions.

**Paced like a player, not a downloader**: The puller reads at 2× realtime (VOD) or 1× (live). This is slower than possible, but it keeps CDN rate limits happy and the spool as a buffer rather than a full download. For VOD, the initial 90-second burst fills the spool quickly, then 2× pacing means the spool only grows — the encode always has data.

**Subtitles as hardsubs, not sidecar tracks**: Samsung renderers can't be trusted to display DLNA-delivered caption tracks, so subtitles are burned into the video via drawtext. The Chromecast path has no subtitles at all (library limitation).

**Config composition**: Each domain package owns its config type; the top-level config composes them (config/config.go:28). The cast package never imports app-level state — it takes a `cast.Config` that the caller composes from the application config. Clean dependency inversion.

**Go 1.26 requirement**: Notably uses Go 1.26 features (newer than the stable release at time of analysis), including `slices.Concat`, `slices.SortedFunc`, `cmp.Or`, and `iter.Seq` patterns.

## Comparison Notes

Unlike browser automation tools like [[Browser Use]] and [[Webwright]] which are general-purpose, Castor uses browser automation for a specific purpose: stream URL extraction. It's not an agent framework — it's a deterministic pipeline wrapped in a CLI.

Unlike [[surf-cli]] which provides general browser automation for agents, Castor's Chrome usage is tightly scoped: launch, navigate, capture CDP network events, run a fixed action pipeline, extract. The anti-detection suite (17 JS scripts) is unusually thorough for a non-scraping tool.

The transcription pipeline (whisper.cpp LocalAgreement-2 → cue shaping → drawtext burn-in) is a genuinely novel engineering achievement: real-time speech-to-subtitle burned into a live video transcode, all in-process, with atomic filesystem coordination between encoder and transcriber. Most tools that do live subtitling use sidecar protocols (WebVTT, SRT sidecars) — Castor burns them in because Samsung TVs don't reliably display DLNA caption tracks.

The spool-based read-once pipeline is a distinctive architectural choice. Most streaming tools either buffer entirely in memory (fragile) or serve directly from the upstream URL (token-expiry-risky). Castor's spool is an append-only disk buffer that decouples the CDN from playback — once data is spooled, the CDN can go away and playback continues.

## Size and Complexity

- ~10,640 lines of Go across 50+ source files
- ~1,400 lines of Go tests
- 17 injected JavaScript files for anti-detection
- Single developer (stupside), MIT licensed
- Active development: release-please CI, Homebrew cask, Docker image, canary releases
