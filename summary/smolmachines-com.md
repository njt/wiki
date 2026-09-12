---
url: https://smolmachines.com/
title: smolvm — Run Any Workload in a Hardware-Isolated Linux VM
author: smolmachines
site: smolmachines.com
date_fetched: 2026-08-25
date_published: unknown
topics:
  - security-and-sandboxing
---

# smolvm — Run Any Workload in a Hardware-Isolated Linux VM

smolvm is a single-binary, open-source (4.6k GitHub stars) tool for running workloads in fast, hardware-isolated Linux VMs. The core promise: the same SDK and the same portable `.smolmachine` artifact run identically in local development, on the hosted cloud, or self-hosted.

## What it does

- **Ephemeral VMs** — `smolvm machine run --net --image alpine -- sh -c "…"` boots a microVM, runs a command, and cleans up on exit. Interactive shells via `-it`.
- **Sandbox untrusted code** — host filesystem, network, and credentials are separated by a hypervisor boundary. Network is off by default; egress is allowlisted per host (`--allow-host registry.npmjs.org`).
- **Portable executables** — `smolvm pack create` turns any workload into a self-contained binary with all dependencies pre-baked; no install step, boots in <200ms.
- **Persistent machines** — `machine create / start / exec / stop`; installed packages survive restarts.
- **GPU workloads** — Vulkan access to the host GPU inside isolated VMs.

## How it's built

The isolation comes from a real hypervisor per workload: Hypervisor.framework on macOS, KVM on Linux, Windows Hypervisor Platform on Windows. The VMM is **libkrun** — a *library* linked into the smolvm binary, not a daemon — paired with a custom kernel, **libkrunfw**. Defaults are 4 vCPUs and 8 GiB RAM; memory is elastic via virtio balloon, so the host commits only what the guest uses.

## Comparison (from the site's own table)

Against containers, QEMU, and Firecracker, smolvm claims: VM-per-workload isolation (unlike containers' shared kernel), <200ms boot (vs. QEMU's 15–30s), library architecture (vs. daemon/process), Vulkan GPU access (Firecracker has none), macOS-native support (containers need a Docker VM, Firecracker doesn't run on macOS), and portable `.smolmachine` artifacts that don't need a daemon to run.

## Host/guest matrix

macOS Apple Silicon → arm64 Linux; macOS Intel → x86_64 Linux (marked "untested"); Linux x86_64/aarch64 → same-arch Linux via KVM; Windows x86_64 → x86_64 Linux via WHP (release zip, not the curl installer).
