---
url: https://tailscale.com/tailcat
title: "Tailcat — Tailscale's official product page"
author: Tailscale Inc
date_fetched: 2026-11-17
date_published: unknown (undated product page)
topics:
  - developer-tools
  - security-and-sandboxing
---

# Tailcat — Tailscale's official product page

The official landing page for Tailcat, Tailscale's netcat-style CLI for connecting two or more machines across different networks with an encrypted connection and **no account, login flow, tailnet, or IP address**. Access comes entirely from possession of a tailcat address: run Tailcat on the machine to reach, share the generated address, connect from the other side. It reuses Tailscale's open-source data plane (WireGuard + NAT traversal + DERP relay) without the control plane; the hosted relay keeps metadata logs and "isn't intended for privacy."

The page is explicit about scope and positioning. Tailcat is for short-lived connections — SSH into a dev environment for an hour, a CI job, a game session, moving a file — while Tailscale proper is for ongoing, governable, auditable connectivity with identity and policy. Identity is not absent, just minimal: each node presents public keys, connections exchange "MEOW messages," and an optional key-based allowlist gates access. Keys default to ephemeral per run, with optional long-lived keypairs.

Two answers stand out for this wiki. First, the address is a bearer capability: "Treat it like a password. Don't log it, commit it to source control, or share it with anyone who shouldn't have access. Definitely don't tweet it." Second, agents are called out as a first-class use case: agents "can't always complete login flows and may not need persistent network access," so you give one a tailcat address for a single task — like piping dev files to an AI agent sandbox for testing — without joining it to a tailnet.
