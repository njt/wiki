# bcc

The BPF Compiler Collection: tools for BPF-based Linux IO analysis, networking, monitoring, and more. Write kernel-level tracing programs in C with Python/Lua frontends, using eBPF to safely instrument a running kernel without risking crashes or hangs. This is the standard toolkit for Linux performance engineering -- 22.4k stars, battle-tested in production at scale.

---

## Key Quotes

> "Tools for BPF-based Linux IO analysis, networking, monitoring, and more."

## Key Themes

#Linux #performance #eBPF #monitoring #systems

eBPF is one of the most important developments in Linux systems engineering in the last decade. It lets you run sandboxed programs in kernel space -- safe by construction (can't crash, can't hang, can't corrupt) -- and instrument anything: syscalls, network packets, file operations, scheduler decisions, memory allocations. bcc makes this accessible through Python bindings and a library of ready-made tools.

The tool collection covers the full surface area of system performance: CPU profiling, disk I/O latency, network connection tracking, memory leak detection, process execution tracing. Each tool is a focused instrument for a specific question, following the Unix philosophy.

## Critical Analysis

bcc is the established toolkit, but the eBPF space has evolved. bpftrace (from the same community) provides a higher-level scripting language for one-off queries. libbpf-tools (also in the iovisor org) pre-compiles tools for lower overhead. The trend is toward CO-RE (Compile Once, Run Everywhere) BPF programs that don't need kernel headers at runtime.

For agent systems running on Linux (which is most of them), eBPF-based monitoring is the right way to understand what your containers are actually doing at the system level. The sandboxing discussion in [[A Deep Dive on Agent Sandboxes]] focuses on restricting agent behavior; bcc provides the complementary capability of *observing* what agents (and their containers) are doing.

The Python frontend makes bcc accessible to developers who aren't kernel programmers, but the C kernel-space programs still require understanding BPF constraints and kernel data structures. It's not plug-and-play for most teams.

---
*Sources: [[raw/bcc]]*
*Last updated: 2026-05-14*
