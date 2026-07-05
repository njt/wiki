---
url: https://agentcookie.dev/
date_fetched: 2026-07-05
backfilled: true
---

cookies, bearer tokens, and per-CLI auth blobs replicated continuously from your laptop to the Mac your agent runs on. encrypted over Tailscale, zero per-site auth ceremony.

no `auth login`, no Keychain prompt, no paste-the-cookie ritual. cookies were already there.

per-CLI bearer tokens and API keys, declared once, synced across the wire. read by every PP CLI; readable by 1Password or whatever else fills the bus.

two surfaces, one encrypted push. cookies for browser-driving agents and adapter-equipped CLIs; secrets bus for everything with bearer auth.

fsnotify on Chrome's Cookies file, debounced, allowlist + blocklist filtered, AES-256-GCM over Tailscale.

Chrome's SQLite re-encrypted for the sink keychain, plaintext sidecar at ~/.agentcookie/cookies-plain.db, or per-CLI adapter session files.

Stripe, Linear, Notion, Granola, Slack, Kalshi, ElevenLabs, Mercury, and dozens more - anything with a bearer token or API key reads the secrets bus. Five (instacart, airbnb, ebay, pagliacci, table-reservation-goat) additionally get a bespoke cookie adapter.

bearer tokens, API keys, KEY=VALUE auth blobs ride the same encrypted push and land at ~/.agentcookie/secrets/<cli>/secrets.env (mode 0600) with an optional sealed twin.

drop an agentcookie.toml in your repo and agentcookie discover auto-detects it. three integration tiers (explicit, pp-cli-derived, legacy v1) coexist.

both ends bind tailnet-private addresses. pair endpoint is rate-limited with a 64-bit code.

persistent replay defense and pairing-derived per-peer keys; pairing-code rotation re-derives both ends.

every release binary signed and timestamped. per-binary -T Keychain ACL on Chrome Safe Storage so no AllowAlways prompt fires after install.

no GUI clicks required. install-beta.sh runs end to end on a Mac mini you have never opened a window on.

binary signature, Tailscale, config, keystore, listener bind, sink/source state, sealing posture, adapter coverage, CDP injector health, and secrets bus coverage.

macOS only on both ends today. 449+ unit tests across 26 packages.
