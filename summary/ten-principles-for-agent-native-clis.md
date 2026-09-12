---
url: https://x.com/trevin/status/2051316002730991795
title: 10 Principles for Agent-Native CLIs
author: Trevin Chow (@trevin)
date_fetched: 2026-05-15
date_published: 2026-05-04
source_type: x.com thread
metrics: 53.4K Views, 233 reposts, 177 likes
blog_version: https://trevinsays.com
topics:
  - agent-architecture
---

Trevin Chow's expanded framework for designing CLIs that agents can use effectively. Builds on his earlier "7 Principles for Agent-Friendly CLIs" (replaced by this post) with practical experience from building his own CLI, Cloudflare's Wrangler rebuild, and HeyGen's CLI launch. Organized into two tiers: Table Stakes (don't break the agent) and Compounding (make the CLI better the more agents use it). The thesis: design for agents first, and humans benefit. Designing for humans first and bolting on agent support is what produces the inconsistent, prompt-prone, stdout-only CLIs these principles correct.

Tier 1 — Table Stakes:
1. Non-interactive by default (--no-input, --yes, --force)
2. Structured, parseable output (always --json, clean stdout/stderr separation)
3. Errors that teach and enumerate (valid set in error message)
4. Safe retries and explicit mutation boundaries (idempotency, --dry-run, job ledger)
5. Bounded responses at every layer (pagination, truncation hints, MCP description budget)

Tier 2 — Compounding:
6. Cross-CLI vocabulary consistency (always get never info, always list never ls, always --force never --skip-confirmations)
7. Three-layer introspection (--help, agent-context JSON, SKILL.md manifest)
8. Async-aware execution (--wait with persistent job ledger)
9. Persistent identity through profiles (profile save/use, agent-context discoverability)
10. Two-way I/O (--deliver stdout/file/webhook, feedback command)

Architecture thesis: Tier 2 is hard to apply by hand, easy to apply mechanically via schema/codegen. Cloudflare's TypeScript schema generating CLI, SDKs, Terraform provider, and MCP server from one source is the load-bearing detail.
