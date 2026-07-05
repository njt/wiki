---
url: https://github.com/quickemu-project/quickemu
date_fetched: 2026-07-05
backfilled: true
---

**Quickemu** is a wrapper for the excellent QEMU that
automatically *"does the right thing"* when creating virtual machines. No
requirement for exhaustive configuration options. You decide what operating
system you want to run and Quickemu takes care of the rest 🤖

- `quickget`- **automatically downloads the upstream OS**and creates the configuration 📀
- `quickemu`enumerates your hardware and launches the virtual machine with the- **optimum configuration best suited to your computer**⚡️

The original objective of the project was to enable quick testing of Linux distributions where the virtual machines and their configuration can be stored anywhere (such as external USB storage or your home directory) and no elevated permissions are required to run the virtual machines.

**Today, Quickemu includes comprehensive support for macOS,
Windows**, most of the BSDs, novel non-Linux operating systems such as FreeDOS, Haiku, KolibriOS, OpenIndiana, ReactOS, and more.

- Host support for **Linux and macOS**
- **macOS**Sequoia, Sonoma, Ventura, Monterey, Big Sur, Catalina & Mojave
- **Windows**10 and 11 including TPM 2.0
- **Windows Server**2022 2019 2016
- **ARM64 guest support**for running aarch64 VMs (native on ARM hosts, emulated on x86_64)
- Ubuntu and all the **official Ubuntu flavours**
- **Nearly 1000 operating system editions are supported!**
- Full SPICE support including host/guest clipboard sharing
- VirtIO-webdavd file sharing for Linux and Windows guests
- VirtIO-9p file sharing for Linux and macOS guests
- QEMU Guest Agent support; provides access to a system-level agent via standard QMP commands
- Samba file sharing for Linux, macOS and Windows guests (*if*)`smbd`is installed on the host
- VirGL acceleration
- USB device pass-through
- Smartcard pass-through
- Automatic SSH port forwarding to guests
- Network port forwarding
- Full duplex audio
- Braille support
- EFI (with or without SecureBoot) and Legacy BIOS boot

## As featured on Linux Matters podcast!

The presenters of Linux Matters 🐧🎙️ are the creators of each of the principal Quickemu projects. We discussed Quickemu's 2024 reboot in Episode 30 - Quickemu Rising From the Bashes.

Once Quickemu is installed, there are two simple steps to create and run a virtual machine:

- `quickget`automatically downloads the ISO image for the operating system you want to run and creates a configuration file for the virtual machine.

`quickget nixos unstable minimal`- `quickemu`starts the virtual machine using the configuration file created by- `quickget`.

`quickemu --vm nixos-unstable-minimal.conf`Execute `quickget` (with no arguments) to see a list of all the supported operating systems.

The wiki describes how to get up and running with Quickemu and also covers more advanced configuration and usage.
