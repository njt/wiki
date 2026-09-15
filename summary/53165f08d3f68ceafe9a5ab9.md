---
url: https://notes.ito.com/53165f08d3f68ceafe9a5ab9/
title: "How I run my agents across many accounts"
author: Ito (notes.ito.com)
date_fetched: 2026-09-15
topics:
  - agent-orchestration
  - personal-agents
---

A first-person account of "Mujin" (無人, lights-out factory), the author's end-to-end pipeline for turning issues into landed code with agents. Everything starts as a kata issue; a scheduled grooming lane (Niwashi) proposes implementation cards with a two-model proposer/reviewer check, but only a human — or a tightly capped auto-promoter — can apply the `designed-ready` label that makes an issue executable. Three producer kinds build branches in parallel worktrees: attended sessions running an autonomous `/do-it` pipeline, dispatched workers launched into tmux panes, and an unattended night lane capped at five issues and two concurrent headless sessions.

The most distinctive section is the seat pool: the author holds multiple personal Claude Max and Codex Pro subscriptions and dispatches work across them, reading Anthropic's own usage meters every five minutes, scoring seats by "coldest" headroom plus live-session load, and sweeping stalled interactive sessions onto cooler seats. A separate section argues this relay shape stays inside the vendors' terms of service.

Landing is gated: fresheyes independent review, Repoman as the single writer of main with compare-and-swap pushes and bounce-as-retry, and roborev post-commit review. The honest closing sections cover what is unsolved — detecting wedged versus working sessions, a sweeper blind to a second tmux server, and a shadow-mode dispatcher not yet armed. Every mechanism is traced to a concrete incident that motivated it.
