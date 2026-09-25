---
url: https://github.com/Geocodio/yak
title: "Yak — autonomous coding agent, PR reviewer, and preview server"
author: Geocodio
date_fetched: 2026-09-25
date_published: 2026 (ongoing; repo commits through 2026-09)
topics:
  - coding-agents-and-frameworks
  - ai-code-review
---

Yak is Geocodio's open-source (MIT) agent platform: one Laravel 13 + Inertia 3 app that runs three workflows — an autonomous coding agent for papercut fixes, a line-by-line PR reviewer, and per-branch preview deployments — all on a shared substrate of Incus system containers backed by ZFS copy-on-write snapshots, one GitHub App, one dashboard, one cost model.

Tasks arrive from Slack, Linear, Sentry, or GitHub webhooks, are classified by a cheap "routing layer" (Haiku/Sonnet via the Anthropic API), then implemented by Claude Code CLI running Opus headlessly (`claude -p --dangerously-skip-permissions`) inside a per-task sandbox cloned from a per-repo snapshot in ~2 seconds. Every task follows a fat-enum state machine (`pending → running → awaiting_ci → success`, with clarification and retry branches); Yak pushes branches and creates PRs via the GitHub App, but never merges — the safety boundary is the sandbox plus "no merge authority", not permission prompts.

The architecture doc is unusually candid: two queues (a 4-worker `yak-claude` queue with 600s timeouts and a responsive `default` queue), bounded retries (two attempts max) that resume the original Claude session via `--resume` to avoid re-reading the codebase, per-task ($5) and daily ($50) cost guardrails, and a dedicated video-walkthrough pipeline where the agent scripts shots in `script.json`, a headless linter dry-runs it before capture, and Remotion renders the final cut through a frame-sampling QA gate.
