---
url: https://devblogs.microsoft.com/commandline/wslc-architecture-deep-dive/
title: "WSLC Architecture Deep Dive"
author: Microsoft Command Line team
date_fetched: 2026-10-03
date_published: 2026-09 (approx, on WSLC general availability)
topics:
  - developer-tools
  - security-and-sandboxing
---

Microsoft's architecture deep dive on WSLC (WSL Containers), published alongside its general availability. The post covers three architectural changes that distinguish WSLC from WSL itself: a new session model, a storage design, and the "Consommé" networking mode.

**Session model.** Like WSL, client processes call into the privileged `wslservice.exe` Windows service, which creates virtual machines via HCS. The key difference: wslservice does not retain ownership of the VM. It instead spawns a child process, `wslcsession.exe`, running as the calling user, which performs all session operations (creating containers, mounting directories, binding ports). This gives strong isolation between sessions (separate processes) and tighter security boundaries, since session operations run in a less privileged process than the service.

**Storage.** Each session gets its own VHD (under `%AppData%\Local\wslc\sessions`) storing session state — images, containers, networks, volumes. Container volumes share Windows paths into containers: implemented as virtiofs shares mounted under `/mnt` inside the Linux VM, then bind-mounted into the container. virtiofs is about twice as fast as the plan9 filesystem WSL historically used. VHD-backed volumes offer a native Linux filesystem (ext4) and size limits, mounted by name across containers.

**Networking (Consommé).** A new networking model: all Linux VM traffic is sent as ethernet frames to a virtio queue read by a Windows process running as the user. That process provides DNS, TCP/UDP routing, and port mapping. Because egress traffic is emitted on behalf of the session's user, it flows like a regular Windows process — giving extensive compatibility with VPNs and firewalls.

The post is open-source–linked (microsoft/WSL) and closes with reader comments noting compose-file support is the top feature request, deferred to future iterations.
