---
url: https://agentcookie.dev/
title: "agentcookie - session state sync for the agent on your second Mac"
og_title: "agentcookie — your agent's session state, synced"
og_description: "Cookies and per-CLI secrets, replicated continuously from your laptop to the Mac your agent runs on. Encrypted over Tailscale, zero per-site auth ceremony."
author: Matt Van Horn (@mvanhorn)
date_fetched: 2026-06-02
date_published: unknown
surf_verified: true
topics:
  - agent-architecture
---

# Agentcookie

"your agent's session state, synced"

Agentcookie is a macOS tool that continuously syncs browser session state (cookies, bearer tokens, API keys, auth blobs) from a primary laptop to a secondary Mac where an AI agent operates. It works over Tailscale with encryption, eliminating the need for manual per-site authentication on the agent's machine.

GitHub: [github.com/mvanhorn/agentcookie](https://github.com/mvanhorn/agentcookie)
MIT licensed. macOS only. 449+ unit tests across 26 packages.

## How It Works

- Cookies and tokens replicate continuously from the user's laptop to the agent Mac
- Encrypted over Tailscale with AES-256-GCM
- "zero per-site auth ceremony"

### Two Core Mechanisms

1. **Cookies** — "for browser-driving agents and adapter-equipped CLIs"
2. **Secrets bus** — "for everything with bearer auth"

### Adoption Manifest (agentcookie.toml)

A TOML config file where services declare what secrets to sync:

```toml
schema_version = 2
name = "stripe-pp-cli"
display_name = "Stripe"

[secrets.file]
path = "~/.config/stripe-pp-cli/config.toml"

[sync.keys]
STRIPE_SECRET_KEY = true
```

Synced secrets land at `~/.agentcookie/secrets/stripe-pp-cli/secrets.env` with mode 0600.

## 11 Features (Working Today)

1. **Continuous laptop → sink sync** — fsnotify on Chrome's Cookies file with debouncing, allowlist/blocklist filtering, AES-256-GCM over Tailscale
2. **Three cookie delivery surfaces** — Chrome's SQLite re-encrypted for sink keychain, plaintext sidecar at `~/.agentcookie/cookies-plain.db`, or per-CLI adapter session files
3. **Compatible with many PP CLIs** — Stripe, Linear, Notion, Granola, Slack, Kalshi, ElevenLabs, Mercury, and "dozens more." Five services (Instacart, Airbnb, eBay, Pagliacci, table-reservation-goat) have bespoke cookie adapters
4. **Per-CLI secrets bus** — KEY=VALUE auth blobs travel the same encrypted path and land at `~/.agentcookie/secrets/<cli>/secrets.env` (mode 0600) with optional sealed twin
5. **V2 adoption standard** — Drop an `agentcookie.toml` in a repo; `agentcookie discover` auto-detects it. Three integration tiers (explicit, pp-cli-derived, legacy v1) coexist
6. **Tailnet-only listeners** — Both ends bind tailnet-private addresses. Pair endpoint is rate-limited with a 64-bit code
7. **Replay defense, per-peer keys** — "persistent replay defense and pairing-derived per-peer keys; pairing-code rotation re-derives both ends"
8. **Apple Developer ID signed** — Every release binary signed and timestamped. Uses per-binary `-T` Keychain ACL on Chrome Safe Storage
9. **Headless install over SSH** — "no GUI clicks required." The `install-beta.sh` script runs end-to-end on a Mac mini that has never had a window opened on it
10. **11-category doctor** — Diagnostic tool covering: binary signature, Tailscale, config, keystore, listener bind, sink/source state, sealing posture, adapter coverage, CDP injector health, and secrets bus coverage
11. **"macOS only on both ends today."** — 449+ unit tests across 26 packages

## References

- GitHub: github.com/mvanhorn/agentcookie
- Quickstart, v1 spec, v2 adoption spec, threat model (all GitHub docs)
- Install instructions (GitHub README)
