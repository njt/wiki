---
url: https://github.com/juanfont/headscale
title: "Headscale: An open source, self-hosted implementation of the Tailscale control server"
author: Juan Font Alonso, Kristoffer Dalby
date_fetched: 2026-05-14
date_published: null
topics:
  - misc
---

# Headscale

An open source, self-hosted implementation of the Tailscale control server.

## What It Is

Headscale aims to implement a self-hosted, open source alternative to the Tailscale control server. Tailscale is a modern VPN built on top of Wireguard. It works like an overlay network between the computers of your network. The control server coordinates the overlay network and provides key exchange, IP assignment, user management, machine sharing between users, and advertised route exposure.

Headscale implements this narrow coordination/management server. It does not replace the Tailscale client apps themselves -- you still use the official Tailscale clients on all platforms.

## Scope

- Designed for self-hosters and hobbyists
- Supports a single tailnet (Tailscale network)
- Suitable for personal use or small organizations
- Not intended for enterprise-scale deployments

## Technical Details

- **Language**: Go (98.6% of codebase)
- **Protocol**: Protobuf (uses Buf for code generation)
- **Networking**: WireGuard-based overlay network with NAT traversal
- **Configuration**: YAML-based
- **License**: BSD-3-Clause
- **Stars**: 38.4k
- **Forks**: 2.1k
- **Latest release**: v0.28.0 (February 2026)

## Features

- Self-hosted Tailscale-compatible control server
- OpenID Connect authentication
- ACL (access control list) management
- DNS configuration
- DERP (relay) server functionality
- Route management
- TLS security
- API access
- Tag-based organization
- Web UI integration
- Reverse proxy compatibility
- Support for Android, Apple, and Windows clients

## How It Works

Headscale functions as an exchange point for WireGuard public keys, assigns IP addresses to clients, manages user boundaries, enables machine sharing between users, and exposes advertised network routes -- essentially replicating the Tailscale control server's core functions.

## Development

- Nix recommended for consistent development environment
- Code formatted with golangci-lint, golines, gofumpt (Go); buf, clang-format (Protobuf); mdformat (docs); prettier (other files)
- Integration testing framework for quality assurance
- Community on Discord

## Governance

Created by Juan Font Alonso. Maintained by Kristoffer Dalby and Juan Font. Active maintainer employed by Tailscale Inc., contributing work hours while following community-reviewed contribution processes.

## Documentation

Available at headscale.net with separate documentation versions for stable releases and development builds.
