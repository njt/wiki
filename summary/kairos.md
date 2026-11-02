---
url: https://github.com/finic-ai/kairos
title: "Kairos — open-source, real-time fraud and risk decisioning"
author: Finic
date_fetched: 2026-11-02
date_published: 2026 (ongoing repository)
topics:
  - coding-agents-and-frameworks
  - mcp-and-tool-protocols
---

Kairos is Finic's open-source (Apache-2.0) real-time fraud and risk decisioning platform, positioned as an open alternative to Sardine, Sift, Riskified and SEON. It decides sign-ups, logins, orders, payments and payouts in one call (approve, step_up, review or block) and keeps every decision with the signals behind it. Its claimed differentiators: coding agents tune it over MCP, and "decision models" catch fraud before labels exist.

The repo is a pnpm monorepo: `apps/server` (Hono on Node.js, Postgres via Drizzle — the HTTP API and MCP server), `apps/console` (React/Vite), `apps/cli`, `packages/core` (the decision path, model scorers, and a from-scratch gradient-boosted tree trainer), `packages/sdk`/`sdk-python` clients, `packages/eval` (a cached, replayable evaluation suite), plus templates, examples and docs.

The architecture rests on **Grayson**, Finic's hosted decision model, which answers plain-language yes/no or multiple-choice questions ("Is this email throwaway?") with calibrated probabilities. Actions declare the questions to ask and windows of totals to compute; the server compresses the subject's event history into a context (recent events verbatim, older ones as statistical baselines), asks the questions, applies a threshold policy with deterministic rules on computed totals, and logs everything. On top of that, workflow **experiments** let an agent propose model variants — decision model alone, logistic regression, gradient-boosted trees (XGBoost-compatible JSON, trained in-process in TypeScript), or stacks — which Kairos trains on old labeled data, validates on the next slice, tests on the newest, and reports with ROC-AUC, PR-AUC and bootstrap intervals. The MCP server exposes ~60 grouped tools (read/write/decide/build) that proxy the HTTP API, with prompt-injection warnings and per-key audit trails.
