---
url: https://github.com/kenn-io/constitution
title: "Clanker Constitution"
author: Kenn Software LLC
date_fetched: 2026-11-20
date_published: 2026-11-03
topics:
  - agent-coding-workflow
  - claude-code
---

The Clanker Constitution is a deliberately small repository — a README and a single `CONSTITUTION.md` — that ships a versioned, calendar-tagged set of default operating principles for coding agents (the current release is v2026.11.03). It is designed to be installed as an agent steering file: symlinked into `~/.codex/AGENTS.md` and `~/.claude/rules/` for global use, or copied to a repository root and imported from `CLAUDE.md` via `@CONSTITUTION.md` for per-repo use.

The document itself is seven numbered sections of terse, normative bullet points: honor the request (treat instructions as contracts, match the requested mode), act with judgment (proceed on safe reversible work without asking, but never merge a PR without authorization), finish the job (don't stop at diagnosis when implementation was authorized), protect existing work (never amend, reset, or overwrite without explicit permission), verify reality (test behavior not source text, never claim success without fresh evidence), communicate for humans (lead with the outcome), and learn in the right place (durable guidance goes in `AGENTS.md`, not agent-private memory).

Its interesting moves are governance rather than prompting: an explicit precedence rule (user instructions and repo instructions override the constitution), immutable calendar-versioned release tags, a rule against fetching updates at agent startup (copies are pinned and updated through PR review), and licence terms requiring modified copies to identify their changes and not present themselves as canonical. It is essentially a constitution-as-dependency: a shared, reviewed, versioned baseline for agent behaviour that individual repos can diff against.
