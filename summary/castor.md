---
url: https://github.com/stupside/castor
title: "Castor"
author: stupside
date_fetched: 2026-07-25
---

Castor is a Go CLI (~10,640 lines, MIT) that casts web video to smart TVs at
full quality. Written by a single developer (stupside), it extracts the real
video stream URL from a web page via headless Chrome, transcodes it for the
target device, and serves it over the local network.

The core architecture is a **read-once spool pipeline**: a single ffmpeg process
pulls the source, remuxes it codec-copy to MPEG-TS on disk, and optionally tees
PCM audio for transcription. Everything downstream — encoding, whisper
transcription, the HTTP replay server — reads from local disk. This decouples
the upstream CDN (and its short-lived signed URLs) from playback: once data is
spooled, the CDN can expire without killing the stream.

The pipeline uses a **planner/executor split**: all decisions about device type
and source properties are made up front into an immutable Plan, and the executor
follows it with no branching. Device discovery (SSDP + DLNA capability
negotiation) runs concurrently with the upstream pull so signed URLs don't
expire during the connection dance. The DLNA path is the primary, well-tested
path; Chromecast support is simpler and experimental.

Notable techniques include:
- **Live subtitle burn-in** via whisper.cpp (LocalAgreement-2 streaming policy)
  coordinated with ffmpeg's drawtext filter through atomic filesystem swaps —
  no IPC protocol, just `os.Rename`.
- **Hardware encoder auto-detection** that runs a real one-frame test encode
  through each GPU candidate (VA-API, VideoToolbox) rather than guessing from
  the platform.
- **Replay-from-zero HTTP server** that serves every TV client from byte 0,
  solving Samsung's HEAD → short-GET → real-GET probe dance that would otherwise
  consume the stream head.
- **Stream extraction** via chromedp with 17 injected anti-detection JS scripts
  (WebGL, canvas, font metrics, WebRTC spoofing) and Cloudflare Turnstile
  bypass.
- **Playback gate** with 60-second stall detection that surfaces ffmpeg's stderr
  ring buffer when the upstream dies, catching hidden CDN errors.

The puller is paced like a player (2× realtime for VOD, 1× for live) rather
than a downloader, so the CDN sees a well-behaved client. Requires Chrome,
ffmpeg 7.1+, and ffprobe 7.1+ on the host.
