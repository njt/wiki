---
url: https://opentunnel.xyz/
title: "OpenTunnel — public URLs for anything"
author: OpenTunnel
date_fetched: 2026-10-18
date_published: unknown
topics:
  - developer-tools
  - security-and-sandboxing
---

OpenTunnel is a CLI and SDK that creates end-to-end-encrypted public URLs for apps running on your machine. It is in the ngrok/tailscale-funnel family of tools, but with an explicit privacy architecture: the relay that terminates public traffic never holds a key that can decrypt it.

Installable via curl, brew, npm, cargo, or git clone. One command — `opentunnel route add 47365` — creates a tunnel and returns a public URL routing to `127.0.0.1:47365`. An SDK (`@opentunnel/client`) lets apps share a tunnel with subdomain routes per service, aimed squarely at agent runtimes (the examples use "opencode" and webhooks).

How it works: the first route generates a private key on your machine that never leaves it; the CLI requests a certificate bound to the tunnel hostname, with the relay seeing only the public half. A service on your machine keeps an encrypted bridge open to the relay. Visitors hit the public URL; the relay reads only the hostname from the TLS handshake and forwards the encrypted stream; your machine terminates TLS with its own private key and proxies to local apps.

The privacy page is honest about limits: the relay cannot read traffic, but tunnel hostnames are public (published to certificate transparency logs, so discoverable), route names are wildcard-covered so they never appear in logs but remain guessable ("api", "postgres"), and the docs' final word is that anything sensitive should authenticate in the services themselves — the tunnel is not authentication.
