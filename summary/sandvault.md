---
url: https://github.com/webcoyote/sandvault
title: "SandVault - Run AI agents and shell commands in a sandboxed macOS user account"
author: Patrick Wyatt (webcoyote)
date_fetched: 2026-11-02
date_published: 2026
topics:
  - security-and-sandboxing
  - coding-agents-and-frameworks
---

SandVault (`sv`) is a ~2,600-line Bash + Python toolkit that sandboxes AI coding agents (Claude Code, Codex, OpenCode, Gemini, pi, Muse Code) and shell commands on macOS without virtual machines. It creates a per-host limited user account (`sandvault-<username>`), gives it a shared writable workspace at `/Users/Shared/sv-$USER` governed by carefully split macOS ACLs, and layers a root-owned `sandbox-exec` (Seatbelt) profile on top for defense in depth — denying writes everywhere except the sandbox home and shared workspace, denying `/Volumes` reads except the boot disk, blocking disk mounts and raw-disk devices, and default-denying `/Users` reads with explicit re-allows.

Commands run either via passwordless `sudo -u` impersonation or via loopback SSH (`127.0.0.1`, deliberately avoiding hostname resolution). Six agent wrappers each launch their tool in its most permissive mode (`--dangerously-skip-permissions`, `--yolo`, etc.) — the argument being that sandvault itself is the sandbox. Tool-awareness prompts are injected via `--append-system-prompt` when optional services are live.

Two bridges expose host-side resources to sandboxed agents over localhost: a headless browser (Lightpanda by default, or Chrome — which must run `--no-sandbox` inside the sandbox) reachable via Chrome DevTools Protocol through `SV_BROWSER_ENDPOINT`, and an iOS Simulator HTTP bridge (`helpers/sv-ios-bridge`, Python) fronting a whitelisted set of `xcrun simctl`/`iosef` subcommands against a per-session scratch simulator, with per-endpoint timeout tuning and path whitelisting for `.app` installs. An SSH `authorized_keys.d` directory acts as declarative key revocation, regenerating the sandbox user's `authorized_keys` from the host-owned source of truth on every run.

The project's pitch: macOS's Unix user-account heritage gives meaningful isolation with zero VM overhead, instant switching, and a clean uninstall — an alternative to Docker/Podman/VM approaches like ClodPod or Chamber.
