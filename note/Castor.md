# Castor

A Go CLI that casts web video to smart TVs at full quality by extracting the real video stream URL from a page, transcoding it for the target device, and serving it over the local network. Written by stupside, MIT licensed, ~10,600 lines of Go.

Smart TVs won't cast arbitrary web video, and screen mirroring is laggy and drops resolution. Castor solves this by finding the real stream, processing it through a purpose-built pipeline, and casting it directly — no catalog, no bundled content, no DRM circumvention.

---

## Architecture

Single-binary CLI built as a deterministic pipeline, not an agent or service. The flow is `extract → resolve → plan → execute`:

**Stream extraction** (`internal/source/extract/`): Launches headless Chrome via the Chrome DevTools Protocol (chromedp), injects 17 anti-detection JavaScript scripts, and captures video stream URLs from network events. An action pipeline (click → navigate into largest iframe → bypass Cloudflare Turnstile → click fallback) starts playback so stream requests fire. URLs are collected, deduplicated, and scored — master HLS playlists preferred over individual variants. MIME-confirmed streams take precedence over URL-pattern matches.

**Stream resolution** (`internal/source/resolve/`): Probes captured streams concurrently with ffprobe (semaphore-bounded). For HLS masters, fetches the playlist and selects the highest-bandwidth variant within the configured height cap. Ranks streams by bandwidth and probes — decoys (no video+audio, ad inserts too short to be real content) are dropped hard; within-height-cap candidates preferred over above-cap; remaining ties broken by bandwidth. Per-host probe cap (5) prevents rate-limiting on embed proxies.

**Cast pipeline** (`internal/cast/`): A planner/executor split. `BuildPlan` (plan.go) makes every decision up front based on device type (DLNA/Chromecast) and source properties, producing an immutable `Plan`. The executor wires stages together with zero branching.

The DLNA path (the primary, well-tested path):
1. **Pull** (source.go): One ffmpeg process remuxes the source codec-copy to MPEG-TS on disk via a spool, paced at 2× realtime after a 90s initial burst. Optionally tees mono PCM audio for transcription.
2. **Device discovery** runs concurrently with the pull — SSDP + connect can take seconds that short-lived signed source URLs can't spare. A `deviceFuture` (single-slot channel + errgroup) starts connecting immediately; the result is awaited only when needed.
3. **Gate** (gate.go): Blocks until whisper has a 10-second transcription lead AND the spool has data. Detects upstream stalls via a 60-second progress watchdog, surfacing ffmpeg's stderr ring buffer to name the failure.
4. **Copy-vs-encode decision**: The growing spool is probed locally (no upstream round-trip). If the source codec matches the renderer's negotiated capabilities and fits within the height cap, video is stream-copied. Otherwise re-encoded through hardware encoder auto-detection.
5. **Transcode + replay server**: The encoder reads the spool via a blocking tail, burns subtitles into frames via drawtext (if enabled), and pipes output to an HTTP replay server.
6. **Replay server** (replay/server.go): Spools the encoder output and serves every HTTP client from byte 0 through an independent spool tail. This survives Samsung's HEAD-probe → short-GET → real-GET dance — each GET gets its own copy of the stream from the start.
7. **DLNA control**: SetAVTransportURI + Play via UPnP SOAP actions against the renderer's AVTransport service.

The Chromecast path is simpler — pass-through when the device accepts the container, otherwise remux (video copied, audio to AAC). No subtitle support (blocked by a library limitation in vishen/go-chromecast).

**Interactive browser** (`internal/browse/`): A Bubble Tea TUI for searching TMDB, browsing curated feeds, drilling into TV seasons/episodes, and casting with one keystroke. Has poster rendering via half-block Unicode characters (pixterm) and genre/discover filtering. Returns a `Selection` to the caller — the TUI owns only the browse experience, not the cast pipeline.

## Key Techniques

### Read-Once Spool Pipeline
The single most important architectural decision. The upstream CDN is touched exactly once: a single ffmpeg process pulls the source into an append-only on-disk spool. Everything downstream reads local data. CDN token expiration can't kill playback mid-movie; rate limits are irrelevant after the initial fetch. The spool (`internal/cast/spool/spool.go`) uses `sync.Cond.Wait()` to implement blocking tails — readers block at end-of-data until more bytes arrive, like Unix `tail -f` in Go. Multiple independent readers can each start from byte 0.

### Live Subtitle Burn-in via Atomic File Swap
Three concurrent goroutines coordinate through the filesystem: the puller tees PCM to whisper for transcription with LocalAgreement-2 streaming policy (commit only word prefixes two consecutive hypotheses agree on); the encoder's `-progress` feed drives a cue writer that looks up the active subtitle line and atomically renames a temp file over the cue text file; ffmpeg's `drawtext` filter with `reload=1` re-opens the file before every frame. The atomic rename prevents ffmpeg from reading a partially-written file and crashing — a remarkably clean way to inject live text into a video pipeline with no IPC protocol, just filesystem operations.

