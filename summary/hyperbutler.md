---
url: https://github.com/Higangssh/hyperbutler
title: "hyperbutler"
author: Higangssh and contributors
date_fetched: 2026-11-06
date_published: 2026 (project; v0.1.1 current)
topics:
  - developer-tools
  - security-and-sandboxing
---

hyperbutler is an open-source, Docker-compatible container engine for Windows 11 that deliberately skips WSL 2. It boots a small Alpine Linux VM through the Windows Host Compute Service (the same API WSL uses), runs real dockerd and containerd inside it on a read-only, hash-pinned system disk, and lets the stock `docker` CLI, Compose, SDKs and VS Code talk to it through a docker context. No WSL, no Docker Desktop, no admin rights after one MSI install.

Its headline numbers, measured on one laptop against Docker Desktop 4.97: 120 MB download vs 592 MB, 319 MiB idle memory vs 1.7–2.7 GB, 2.5 s to a running container vs 5.6 s, and — its most distinctive feature — deleted disk space comes back automatically thanks to a sparse data disk and a host-side "spaceguard". Bind mounts from `C:\` run through its own FUSE-over-hvsock file server (2000 small file reads in 0.4 s vs Docker Desktop's 3.4 s) and fire inotify watchers in milliseconds, which Vite/webpack rebuilds need.

The architecture splits privilege carefully: a per-user engine process (`hyperbutler.exe`) relays the Docker API, runs a gVisor-based user-mode network stack, and serves bind mounts; a tiny broker Windows service is the only privileged component, exposing exactly seven typed requests over a named pipe so users never need Hyper-V Administrators membership. Two agent-facing pieces stand out: a built-in MCP server with a strict allowlist security model (read tools by default, writes behind flags, every untrusted string quarantined in an `untrustedText` field), and a proposed per-session VM sandbox for AI coding agents whose threat model is prompt injection, fought with egress control rather than filesystem isolation. Apache-2.0, no telemetry, written in Go.
