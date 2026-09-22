---
url: https://gist.github.com/obra/4fc6e77c99df6bd2c8e19f68576b4e0b
title: "windows-vm (Claude Code command)"
author: Jesse Vincent (obra)
date_fetched: 2026-09-22
date_published: 2026-03 or earlier (underlies the 2026-03-11 blog post "Windows in Docker")
topics:
  - claude-code
  - developer-tools
---

# windows-vm (Claude Code command)

The raw Claude Code command definition for managing a headless Windows 11 VM running in Docker, written by Jesse Vincent (obra). The frontmatter declares a command named `windows-vm` with argument-hint `[create|start|stop|restart|ssh|status]` and allowed-tools `Bash, Read, Write`; the body is the procedure the agent follows when the user asks to spin up, stop, restart, or SSH into a Windows VM.

The VM runs via the `dockurr/windows` image with KVM acceleration (`--device /dev/kvm`, `--cap-add NET_ADMIN`) — 8GB RAM, 4 cores, 64GB disk, credentials user/password. Access is SSH-only on `localhost:2222`, with an RDP fallback on 3389 and a VNC-in-browser console on 8006 for debugging; all ports are bound to 127.0.0.1 only, with remote access via Tailscale (`ssh -J jesse@magic-kingdom -p 2222 user@localhost`). A directory layout outside the container does the caching: the 7.3GB Windows ISO lives in `iso/` and is mounted as `/boot.iso`, because dockur wipes its `/storage` volume on every recreate; an `oem/install.bat` runs at the end of Windows OOBE to install OpenSSH Server.

The create flow ends with a stdin-piped PowerShell script (`powershell -ExecutionPolicy Bypass -Command -`, to dodge what the author calls "PowerShell escaping hell") that silently installs Node.js from an MSI, runs `npm install -g @anthropic-ai/claude-code`, adds the npm global bin directory and Git to the **system** PATH (sshd ignores the user PATH), sets `RemoteSigned` execution policy at `LocalMachine` scope, writes an all-users PowerShell profile that rebuilds `$env:Path` from the registry on every SSH login, and restarts sshd. Fresh install takes 20–30 minutes; warm boots from the existing disk image take ~2 minutes.

The closing gotchas list is the hard-won payload: `irm https://claude.ai/install.ps1 | iex` reports success but `claude` won't work without Node; winget fails on Microsoft Store certificate validation inside the VM; each recreated VM rotates SSH host keys (`ssh-keygen -R '[localhost]:2222'`); and interactive SSH sessions don't inherit the full system PATH. Screenshot debugging is done by asking the QEMU monitor over `nc localhost 7100` for a screendump and converting the PPM with ImageMagick.
