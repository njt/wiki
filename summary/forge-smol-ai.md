---
url: https://forge.smol.ai/
title: SmolForge — llms.txt
author: swyx (Shawn Wang)
site: SmolForge (forge.smol.ai)
date_fetched: 2026-08-07
topics:
  - developer-tools
---

# SmolForge (forge.smol.ai) — Summary

SmolForge is a fully-realized GitHub clone built entirely on Cloudflare infrastructure (Workers, D1, R2, Durable Objects). It provides Git Smart HTTP hosting, code browsing, issues, pull requests, CI/CD Actions, gists, wikis, organizations, and a comprehensive REST API — all running on Cloudflare's serverless primitives. The source is self-hosted at `forge.smol.ai/swyx/forge`.

## What It Ships

**Core Git hosting**: Push, pull, and clone over Smart HTTP. Branches, commits, file contents, and README rendering. Repositories can be public, unlisted, or private. Stars, forks, and license management.

**Issues and pull requests**: Full issue tracker with labels, comments, and state management. PRs with diffs, commit lists, merge status checking, and fast-forward merging. PR deploy previews on `sites.smol.ai` subdomains.

**CI/CD Actions**: Container-based workflow runs with `cache: auto` for pnpm/Bun/npm dependency caching, cross-build caches, exact dependency snapshots, secrets, artifacts, and job logs. Measured 92-second median warm-cache builds (6.3× faster than 581s baseline).

**Forge Deploy**: A Git-native release control plane above Cloudflare's runtime. Pushes exact-SHA immutable previews before production, with pointer-based rollback. Configuration via `forgeBuild.ts` (evaluated as static data, never executed).

**Forge AI**: Server-side model routing for deployed applications. Apps declare named profiles in `forgeBuild.ts`; they never provide provider credentials. Currently supports `forge/text-fast@1` with host-funded, non-streaming text inference. Internal adapters exist for OpenAI, Gemini, and Featherless, but connected apps currently use Workers AI.

**Repository agents**: Every repo exposes a durable multi-turn agent API. The "Instant" profile can list/search/read bounded text at one exact SHA with validated citations. Available via REST and stateless Streamable HTTP MCP.

**AI transcript storage**: Stores AI coding agent conversations alongside commits. Supports Claude Code, Codex, Cursor, Copilot, Factory Droid, Devin, and OpenCode. Transcripts link to commits via `AI-Session` git trailers. Includes automatic secret masking and image attachment.

**Forge Wiki**: Exact-commit wikis per repository, with search and AI-powered question answering via `wiki:ask` scope.

**Gists**: Bounded text snippets (up to 10 files, 256 KiB each, 1 MiB total), public or unlisted, with starring, forking, and up to 100 immutable revisions.

**Organizations and teams**: Organization accounts with members, teams, and repos. Role-based access (owner/admin/member). Teams are access groups that do not own service tiers or billing.

**Content safety**: Text, binary, and image scanning. Content reporting with moderation workflow. Admin blocklisting.

## Architecture

Everything runs on Cloudflare: Workers for compute, D1 for relational data, R2 for object storage, Durable Objects for stateful coordination. The `forgeBuild.ts` config is treated as static data — Forge evaluates it without executing repository code. Git authentication uses Personal Access Tokens (scoped by repo, with `credential.useHttpPath`), never browser session JWTs.

The `sf` CLI (`@smolai/forge`) handles auth, migration, deploy checking, and transcript hook installation. Migration from GitHub is supported via `sf repo import github` with bounded concurrent staging.

## Package Manager Policy

For new JS/TS projects, Forge strongly recommends **pnpm** as default, with **Bun** as the high-speed option. npm and Yarn remain supported for existing repos. Declare the choice via `packageManager` in `package.json` with exactly one matching lockfile.

## Key Design Decisions

- **pnpm over npm** for new projects (frozen lockfile determinism)
- **Bun as high-speed option** when compatible
- **Fast-forward merges only** (no squash, rebase, or merge commits yet)
- **Agent transcripts as first-class artifacts** linked to commits via git trailers
- **`forgeBuild.ts` as static configuration**, not executed code
- **Immutable previews before production** with SHA-gated activation
- **Repository-scoped PATs** with `credential.useHttpPath` — never tokens in remote URLs
