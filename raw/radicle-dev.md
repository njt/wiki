---
url: https://radicle.dev/
date_fetched: 2026-07-05
backfilled: true
---

**Radicle** is a sovereign
      {code forge} built on Git.
    

# Synopsis

Radicle is an open source, peer-to-peer code collaboration stack built on Git. Unlike centralized code hosting platforms, there is no single entity controlling the network. Repositories are replicated across peers in a decentralized manner, and users are in full control of their data and workflow.

  
  The Radicle `heartwood` repository. Repository ID
  `rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5`.

# Get started

💾 Radicle 1.9.1 · 5bd3569e120a6172d9df68e1d1d0eed15e8104b1 2026-05-21T17:33:25+02:00

To install Radicle, simply run the command below from your shell, or go to the download page.

`curl -sSLf https://radicle.dev/install | sh`
  
    
  
Alternatively, you can build from source.

For now, Radicle only works on Linux, macOS and BSD variants, as well as Windows.

## Radicle Desktop 🖥️

For a graphical collaborative experience check out the Radicle Desktop client, as well.

# How it works

The Radicle protocol leverages cryptographic identities for code and social artifacts, utilizes Git for efficient data transfer between peers, and employs a custom gossip protocol for exchanging repository metadata.

## Your Data, Forever and Secure

All social artifacts are stored in Git, and signed using public-key cryptography. Radicle verifies the authenticity and authorship of all data for you.

## Unparalleled Autonomy

Radicle enables users to run their own nodes, ensuring censorship-resistant code collaboration and fostering a resilient network without reliance on third-parties.

## Local-first

Radicle is local-first, providing always-available functionality even without internet access. Users own their data, making migration, backup, and access easy both online and offline.

## Evolvable & Extensible

Radicle’s Collaborative Objects (COBs) provide Radicle’s *social
primitive*. This enables features such as issues, discussions and code review
to be implemented as Git objects. Developers can extend Radicle’s capabilities
to build any kind of collaboration flow they see fit.

## Modular by Design

The Radicle Stack comes with a CLI, web interface and TUI, that are backed by the Radicle Node and HTTP Daemon. It’s modular, so any part can be swapped out and other clients can be developed.

┌─────────────────┐┌────────────────┐ │ Radicle CLI ││ Radicle Web │ └─────────────────┘└────────────────┘ ┌───────────────────────────────────┐ │ Radicle Repository │ │ ┌────────┐ ┌────────┐ ┌─────────┐ │ │ │ code │ │ issues │ │ patches │ │ │ └────────┘ └────────┘ └─────────┘ │ ├───────────────────────────────────┤ │ Radicle Storage (Git) │ └───────────────────────────────────┘ ┌────────────────┐┌─────────────────┐ │ Radicle Node ││ Radicle HTTPD │ ├────────────────┤├─────────────────┤ │ NoiseXK ││ HTTP + JSON │ └────────────────┘└─────────────────┘

# Contributing

Radicle is *free and open source* software under the MIT and Apache 2.0
licenses. Get involved by contributing code.

# Updates

**Follow us** on 🐘 Mastodon, 🦋 Bluesky or 🐦 Twitter to stay
updated, join our community on 💬 Zulip, or 
  Subscribe 

- 22.05.2026 Radicle 1.9.1 released.
- 19.05.2026 Radicle 1.9.0 released. ☀️
- 23.04.2026 We have moved to radicle.{dev,network}! 🌐
- 30.03.2026 Radicle 1.8.0 released. ✨
- 30.03.2026 Disclosure of Vulnerability in Signed References. 🔐
- 20.03.2026 Radicle 1.7.1 released. 🛠️
- 18.03.2026 Radicle 1.7.0 released. 🌤️
- 14.01.2026 Radicle 1.6.0 released.
- 30.09.2025 Radicle 1.5.0 released.
- 04.09.2025 Radicle 1.4.0 released.
- 12.08.2025 Radicle 1.3.0 released.
- 17.07.2025 Radicle 1.2.1 released.
- 13.06.2025 Radicle Desktop is out. 🖥️
- 02.06.2025 Radicle 1.2.0 released.
- 05.12.2024 Radicle 1.1.0 released.
- 10.09.2024 Radicle 1.0.0 released.
- 26.03.2024 Radicle 1.0.0-rc.1 released.
- 10.03.2024 New Radicle homepage.
- 05.03.2024 Radicle Guides launch.
- 05.03.2024 Radicle makes it to the top of Hacker News!
- 18.04.2023 Radicle heartwood is announced.

## Blog

- 02.06.2026 Plant your Seeds in the Radicle Garden 🌱
- 14.08.2025 Jujutsu + Radicle = ❤️
- 12.08.2025 Canonical References
- 23.07.2025 Using Radicle CI for Development
- 30.05.2025 How we used Radicle with GitHub Actions

# Feedback

If you have feedback, join our Zulip or send us an email at feedback@radicle.dev. Emails sent to this address are automatically posted to our #feedback channel on Zulip.
