---
title: "onecli"
url: https://github.com/onecli/onecli
date_fetched: 2026-05-14
section: "Security"
topics:
  - security-and-sandboxing
---

# OneCLI: Credential Vault for AI Agents

## Main Purpose
Open-source gateway that manages API credentials for AI agents. Instead of embedding secrets directly, credentials are stored centrally and injected transparently during API calls, so agents never directly access real keys.

## How It Works
Agents use placeholder credentials (e.g., `FAKE_KEY`). When making API requests through the gateway, OneCLI:
1. Intercepts the outbound request
2. Matches it against stored credential patterns
3. Replaces fake credentials with real ones
4. Injects actual secrets into the request
5. Forwards the authenticated call to the target service

## Key Features
- Transparent credential injection via HTTP gateway
- AES-256-GCM encryption at rest; secrets decrypted only during request processing
- Host & path matching to route credentials to specific API endpoints
- Multi-agent support with scoped access tokens
- Dual authentication: single-user local mode or Google OAuth for teams
- Bitwarden and other password manager integration
- Rust-based gateway for high performance
- HTTPS interception capabilities

## Architecture
- **Rust Gateway** (port 10255): Intercepts and modifies outbound HTTP requests
- **Web Dashboard** (port 10254): Next.js app for managing agents, secrets, permissions
- **Secret Store**: Encrypted credential repository matching requests by host and path

## Tech Stack
TypeScript (61.4%), Rust (36.8%), PostgreSQL, Next.js. Apache-2.0 licensed.
