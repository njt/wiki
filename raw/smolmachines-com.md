---
url: https://smolmachines.com/
date_fetched: 2026-08-25
---

Run any workload in a fast, hardware-isolated Linux VM. The same SDK and the same `.smolmachine` artifact run identically:

Develop locally, then deploy to the hosted cloud — or self-host with smolvm, open source and 4.6k stars on GitHub.

```
# install (macOS + Linux)
curl -sSL https://smolmachines.com/install.sh | bash
# Windows x86_64: download the release zip (requires WHP)
# https://github.com/smol-machines/smolvm/releases
# for coding agents — install + discover all commands
curl -sSL https://smolmachines.com/install.sh | bash && smolvm --help
```
```
# run a command in an ephemeral VM (cleaned up after exit)
smolvm machine run --net --image alpine -- sh -c "echo 'Hello world from a microVM' && uname -a"
# interactive shell
smolvm machine run --net -it --image alpine -- /bin/sh
# inside the VM: apk add sl && sl && exit
# uninstall
curl -sSL https://smolmachines.com/install.sh | bash -s -- --uninstall
```
**sandbox untrusted code** — run untrusted programs in a hardware-isolated VM. Host filesystem, network, and credentials are separated by a hypervisor boundary.

```
# network is off by default — untrusted code can't phone home
smolvm machine run --image alpine -- ping -c 1 1.1.1.1
# fails — no network access
# lock down egress — only allow specific hosts
smolvm machine run --net --image alpine --allow-host registry.npmjs.org -- wget -q -O /dev/null https://registry.npmjs.org
```
**pack into portable executables** — turn any workload into a self-contained binary. All dependencies are pre-baked — no install step, no runtime downloads, boots in <200ms.

```
smolvm pack create --image python:3.12-alpine -o ./python312
./python312 run -- python3 --version
# Python 3.12.x — isolated, no pyenv/venv/conda needed
```
**persistent machines for development** — create, stop, start. Installed packages survive restarts.

```
smolvm machine create --net --name myvm
smolvm machine start --name myvm
smolvm machine exec --name myvm -- apk add sl
smolvm machine exec --name myvm -it -- /bin/sh
smolvm machine stop --name myvm
```
**GPU-accelerated workloads** — Vulkan access to the host GPU inside isolated VMs.

```
smolvm machine run --gpu --net --image fedora:42 -- bash -c '
  /usr/lib64/chromium-browser/headless_shell \
    --no-sandbox --screenshot=/tmp/shot.png \
    --window-size=1920,1080 https://example.com'
```
| smolvm | Containers | QEMU | Firecracker | |
|---|---|---|---|---|
| Isolation | VM per workload | Namespace (shared kernel) | Separate VM | Separate VM | 
| Boot time | <200ms | ~100ms | ~15-30s | <125ms | 
| Architecture | Library (libkrun) | Daemon | Process | Process | 
| GPU | Yes (Vulkan) | Host passthrough | VFIO | No | 
| macOS native | Yes | Via Docker VM | Yes | No | 
| Portable artifacts | .smolmachine | Images (need daemon) | No | No | 

Each workload gets real hardware isolation — its own kernel on Hypervisor.framework (macOS), KVM (Linux), or Windows Hypervisor Platform. Pack it into a `.smolmachine` and it runs anywhere the host architecture matches.

Defaults: 4 vCPUs, 8 GiB RAM. Memory is elastic via virtio balloon — the host only commits what the guest actually uses. libkrun VMM + custom kernel: libkrunfw. No daemon — the VMM is a library linked into the smolvm binary.

| host | guest | requirements | 
|---|---|---|
| macOS Apple Silicon | arm64 Linux | macOS 11+ | 
| macOS Intel | x86_64 Linux | macOS 11+ (untested) | 
| Linux x86_64 | x86_64 Linux | KVM (/dev/kvm) | 
| Linux aarch64 | aarch64 Linux | KVM (/dev/kvm) | 
| Windows x86_64 | x86_64 Linux | WHP enabled; download the release zip (not the curl installer) |
