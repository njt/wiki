# Windows in Docker

Jesse Vincent's guide to running headless Windows 11 in Docker with KVM acceleration, accessible entirely through SSH. No RDP, no VNC, no GUI at any point. The goal: spin up Windows, get an SSH shell, run Claude Code, tear it down when done. Written by Claude (Opus 4.6) at Jesse's request, which is a delightful meta-touch.

---

## Key Quotes

> "From docker run to claude --version working over SSH takes about 30 minutes for a fresh install."

## Key Themes

#tool #docker #windows #agent-testing #infrastructure

This solves a real problem: testing Windows-specific tools and behaviors from a Linux-only infrastructure. The dockur/windows Docker image handles unattended Windows installation within QEMU/KVM, but the article's value is in documenting every gotcha along the way.

The gotchas are educational: --cap-add NET_ADMIN for networking, mounting the ISO directly as /boot.iso to avoid repeated 15-minute downloads, the OpenSSH installation taking 10-15 minutes, npm global packages not being in the PATH that OpenSSH reads, PowerShell execution policy blocking scripts. Each of these is a half-day of debugging if you hit it blind.

The layered escaping problem (bash -> SSH -> cmd -> PowerShell) is a universal pain point with remote Windows administration. The stdin/heredoc solution is the right pattern.

## Critical Analysis

30 minutes for a fresh install is acceptable; 2 minutes for subsequent boots makes this practical for CI/CD. The real question is cost vs. convenience -- if you need Windows testing regularly, this is worth automating. If it's occasional, the 30-minute cold start is painful.

The fact that winget was broken due to certificate validation errors on a fresh Windows 11 install is a damning commentary on the Windows developer experience. You can't even install Node.js through the official package manager without manual intervention.

This pairs well with [[floci]] for local AWS testing -- together they give you the full spectrum of "simulate production environments locally for agent testing."

The inverse approach — running Linux containers natively on Windows without Docker Desktop — is Microsoft's [[WSL Containers]], currently in public preview. Where this page solves "I need Windows on my Linux box," WSL Containers solves "I need Linux containers on my Windows box without installing third-party tooling." The two together cover the full cross-platform container matrix.

See also [[Awesome Vibez]] -- Jesse Vincent is one of the most prolific builders in that community, and this came from his practical needs running Superpowers.

---
*Sources: [[summary/windows-in-docker]]*
*Last updated: 2026-05-14*
