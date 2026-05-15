---
title: "Windows in Docker"
url: https://blog.fsck.com/releases/2026/03/11/windows-in-docker/
date_fetched: 2026-05-14
section: "Random"
---

# Headless Windows 11 in Docker Over SSH

Written by Claude (Opus 4.6) at Jesse Vincent's request. Documents running Windows 11 in Docker with KVM acceleration, accessible entirely through SSH. No RDP, VNC, or GUI.

Uses dockur/windows Docker image for unattended Windows install within QEMU/KVM container.

Major gotchas:
1. Networking: Must add --cap-add NET_ADMIN or QEMU defaults to broken user-mode networking
2. ISO Caching: Container wipes storage volume on init, causing repeated 15-minute ISO downloads. Fix: mount ISO directly as /boot.iso
3. OpenSSH: Install takes 10-15 minutes. Total first-boot ~25 minutes; subsequent boots ~2 minutes

Claude Code installation challenges:
- Node.js missing (winget broken due to certificate validation error, need manual MSI install)
- PATH issue (npm global packages not in system PATH that OpenSSH reads)
- PowerShell execution policy (Restricted by default, need RemoteSigned at LocalMachine scope)

PATH propagation problem: Interactive SSH sessions from remote machines get truncated PATHs. Fix: system-wide PowerShell profile that rebuilds PATH from registry on login.

PowerShell over SSH escaping: Layered escaping (bash -> SSH -> cmd -> PowerShell) is nearly impossible. Solution: pipe scripts via stdin using heredocs and -Command - flag.

"From docker run to claude --version working over SSH takes about 30 minutes for a fresh install."
