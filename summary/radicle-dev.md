---
url: https://radicle.dev/
title: "Radicle: The Sovereign Code Forge"
author: Radicle Team (Radworks)
date_fetched: 2026-05-16
date_published: 2026-04-23 (domain migration from radicle.xyz to radicle.dev)
topics:
  - databases-and-data
---

# Radicle

Radicle is an open-source, peer-to-peer, local-first code collaboration stack built on Git. It functions as a decentralized alternative to centralized forges like GitHub/GitLab, with no single entity controlling the network. Repositories are replicated across peers using cryptographic identities (public-key cryptography), and all social artifacts (issues, patches, code reviews) are stored as Collaborative Objects (COBs) within Git.

Tagline: "The Sovereign Code Forge"
FOSDEM 2026 tagline: "Free your code!"

## Architecture

- **Heartwood**: Third iteration of the Radicle Protocol, implemented in Rust
- **radicle-node**: Network daemon using NoiseXK for encrypted connections
- **rad**: Command-line interface
- **Radicle HTTPD**: HTTP + JSON API
- **Radicle Web**: Read-only web frontend (Svelte)
- **Radicle Desktop**: Desktop application (Svelte)
- **Radicle TUI**: Terminal user interface (Rust)

Gossip protocol for peer discovery, secure data replication via cryptographic verification. Collaborative Objects (COBs) extend Git to store issues, patches, and code reviews directly in the repository.

## 2026 Releases

- 1.6.0 (Jan 14, "Amaryllis"): Native Windows support, CLI migration to clap
- 1.7.0 (Mar 18, "Daffodil"): Signed references rework, node blocking, reduced I/O load, security fix
- 1.7.1 (Mar 20): IPv6 parsing hotfix, sigrefs verification regression
- 1.8.0 (Mar 30): Replay attack + graft attack defenses, full vulnerability disclosure

## Community

- HardenedBSD exploring Radicle as primary code host after GitLab issues
- 24-hour community response: 6 global replicas set up
- New engineer (ade) joined Q1 2026, 18 years P2P/real-time systems experience
- FOSDEM 2026: Two talks (P2P code collaboration, local-first code collaboration)
- Budget Q1 2026: CHF 122,800 (13% under budget)

## Limitations

- Web frontend is read-only — interaction requires CLI or desktop app
- No built-in releases, project management (kanban), milestones, or due dates
- Simple issue management (tags, comments, reactions, assignments)
- No bundled package registries

## Upcoming

- Radicle 1.9.0: I2P network integration, symbolic references
- Signed push certificates (migrating toward Git-native push certificates)
- Simulation testing framework
- Radicle URI standard (RIP near acceptance)
- Improved internal architecture (storage layer isolation, testability)

Sources: radicle.dev (403 from WebFetch), GitHub radicle-dev/heartwood README, Radworks Community Q1 2026 Update, FOSDEM 2026 schedule
