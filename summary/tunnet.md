---
url: https://github.com/tunnetio/Tunnet
title: Tunnet
author: tunnetio
date_fetched: 2026-07-18
---

# Tunnet

Open-source mesh VPN platform (~26K lines of Rust, AGPL-3.0) that connects machines into a private mesh network over QUIC (iroh). Bundles six networking primitives under one identity system and policy engine: mesh networking, internal service exposure (Serve with TLS and ACLs), public tunneling via self-hosted relay (ACME support), P2P file transfer (BLAKE3-verified, consent-based), and identity-based SSH (session recording, re-auth, SFTP).

Operates in two modes: **Managed** (control plane with PostgreSQL, WebSocket sync, dashboard, SSO/OIDC, centralized policies) and **Direct** (pure P2P with CRDT membership via iroh-docs, Mainline DHT discovery, PSK auth — no server needed). Migration from Direct to Managed is a single command.

Key architectural choices: lock-free routing table via `ArcSwap` for the packet-forwarding hot path, on-demand connection pool that buffers packets during reconnect (Direct mode), dual-path sync (WebSocket + polling fallback) for resilience, self-certifying Ed25519 identities, signed policy bundles, stateful conntrack firewall, and tiered secret sealing (platform keychain/TPM).
