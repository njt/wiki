---
url: https://rewindvm.dev/
title: "Rewind VM — Deterministic Linux VMs"
author: Farid Zakaria (fzakaria)
date_fetched: 2026-10-10
date_published: unknown
topics:
  - developer-tools
  - software-engineering-craft
---

Rewind VM (rewindvm.dev) runs a Nix build, a test suite, or any Linux command inside a deterministic KVM virtual machine: the same inputs produce the same events at the same steps, every run. Because nothing is recorded — a run is a pure function of its inputs — any step can be recomputed. The engine and CLI are MIT-licensed open source; a desktop app (timeline scrubbing, fork-from-step, side-by-side passing/failing runs) costs $49 personal / $99 per seat.

The mechanics: inputs (a Nix closure, a directory, a `docker export` tarball) become a read-only erofs image mapped into the VM's memory; one vCPU on stock KVM with a patched kernel that only lets interrupts, time, the TSC and the hardware RNG advance when the VM hands control to Rewind — each handoff is a "step." Every quarter-second of wall time, a keyframe snapshot is taken via KVM's dirty-page log into a content-addressed BLAKE3+zstd store. Seeking restores the nearest keyframe and replays forward.

Beyond replay, `rewind check` hunts flaky builds: it reruns the workload under perturbed schedules until one ends differently, then names the exact step whose reschedule decided the outcome. For Nix users the flake attribute is the entry point (`rewind nix .#mylib`), with per-output hash comparison against the host store and the VM's kernel itself pinned as a derivation. Case studies show real reproductions: a previously unreported SIGPIPE flake in Nix's `gc-closure.sh` test, a known SQLite `SQLITE_BUSY_SNAPSHOT` hang in Nix's schema migration, and a lost-task-output bug in devenv traced to `tokio::select!`.

Limits are stated plainly: x86_64 Linux with KVM, one vCPU (threads take turns; parallelism comes from many VMs), no network beyond loopback, no interrupting a loop that spins without a syscall, and replays only across same-CPU-vendor machines.
