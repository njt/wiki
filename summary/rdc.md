---
url: https://github.com/bscott/rdc
title: "rdc — Remote Desktop Control for AI Agents"
author: bscott
date_fetched: 2026-09-13
date_published: 2026-09-10
topics:
  - mcp-and-tool-protocols
  - security-and-sandboxing
---

rdc is a single Rust binary (GPLv3+, ~4,900 lines) that lets a coding agent on one machine see and drive another computer's desktop over a Tailscale tailnet: screenshots, mouse, keyboard, window focus, clipboard. One executable plays three roles by subcommand — `rdc serve` is the daemon on the controlled machine, `rdc mcp` is a stdio MCP server for the agent's machine, and the same operations exist as CLI subcommands. Everything sits on one seam, the `Desktop` trait, implemented twice: in-process (xcap capture, enigo input, arboard clipboard) and over HTTP against a remote daemon, so `--target local` and `--target somehost` are the same code. Linux (Wayland/X11), macOS and Windows are supported, with an honest status table (GNOME/KDE Wayland is capture-only).

Its defining choice is authentication: no passwords, tokens or certificates. The daemon refuses to bind anything but its Tailscale address (or loopback under an explicit dev flag), takes the peer IP from the socket, resolves it to a login, node name or tag via tailscaled's `whois`, and matches against a config-file allowlist whose entries can be scoped to `view`, `input` or `clipboard` capability. The tailnet itself — WireGuard-encrypted, identity-carrying — is the security boundary. Fail-closed rules run throughout: empty allowlist refuses to start, tagged devices are identified by tags only (the creating user's login is deliberately discarded), the Host header must name the machine (DNS-rebinding defense), and every request or rejection is written to a JSON-lines audit log that never records typed text or clipboard contents. There is deliberately no shell and no file transfer — rdc is a screen-and-input surface only.

For the agent, the MCP layer speaks screenshot coordinates: tools like `click` take pixels in the most recent screenshot, and a `ViewMap` converts them back to logical desktop points, absorbing Retina scaling, multi-monitor offsets and the 1568px downscale that keeps images cheap for a vision model. Every action returns a fresh screenshot by default (with a 350ms settle), so the agent can verify its work; tool calls are serialized behind a mutex so a burst keeps its coordinate mapping valid. An Agent Skill (`skills/rdc/SKILL.md`) and a contributor-facing `AGENTS.md` of design and security principles ship in the repo. Version 0.3.0 (2026-09-10) added a Unix config-file ownership check: a group- or world-writable `config.toml` refuses to run, because editing the allowlist is equivalent to desktop access.
