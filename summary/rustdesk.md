---
url: https://github.com/rustdesk/rustdesk
title: "RustDesk — Open Source Remote Desktop"
author: "rustdesk (Purslane Tech Pte. Ltd.)"
date_fetched: 2026-07-25
date_published: 2021
---

RustDesk is an open-source, self-hostable remote desktop application — the leading open alternative to TeamViewer and AnyDesk, with ~80K GitHub stars. The core is written in Rust (~23K lines), with a Flutter/Dart UI frontend linked via `flutter_rust_bridge`. It supports Windows, macOS, Linux, Android, iOS, and web.

The network architecture has three tiers: a **rendezvous server** (hbbs) for device registration and NAT-introduction, a **relay server** (hbbr) as a fallback when direct P2P fails, and **direct peer-to-peer** as the preferred path. Connections race TCP, UDP, IPv6, and relay simultaneously via `select_ok` to minimise latency. All traffic is end-to-end encrypted with NaCl/libsodium: Ed25519 long-term identity keys plus per-session ephemeral Curve25519 for forward secrecy. The wire protocol uses Protocol Buffers throughout.

Notable design choices include an adaptive audio buffer that monitors occupancy over 12-second windows to compensate for jitter without adding excessive latency, dynamic NAT-timeout computation based on detected NAT types, and a plugin framework (compile-time opt-in) that can intercept input events. The release profile favours binary size over compilation speed with LTO, single codegen unit, and symbol stripping.

The project maintains its own forks of several dependencies (scrap, enigo, clipboard, and others) rather than using upstream crates, trading maintenance burden for platform-specific customisation. The original Sciter-based UI is deprecated but still present in the source tree.
