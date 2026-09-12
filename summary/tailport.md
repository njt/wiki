---
url: https://github.com/gruen/tailport
title: "Tailport: a TUI for exposing local ports across your tailnet via tailscale serve"
author: Michael E. Gruen
date_fetched: 2026-09-04
date_published: null
topics:
  - developer-tools
---

# Tailport

A terminal UI (Go, Bubble Tea) that lists a machine's locally listening TCP ports and toggles Tailscale `serve` on or off for each one, so you can expose a local dev server to your private tailnet at `http://<hostname>:<port>` without remembering `tailscale serve` syntax. It is a personal tool built for a specific home tailnet setup, shared as-is.

## What It Is

tailport discovers listening ports with `ss` (Linux) or `lsof` (macOS), reads current serve/funnel state from `tailscale serve status --json`, and drives exposure entirely by shelling out to the official `tailscale` CLI. It has no daemon and no dependencies beyond the `tailscale` CLI and the OS tools above.

Exposure is layered and deliberately safe. The default path — `tailscale serve --http=<port>` — exposes a port tailnet-only over plain HTTP at a 1:1 port mapping. Public exposure to the internet is possible two ways, both explicit opt-ins behind strong confirms: `tailscale funnel` (Tailscale's HTTPS ingress on ports 443/8443/10000) and a "publish" path that routes a custom public hostname through a user-run Caddy edge node via its tailnet-only admin API.

## Key Features

- **Reachability is derived from bind scope**, not from serve state: a wildcard-bound socket is already tailnet-reachable at the IP layer, so `serve` only matters for loopback-bound apps. tailscaled's own proxy sockets are filtered out (100.64/10, fd7a::/48) so a "dangling forward" — a serve mapping with nothing listening — is detected rather than shown as listening.
- **A port registry** (labels, favorites, locks) persisted to `~/.config/tailport/config.yaml`, seeded with `:22` locked by default. Undo/redo step through registry edits as per-port deltas, never touching what's exposed.
- **Headless modes** (`status`, `quickstart`, `update`) share the exact same read-only data path as the TUI so the report can't drift from the interactive list.
- **Caddy-edge publishing** with optimistic concurrency, byte-faithful undo, and a conflict-classification ladder.
- **Self-update** that verifies the published sha256 before writing and atomically swaps the binary in place.

## Scope

A single Go binary, ~28.7k lines, with a small set of internal packages (`portscan`, `tsserve`, `config`, `caddyedge`, `statusreport`, `clip`, `selfupdate`, `ui`). MIT licensed. Prebuilt for linux/amd64, linux/arm64, and darwin/arm64; packaged for AUR and Homebrew.
