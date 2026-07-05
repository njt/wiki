---
url: https://zed.dev/blog/introducing-deltadb
title: "Introducing DeltaDB"
author: Nathan Sobo
date_fetched: 2026-06-12
date_published: 2026-06-11
---

# Software Is Made Between Commits — Full Article Summary

**Author:** Nathan Sobo
**Published:** June 11th, 2026
**Publication:** Zed's Blog

## Core Thesis

Nathan Sobo explains that he was never a fan of pull requests, finding that the "ceremony of trading comments on snapshots" didn't work for Zed's team. They preferred discussing code *as* they wrote it, but GitHub only allows discussion after commits are pushed — by which time key conversations were over.

## Founding Vision

Zed was founded in 2021 to move "beyond the constraints of commits." The plan was to build an editor first, then offer better collaboration inside it. The rise of AI agents made these problems even more pressing: the conversation generating code is becoming "the true source of our software."

## DeltaDB — The Core Innovation

DeltaDB is described as "a new kind of version control built on a single coherent abstraction." It transforms agent conversations and the worktrees they edit into shared artifacts. A beta version was promised "in a few weeks."

### Key Technical Details

| Aspect | How It Works |
|--------|-------------|
| **Unit of tracking** | Fine-grained *deltas* rather than commit snapshots |
| **Addressability** | Every delta has a stable identity; code can be referenced at any moment even as it changes |
| **Conversation binding** | Messages and the edits they produce are "recorded side by side" |
| **Replication** | Uses conflict-free replicated worktrees enabling multi-user/multi-agent concurrent editing |
| **Filesystem** | Files are real — agents edit through a terminal; the worktree can be mounted to disk for use with local tools |

### Reference Anchoring

References anchor to deltas instead of line numbers, so they survive as code shifts. Users can jump from a past conversation line to the code "as it stands now or as it stood the moment the agent wrote it." Conversely, any line of code reveals the conversation that produced it and all subsequent conversations touching it.

### Agent Integration

Agents can draw on this context too — they "pick up the context behind the code they're touching" or can convene prior agents that worked on it to ask about intent.

## Collaboration Without Commits

The vision is that "the conversation with the agent becomes the only conversation you need to have." A teammate can join mid-work, talk to the agent that made changes, and annotate in real time without waiting for a commit/push cycle. Pull requests, review threads, and inline comments exist only because discussion and code were separated — put them together and "the ceremony disappears." Git and CI remain for running checks and external connectivity.

## Related Posts Referenced

1. **"We're Not Building AI Features for the Money"** — Conrad Irwin, May 5, 2026
2. **"Introducing Parallel Agents in Zed"** — Mikayla Maki & Richard Feldman, April 22, 2026
3. **"Introducing Zed AI"** — Nathan Sobo, August 20, 2024

## Call to Action

A waitlist for early DeltaDB access was available at `/deltadb`. The post ends by asserting that software "now takes shape in the conversation, not the commit."
