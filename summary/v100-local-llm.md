---
url: https://blog.tymscar.com/posts/v100localllm/
title: "I Put a Datacenter GPU in My Gaming PC for £200"
author: Oscar Molnar (Tymscar)
date_fetched: 2026-06-02
date_published: 2026-05-30
tags: homelab, gpu, nixos, local-llm, hardware, debugging
topics:
  - local-and-open-source-inference
---

# I Put a Datacenter GPU in My Gaming PC for £200

Author: Oscar Molnar (blog: Tymscar)
Published: 2026-05-30
Read time: 13 min (2,710 words)

## Summary

Molnar documents installing a used NVIDIA Tesla V100 SXM2 (16GB HBM2) datacenter GPU alongside his existing RTX 4080 to achieve 32GB total VRAM for local LLM inference, all for roughly £200.

## Key Hardware Details

**The GPU:** Tesla V100 SXM2 16GB — a Volta architecture card with 5,120 CUDA cores, 900 GB/s memory bandwidth over a 4096-bit HBM2 bus. Purchased on eBay for ~£150.

**Bandwidth comparison (article's data):**
- V100: 900 GB/s (2017)
- RTX 4080: 736 GB/s
- M3 Max: 400 GB/s | M4 Max: 546 GB/s | M5 Max: 614 GB/s
- RX 7900 XTX: 960 GB/s (~£700+)
- RTX 5090: 1,792 GB/s (~£2,000+)

**The Adapter:** An SXM2-to-PCIe adapter PCB (~£50) that allows the proprietary-connector card to slot into a standard motherboard.

## The Fan Issue

The card's fan measured **82 dB** — described as between "a garbage disposal and a lawnmower." It runs at 100% uncontrollably via software. The author traced the pinout using jumper wires and a 9V battery, confirmed it's a standard PWM fan on a JST PH2.0 connector, then wired it to a motherboard fan header via a £2 adapter cable. At 10% PWM, "it never goes above 50C even at full load."

## Software Configuration (NixOS)

Critical constraints: NVIDIA dropped Volta support in driver branch 560, so the author uses **branch 550.x** (`legacy_535`), which requires **kernel 6.6** and only supports **CUDA 12.2** (backported from nixpkgs 24.05). A quirk: `services.xserver.enable = true` is required even for a headless setup, or NVIDIA kernel modules won't load.

### Driver/kernel config

```
boot.kernelPackages = pkgs.linuxPackages_6_6;
hardware.nvidia.package = config.boot.kernelPackages.nvidiaPackages.legacy_535;
services.xserver.enable = true;
services.xserver.videoDrivers = [ "nvidia" ];
```

### CUDA 12.2 overlay

```
nixpkgs.overlays = [
  (final: prev: {
    cudaPackages_12_2 = nixpkgs-cuda.legacyPackages.${prev.system}.cudaPackages_12_2;
  })
];
```

### NFS mount for models

```
fileSystems."/mnt/nas" = {
  device = "truenas-nfs.tymscar.com:/mnt/oasis/services";
  fsType = "nfs";
  options = [ "nfsvers=4" "_netdev" "auto" "nofail" ];
};
```

### Llama.cpp vision flags

```
--mmproj /mnt/nas/llamacpp/mmproj-F16.gguf --mmproj-offload
```

The full machine definition is linked: [GitHub dotfiles commit](https://github.com/tymscar/dotfiles/commit/9f3d647884c498d0b98b55ffcfa50dd806aed146).

## Running the Model

| Setting | Value |
|---|---|
| Model | Qwen3.6-27B-MTP Q5_K_M (~19GB) |
| Context | 128k tokens |
| GPU layers | 99 (all offloaded) |
| Tensor split | `-ts 1.0,1.0` (even across both GPUs) |
| Inference speed | ~32 tok/s |
| Prompt processing | ~133–160 tok/s |

With Multi-Token Prediction (MTP), speed can reach 50–60 tok/s on predictable output like code. The author had to compile llama.cpp from source at a specific commit for MTP support.

## Model Quality Claim

Molnar asserts the Qwen3.6-27B "ties with Claude Sonnet 4.6 on Artificial Analysis's Agentic Index" and beats Sonnet 4.6 on MMMU-Pro and Terminal-Bench 2.0, while trailing on GPQA and SWE-Bench Verified. His conclusion: "the model you run in your bedroom is in the same conversation as the ones that charge you per token."

## Vision Support

The Qwen3.6-27B model accepts image input via a ~928MB multimodal projector (mmproj). A vision encoder "compresses the image into a sequence of vectors that live in the same mathematical space as text tokens."

## System Architecture

- **Dual-boot via USB:** NixOS runs from a Corsair MP600 MINI in a USB-C NVMe enclosure. To game, the author unplugs the drive and boots Windows (4080 only). To use LLMs, plugs it back in and reboots.
- **Models stored on TrueNAS** mounted over NFS; llama.cpp waits for the mount before starting.
- **Remote access** via OpenCode over Tailscale.

## Annoyance

The V100 occasionally disappears from `lspci`/`nvidia-smi` after a warm reboot — an ACPI enumeration issue. A cold power cycle always fixes it.

## Cost Breakdown & Alternatives

| Item | Cost |
|---|---|
| Tesla V100 SXM2 16GB (eBay) | ~£150 |
| SXM2-to-PCIe adapter | ~£50 |
| PWM fan jumper cable | ~£2 |
| **Total** | **~£200** |

Alternatives mentioned: P40 (24GB, slower, no tensor cores) and V100 32GB variant (more expensive but still undercuts consumer cards). The author notes: "the 7900 XTX costs £700+ and ROCm support for LLM inference is still rough compared to CUDA."
