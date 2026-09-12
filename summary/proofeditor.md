---
url: https://proofeditor.ai/
title: Proof — Agent-first collaborative document editor
author: Every Inc. (every.to)
date_fetched: 2026-05-14
date_published: unknown
topics:
  - agent-architecture
---

# ProofEditor

Proof is an online collaborative document editor "built for agents and humans to collaborate." Built by Every (every.to). Free, no login required.

## Key Features

- Live collaboration between humans and AI agents (Claude Code, ChatGPT, Codex, OpenClaw)
- Comments and threads with reply/resolve functionality
- Suggestions mode — agents suggest edits; humans accept, reject, or reply
- Provenance tracking — every character tracks who wrote it
- Shareable links — each doc gets a unique URL
- API integration for programmatic document creation

## How It Works

1. Create a doc via "Get started" or the API
2. Share the link with agents or humans
3. Collaborate — agents comment/suggest, humans review

## Use Cases

Bug reports, PRDs, implementation plans, research briefs, growth reports, copy audits, strategy docs, memos, proposals.

## Agent Integration

Skill installation for Claude Code: `mkdir -p ~/.claude/skills/proof && curl -fsSL https://www.proofeditor.ai/proof.SKILL.md -o ~/.claude/skills/proof/SKILL.md`

Same pattern for Codex under `~/.codex/skills/proof`.

Setup asks: when should new docs be opened in Proof? Options: all markdown docs, only collaborative docs (plans/specs), or only when explicitly asked.

## API

- `POST /share/markdown` — create shared doc from markdown
- `GET /api/agent/<slug>/state` — document state with comment threads
- `GET /api/agent/<slug>/snapshot` — mutationBase token and block refs
- `POST /api/agent/<slug>/edit/v2` — edit blocks (idempotency key required)
- `POST /api/agent/<slug>/ops` — resolve/unresolve comments
- `GET /api/agent/<slug>/events/pending` — check for new activity
- `POST /api/agent/<slug>/presence` — join with presence identity
- `PUT /api/documents/<slug>/title` — update document title

Auth via bearer tokens or share tokens. `X-Agent-Id: ai:<agent-id>` header for presence.

Local macOS app bridge available at `localhost:9847`.

## SDK & Docs

- GitHub SDK: https://github.com/EveryInc/proof-sdk
- Agent docs: https://proofeditor.ai/agent-docs
- Agent manifest: https://proofeditor.ai/.well-known/agent.json
- Bug reporting: POST /api/bridge/report_bug
