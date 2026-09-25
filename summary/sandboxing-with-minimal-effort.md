---
url: https://yorickpeterse.com/articles/sandboxing-with-minimal-effort/
title: "Sandboxing with Minimal Effort"
author: Yorick Peterse
date_fetched: 2026-09-25
date_published: not stated in fetched text
topics:
  - security-and-sandboxing
  - software-engineering-craft
---

Yorick Peterse describes a new Inko language feature: an application-level sandboxing API that wraps OS primitives (Linux Landlock, macOS Seatbelt, FreeBSD Capsicum, OpenBSD pledge/unveil) behind one small cross-platform interface. His thesis is that containers are a good outer layer but shouldn't be the only layer — an application should be able to restrict its own capabilities regardless of how it's run.

He demonstrates with shost, his static file server: a Podman quadlet already drops capabilities and mounts everything read-only, yet the in-process sandbox took only a handful of lines — allow reading the TLS cert directory and sites directory, allow binding the TCP port, deny everything else. He argues security features earn adoption through ease of use, not capability, and that an API trivial enough to be used even when it seems redundant is the point.

Trade-offs are discussed honestly: FreeBSD is a no-op because Capsicum demands restructuring programs around pre-opened resources and `openat`, whereas Landlock and Seatbolt can be applied without changing program structure. He also notes platform differences (e.g. Landlock needing rules for the ELF interpreter).