### Hardware Encoder Auto-Detection
Rather than guessing from the platform, `SelectEncoder` (`internal/cast/ffmpeg/encoder.go`) runs a real one-frame test encode through each hardware encoder (VA-API, VideoToolbox) and caches the result. A working GPU is proven once per process; a wedged one isn't retried. The encoder registry is codec-ordered with hardware candidates ahead of software baselines, all contributing their own init args, filters, and flags — `EncodeArgs` splices them in verbatim with no per-encoder branching.

### DLNA Capability Negotiation
The DLNA path negotiates the renderer's actual capabilities at connect time via UPnP ConnectionManager `GetProtocolInfo`, parsing the Sink protocolInfo CSV for supported codecs (AVC/HEVC from DLNA.ORG_PN tokens) and containers. This drives the copy-vs-encode decision. Falls back to conservative H.264+MPEG-TS if negotiation fails.

### Subtitle Cue Shaping
The `cue` package (`internal/cast/cue/cue.go`) folds a stream of committed timed words into display cues: groups words into readable lines, coalesces staccato one-word sentences that would otherwise flash, cuts at natural clause boundaries when a line would overrun character or duration budgets, and trims cue edges inward to compensate for whisper's over-reported timestamps (early onsets, late offsets). Broadcast-subtitle-grade output with no dependency on the transcription backend — the cue package knows nothing about whisper, ffmpeg, or files.

### Replay-from-Zero HTTP Server
The replay server spools the encoder's output and serves every HTTP client from byte 0 through an independent spool tail. This solves the Samsung TV problem: the TV sends HEAD → short GET (probe) → real GET; without replay, the probe GET consumes the stream head and the real GET joins mid-stream at an undecodable byte offset. With replay, each connection gets its own copy from the start. The server uses a 30-second idle grace window after the stream is fully produced, allowing renderer hiccups and reconnects.

## Design Decisions

**Optimized for DLNA, Chromecast experimental**: The DLNA path has spool, replay server, capability negotiation, and subtitle support. The Chromecast path is simpler with no subtitles. The README is honest about this asymmetry.

**Single upstream connection, paced like a player**: The puller remuxes at 2× realtime (VOD) or 1× (live). Slower than possible, but CDNs see a well-behaved client rather than a ripper. No retries — signed URLs are too short-lived for retry logic to help.

**Always MPEG-TS for DLNA**: MPEG-TS is strictly append-only (no trailer seek-back), making it the right spool format. ffmpeg auto-inserts the correct bitstream filter for the actual codec — no hardcoded h264 assumption.

**Subtitles as hardsubs**: Samsung TVs don't reliably display DLNA-delivered caption tracks, so subtitles are burned into the video. The Chromecast path has no subtitles (library limitation in the go-chromecast library — doesn't expose tracks on MediaItem).

**Config composition pattern**: Each domain package owns its config type; the top-level config composes them. The cast package never imports app-level state — clean dependency inversion.

**Whisper init failure degrades gracefully**: If whisper fails to initialize, the cast proceeds without subtitles rather than blocking playback entirely. The PCM drain goroutine ensures backpressure doesn't stall the puller.

**Per-host stream probe cap**: Extractors can return many candidates from one proxy host (master + long tail of variants). Probing all of them trips rate limiters. The `maxProbePerHost=5` cap keeps the master and drops the redundant tail.

**Go 1.26**: Uses `slices.Concat`, `slices.SortedFunc`, `cmp.Or`, `iter.Seq` patterns — notably forward-leaning for a tool targeting end-user machines.

## Comparison Notes

Castor is a practical tool, not an AI agent or framework. Unlike [[Browser Use]] and [[Webwright]] which provide general browser automation for agents, Castor uses browser automation for a single purpose (stream URL extraction). Unlike general-purpose casting tools, it doesn't mirror the screen or re-encode the display — it finds and casts the real stream, which means full quality and low latency.

The transcription pipeline is distinctive: most tools that do live subtitling use sidecar protocols (WebVTT, SRT files). Castor burns subtitles into the video stream because Samsung TVs don't reliably display DLNA caption tracks. The architectural consequence — a paced encoder, atomic text file swaps, and a cue writer synchronised to the encoder's progress feed — is genuinely novel.

The read-once spool pattern is unusual for a streaming tool. Most either buffer entirely in memory or serve directly from the upstream URL. Castor's spool decouples CDN from playback, meaning once data is on disk, the CDN can go away and playback continues. This is the right trade-off for sources with short-lived signed URLs.

The interactive TUI (`castor cast`) shows mature product thinking: half-block poster rendering, async asset loading, genre filtering, discover mode with sort controls, and a TV seasons → episodes drilldown — all without touching the cast pipeline. The TUI returns a `Selection`; the caller hands it off. Clean separation of concerns.

Compared to the broader wiki's agent-design landscape, Castor is a reminder that not everything needs to be an agent. It's a deterministic pipeline wrapped in a CLI, solving a specific real-world problem with thoughtful engineering.

---
*Sources: [[raw/castor]]*
*Last updated: 2026-07-25*
