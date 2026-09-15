---
url: https://aprilnea.me/en/blog/reverse-engineering-claude-code-antspace
title: "Reverse Engineering Claude Code: Antspace, Anthropic's Hidden PaaS"
author: AprilNEA
date_fetched: 2026-09-15
date_published: 2026-03-18
topics:
  - claude-code
  - security-and-sandboxing
---

A reverse-engineering walkthrough of Claude Code Web's runtime, performed entirely with standard Linux tooling (`strace`, `strings`, `go tool objdump`) from inside the author's own Claude Code session. The environment turns out to be a Firecracker microVM (4 vCPU, 16GB RAM) restored from a frozen snapshot, with a custom Rust init called `process_api` as PID 1 — no systemd, no sshd — exposing a WebSocket process-supervisor API on port 2024 and an HTTP container-control API on port 2025.

The bigger discovery is `/usr/local/bin/environment-runner`, a 27MB Go binary shipped unstripped with full debug symbols, built from Anthropic's private monorepo. Its symbol table reveals the internal package structure, a BYOC (bring-your-own-cloud) mode for enterprise customers, and two deployment clients: the expected Vercel client, and an undocumented "AntspaceClient" implementing a complete from-scratch deployment protocol. A web-wide search for "Antspace" returns zero results — it appears to be an internal Anthropic hosting platform, likely named for "Ant" (internal nickname) + "Space".

The binary also exposes "Baku", the codename for claude.ai's web app builder: a Vite + React + TypeScript template with auto-provisioned Supabase via six MCP tools, stop hooks that block session end on uncommitted changes or type errors, and Antspace as the default deploy target. The author's conclusion: Anthropic is building a vertically integrated AI-native PaaS — model, runtime, and hosting in one stack — positioned against Vercel, Replit, and Supabase simultaneously.
