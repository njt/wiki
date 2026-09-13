---
url: https://github.com/peetzweg/opendisplay
title: "OpenDisplay"
author: Philip Poloczek
date_fetched: 2026-09-13
date_published: 2026-09-01
topics:
  - developer-tools
  - distributed-systems
---

OpenDisplay (GPL-3.0, by Philip Poloczek) turns a spare iPhone, iPad, or Mac into a true extended second monitor for a Mac — free, self-hosted, no subscription, no dongle, no Apple-ID pairing. Where Sidecar is iPad-only and same-Apple-ID, Duet went subscription, and Luna needs hardware, OpenDisplay is the missing option: two small native apps (a Mac sender and an iOS/Mac receiver, ~4 Swift files per platform), one direct TCP connection, and nothing in between.

The architecture is a deliberately simple pipeline: a virtual display created through the *private* `CGVirtualDisplay` CoreGraphics API (the same reverse-engineered interface DeskPad and BetterDisplay use — the reason the app can't ship on the Mac App Store), captured with ScreenCaptureKit, hardware-encoded to H.264 via VideoToolbox (real-time mode, no B-frames, no periodic keyframes), and streamed as length-prefixed Annex B frames over TCP. Touch, scroll, and Apple Pencil input travel back as JSON control messages and are injected via `CGEvent`. The receiver listens on port 9000 and the sender connects — that single role inversion is what makes USB and WiFi one code path: over a cable the sender dials through macOS's built-in `usbmuxd` daemon (a native plist-protocol client replaces the usual external `iproxy` tool), over the air it dials a Bonjour-advertised address. Latency is managed by two-stage frame-drop backpressure (skip the capture while an encode is in flight; skip again if three sends are still un-ACKed), a local-cursor echo that hides the cursor from the video and ships its position on a control channel (optionally a UDP side channel with sequence numbers, to escape TCP head-of-line blocking), and NTP-style clock sync so the receiver can display true end-to-end latency.

What elevates the project above a weekend hack is the protocol engineering. `PROTOCOL.md` is a genuine normative wire spec (RFC 2119 key words, a version handshake where absence means protocol 1, additive changes free, breaking changes two-phase), published so third-party receivers exist for Android, iOS 12, and Linux Wayland senders. `COMPATIBILITY.md` documents the harder policy problem: Mac and iOS apps update on different schedules (Sparkle hours vs App Store weeks), so each side must be able to detect that the *other* is too old. The code is dense with field-hardening: poisoned virtual-display identities are escaped by bumping serial numbers, HiDPI mode and mirror state are continuously re-enforced because macOS restores saved display state asynchronously, rotation reuses the same virtual monitor via a mode switch so windows aren't redistributed, and pulling the host-to-host cable is treated as deliberate disconnect intent.

Rough edges are acknowledged in the code itself: the sender-to-receiver channel demux is a heuristic (JSON iff payload <32 KB, starts with `{`, contains no NUL) that the spec explicitly labels design debt slated for a typed frame header in protocol 4, and the WiFi transport is currently unencrypted (pairing-code encryption is roadmap item #16).
