---
title: "QuickEmu"
url: https://github.com/quickemu-project/quickemu
date_fetched: 2026-05-14
section: "Random"
---

# QuickEmu: Quick Virtual Machines

Wrapper for QEMU that "automatically does the right thing" when creating virtual machines.

## How It Works
Two main components:
1. **quickget**: Automatically downloads the upstream OS and creates the configuration
2. **quickemu**: Enumerates hardware and launches the VM with optimum configuration

## Supported Operating Systems
- macOS (Sequoia through Mojave)
- Windows 10, 11 (including TPM 2.0), Server 2016-2022
- Ubuntu and official flavors
- Nearly 1000 total OS editions
- BSD variants, FreeDOS, Haiku, ReactOS
- ARM64 guest VMs (native on ARM hosts, emulated on x86_64)

## Key Features
- Host support for Linux and macOS
- SPICE with host/guest clipboard sharing
- Multiple file-sharing protocols (VirtIO-webdavd, VirtIO-9p, Samba)
- QEMU Guest Agent integration
- USB and smartcard pass-through
- Automatic SSH port forwarding
- VirGL acceleration
- EFI/SecureBoot and Legacy BIOS boot
- Full-duplex audio and braille support
- No elevated permissions required
- Portable configurations
