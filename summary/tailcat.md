---
url: https://github.com/tailscale/tailcat
title: Tailcat
author: Tailscale Inc
date_fetched: 2026-09-04
published: 2026-08
topics:
  - developer-tools
---

# Tailcat

"Tailscale without Tailscale, by Tailscale" — a remix of Tailscale's open-source data plane into a netcat-like pipe: point-to-point WireGuard-encrypted tunnels between two machines with DERP as NAT-hole-punching side channel and relay-of-last-resort, but **no Tailscale control plane, no account, no root/admin, no routing-table or DNS changes**. All connection metadata is exchanged out of band, however you want.

One side runs a `tailcat` server and prints a short `tc…` **tailcat address**; the other side passes that address to the client side to connect. The address is a bearer capability: `"tc"` + base64url of CBOR-encoded connection info — the server's WireGuard public key, a separate path-discovery public key, an independent WireGuard pre-shared key (256 random bits, on by default), and either a DERP region ID or full embedded DERP metadata. Traffic bootstraps through DERP, then magicsock upgrades to direct UDP when NAT traversal succeeds.

Built as one Go module: a library (`tailcat`, importable) and a CLI (`cmd/tailcat`) that serves stdin/stdout pipes, TCP ports, exit nodes, SOCKS5 proxies, public-key-authenticated or auth-free SSH, and SFTP file serving (read-only, read-write, or privacy-preserving write-only drop boxes). There's also an experimental WebAssembly web demo that interoperates with the CLI over DERP. Uses `tailscale.com` as a regular Go module dependency — not a fork — and ships smaller binaries by computing a `ts_omit_*` build-tag allowlist from upstream's feature registry.
