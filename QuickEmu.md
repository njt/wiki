# QuickEmu

A wrapper for QEMU that "automatically does the right thing" when creating virtual machines. Downloads the OS, detects your hardware, configures the VM optimally. Nearly 1,000 supported OS editions including macOS, Windows, and Linux.

---

## Key Quotes

> "Automatically does the right thing when creating virtual machines."

## Key Themes

#tool #virtualization #devops #testing

QEMU is powerful but hostile to configure. QuickEmu's value is entirely in the abstraction: `quickget` downloads the right OS image, `quickemu` detects your hardware and generates an optimal configuration. No manual RAM allocation, no display driver selection, no storage format decisions. It just works.

The breadth of OS support is impressive -- macOS guests (Sequoia through Mojave), Windows 10/11 with TPM 2.0, Windows Server, nearly every Linux distribution, BSD variants, FreeDOS, Haiku, ReactOS. ARM64 guest support (native on ARM hosts, emulated on x86_64) covers the Apple Silicon case.

The "no elevated permissions" and portable configuration design means VMs live as simple config files you can share, version, and move between machines.

## Critical Analysis

QuickEmu solves the "I just need a quick VM to test something" problem better than anything else. It doesn't compete with VMware or Parallels for daily-driver VMs -- it competes with "I spent 45 minutes reading QEMU docs and gave up."

The SPICE clipboard sharing, VirtIO file sharing, and automatic SSH port forwarding are the quality-of-life features that separate "technically works" from "actually useful." You can copy-paste between host and guest, share files, and SSH in without manual network configuration.

The limitation is performance. QEMU with software emulation is slower than native hypervisors (Hyper-V, Apple Hypervisor Framework). QuickEmu uses VirGL acceleration where available, but a macOS guest on a Linux host will never match Parallels-on-Mac performance. The use case is testing and exploration, not production workloads.

---
*Sources: [[raw/quickemu]]*
*Last updated: 2026-05-14*
