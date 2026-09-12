---
title: "bcc"
url: https://github.com/iovisor/bcc
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# BCC (BPF Compiler Collection)

## Primary Purpose
Toolkit for creating efficient kernel tracing and manipulation programs using extended BPF (eBPF). Enables developers to write performance analysis, networking, and system monitoring tools with minimal complexity.

## Key Technologies
- **Foundation**: Extended BPF (eBPF), introduced in Linux 3.15
- **Requirements**: Linux 4.1 and above
- **Languages**: C for kernel programs, Python and Lua frontends
- **Compilation**: LLVM-BPF backend for JIT compilation

## Core Capabilities

### Tracing & Monitoring
- **Performance Analysis**: CPU profiling, scheduler latency, cache analysis, function call timing
- **Storage & I/O**: Block device monitoring, filesystem operation tracking, page cache analysis, latency distribution
- **Networking**: TCP connection tracking, packet analysis, network queue monitoring, traffic flow analysis
- **System Processes**: Memory leak detection, process execution tracking, signal monitoring, resource allocation analysis

## Architecture Components

**Kernel-Space**: BPF programs written in C that execute safely within kernel boundaries

**User-Space**: Python bindings and command-line tools for program management and data collection

**Security Model**: Programs cannot crash, hang, or negatively impact the kernel—fundamental to eBPF design philosophy

## Statistics
- 22.4k GitHub stars
- 5,077 commits on master branch
- Code: 88.2% C, 6.5% Python, 3.8% C++, 1.2% Lua
