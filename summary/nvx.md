---
url: https://github.com/microsoft/nvx
title: "NVX: An Ultra-Light Micro-VM Sandbox"
author: Microsoft Research (MSR Systems Research Group & Azure Research – Systems)
date_fetched: 2026-10-08
topics:
  - security-and-sandboxing
  - coding-agents-and-frameworks
---

NVX is Microsoft's open-source ultra-light micro-VM sandbox for running untrusted workloads with hardware-enforced isolation, built on OpenVMM and running an x86-64 Linux guest with no firmware or PC platform. It grew out of the Nanvix research system and runs on KVM and MSHV on Linux and WHP on Windows, with the same guest-visible machine contract across all three hypervisors.

The repository is mostly a versioned integration: pinned Linux and OpenVMM revisions, patches, guest sources, and a Python harness (`scripts/nvx.py`, ~1,300 lines) that downloads prebuilt packages and drives the VM, plus a Rust crate `aci_edge_sandboxes` (~5,700 lines) exposing a typed five-operation lifecycle (provision → start → exec → stop → deprovision) with pluggable backends and a strict validate-before-run policy layer.

The distinctive part is the guest architecture: exactly one workload container per microVM, assembled from role-bearing EROFS lower layers plus an ext4 scratch via overlayfs, supervised by an agent living in the initramfs outside the workload's namespaces. A 2,300-line C agent (`nvx-managed-agent.c`) speaks a framed, credit-based protocol over a control console with feature negotiation, and snapshots come in three tiers (platform / workload-start / instance-checkpoint) with strict validation of tier–policy–scratch combinations.

NVX also ships an unusual adversarial-testing design: GitHub Copilot CLI is used only as a bounded "strategist" that picks from a catalog of deterministic containment probes, while a typed broker, credential-free executor, and watchdog keep exclusive control — the LLM never supplies commands, paths, or secrets, only a catalogued case ID.
