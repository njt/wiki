---
url: https://www.skyvern.com/blog/getting-claude-to-qa-its-own-work/
title: Getting Claude to QA its own work
author: Suchintan Singh
date_fetched: 2026-05-15
date_published: 2026-04-03
source: Skyvern Blog
---

Skyvern built an MCP server with 33 browser tools (sessions, navigation, form filling, data extraction, credential management, workflows) and integrated it with Claude Code so that after every frontend change, Claude would "check its own work by opening the page, looking at the pixels, and running the interactions." Two skills were released: `/qa` (local, ~700 lines) and `/smoke-test` (CI, ~300 lines).

The workflow: read git diff, classify the change (Frontend/Backend/Mixed), identify a validation strategy, run full QA of impacted areas, report results as a PASS/FAIL table, and optionally post evidence to a PR. The one-shot success rate on PRs rose from ~30% to ~70%, and the QA loop was cut in half.

In CI, a GitHub Action triggers on PR, reads the diff, starts the app in a minimal viable environment, runs browser-based smoke tests against affected flows, and posts evidence back to the PR. The system stays narrow: "read the diff, form a hypothesis about what changed, and test only the nearby flows" to avoid flaky E2E sprawl.

Setup: `pip install skyvern && skyvern setup claude-code`

Open source prompts: https://github.com/Skyvern-AI/skyvern/blob/main/skyvern/cli/skills/qa/SKILL.md and .../smoke-test/SKILL.md

Remaining hard problems: keeping existing tests up to date, determining blast radius for mixed frontend/backend diffs, and detecting when an agent-generated test plan is too shallow.
