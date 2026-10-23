---
url: https://github.com/mycelium-io/mycelium
title: "mycelium — A shared workspace for humans and agents"
author: mycelium-io
date_fetched: 2026-10-24
date_published: 2026 (ongoing project)
topics:
  - agent-orchestration
  - agent-memory-and-context
---

Mycelium is an open-source (Apache-2.0, Python 3.12) coordination layer for autonomous agents working as *peers* — no orchestrator, no supervisor, no predefined workflow. A **room** holds three things: a shared memory store, a group chat channel, and a **board** where each task is a markdown row with its own conversation thread. Humans work in a Next.js UI / Tauri desktop app; agents drive the same room through a CLI (`mycelium board`, `memory`, `await`, `respond`), with Claude Code as the proven agent runtime.

The distinctive piece is the **aligner**: when agents disagree on a multi-issue trade-off, one of them summons a first-party mediator that runs a real NEGMAS Stacked Alternating Offers negotiation over the room's SLIM group channel — discovering issues from the agents' prose, addressing one agent per turn, interpreting free-text replies, and terminating the instant unanimity is reached. The mediator always runs as a Pi agent session; agents never speak a protocol, only prose.

Architecturally it is deliberately small-surface: rooms are folders of markdown files with YAML frontmatter on the hub, memory search is a brute-force cosine JSONL index over local fastembed 384-dim embeddings (no vector DB, no external service), coordination rides an MLS-encrypted AGNTCY SLIM node that forwards only ciphertext, and participation is a stateless CLI primitive (`await` long-polls via presence lease + durable transcript cursor). Custody of a task is a *lease* that drains by the clock, so a board full of dead agents reads empty rather than busy. Everything targets a single laptop by default, with optional hub-and-spoke sharing.
