---
url: https://arcan-fe.com/2022/04/02/the-day-of-a-new-command-line-interface-shell/
title: "The Day of a New Command-Line Interface Shell"
author: Björn Stahl (Arcan project)
date_fetched: 2026-10-10
date_published: 2022-04-02
topics:
  - developer-tools
  - software-engineering-craft
---

A long-form design essay from the Arcan display-server project arguing that the terminal stack — tty devices, escape sequences, terminal emulators — is a 50–70-year-old device model that should be retired, not extended. The "shell" is dissected into two roles: a job/window manager (the prompt, pipelines, foreground/background) and a scripting environment, and the essay shows how the second role colonised the first.

The core diagnosis is that stdio (fd 0/1/2) mixes data with control: in-band escape sequences mean any program — or an accidental `cat /dev/random` — executes arbitrary instructions against the terminal's state machine. Emulators are "a poor take on a display server": one shared serial triplet instead of per-job bidirectional connections, no safe layering or nesting (hence tmux/screen embedding recursive emulators), and line-mode/screen-mode history contradictions inherited from when output was paper.

The replacement architecture removes the terminal emulator and tty layer entirely. The shell talks to a display server (Arcan's shmif IPC) using handover allocation — a shell requests a window for a job and forwards the connection primitives, retaining a chain of trust — and live migration for crash resilience. Each pipeline stage becomes a separate, addressable client; stdin/stdout stay pure data.

The gains catalogue reads as a checklist of everything today's terminals fake: uncorruptible UI state, non-ambiguous input (ESC key ≠ ESC byte, Ctrl+C as a symbol not SIGINT), 24-bit colour with semantic palettes, tear-free atomic presentation, semantic tagging for screenreaders, clipboard/file-picker/notification integration delegated to the graphical shell, and locale as a window property rather than a process-global environment variable.
