---
url: https://agent-safehouse.dev/
title: "Agent Safehouse — Go full --yolo. We've got you."
author: eugene1g
date_fetched: 2026-11-26
date_published: unknown
topics:
  - security-and-sandboxing
  - coding-agents-and-frameworks
---

Agent Safehouse is a macOS-native sandboxing wrapper for local coding agents, pitched directly at the `--yolo`/`--dangerously-skip-permissions` crowd: run your agent unsandboxed-by-the-agent, but sandboxed-by-the-kernel. It installs via Homebrew or a single self-contained shell script with no dependencies beyond Bash and macOS, and wraps `claude`, `codex`, `amp`, `gemini` and similar CLIs.

The core inversion: agents normally inherit your full user permissions, and Safehouse flips this to deny-by-default — nothing is accessible unless explicitly granted. Read/write on the selected workdir (git root by default) and read on installed toolchains are auto-granted; SSH keys, other repos and personal files are denied at the kernel level, so `cat ~/.ssh/id_ed25519` fails with "Operation not permitted" before the process ever sees the data.

The pitch leans on probability: LLMs are probabilistic, so "1% chance of disaster makes it a matter of *when*, not *if*". Shell function examples show the ergonomic endgame — alias `claude()` to always run sandboxed, and bypass with `command claude` when you really want out. It also offers a ready-made prompt that has an LLM inspect the profile templates and generate a least-privilege `sandbox-exec` profile tailored to your home directory and toolchain.
