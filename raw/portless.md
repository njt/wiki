---
title: "Portless"
url: https://github.com/vercel-labs/portless
date_fetched: 2026-05-14
section: "Random"
---

# Portless: Named Local URLs for Development

## Overview

Portless replaces port numbers with stable, named `.localhost` URLs for local development, designed for both humans and AI agents. Instead of accessing `http://localhost:3000`, developers use `https://myapp.localhost`.

## Key Features

**HTTPS by Default with HTTP/2**
The tool enables HTTPS with HTTP/2 automatically, generating a local certificate authority on first run and adding it to the system trust store.

**Automatic Port Assignment**
When you execute `portless myapp next dev`, the system assigns a random port (4000-4999) via the `PORT` environment variable and injects appropriate flags for frameworks like Vite, Astro, and React Router.

**Git Worktree Support**
The tool automatically detects git worktrees and prepends branch names as subdomains, allowing multiple worktrees to coexist without configuration conflicts.

**Monorepo-Friendly**
A single configuration at the repository root covers all workspace packages, with automatic discovery from `pnpm-workspace.yaml` or the `workspaces` field in `package.json`.

**LAN Mode & Tailscale Integration**
Share development servers across network devices using mDNS (`.local` domains) or securely via Tailscale networks with the `--tailscale` flag.

## How It Works

The system operates through three components:

1. **Proxy Management**: A local HTTPS proxy (port 443 by default, or 80 without TLS) routes incoming requests
2. **App Registration**: Running `portless <name> <command>` assigns a free port and registers the app with the proxy
3. **URL Routing**: Requests to `https://<name>.localhost` pass through the proxy to the appropriate app instance

## Main Use Cases

- Consistent Development URLs
- Framework Compatibility (Next.js, Express, Nuxt, Vite)
- Team Collaboration via Tailscale
- Multi-Service Development with subdomains
