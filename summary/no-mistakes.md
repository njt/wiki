---
url: https://github.com/kunchenguid/no-mistakes
title: "no-mistakes: AI-Driven Git Gate for Clean PRs"
author: kunchenguid
date_fetched: 2026-07-11
date_published: 2025
topics:
  - guardrails-and-feedback-loops
---

`no-mistakes` is a Go CLI tool that inserts an AI-driven validation gate between your local repo and its remote. Instead of `git push origin`, you `git push no-mistakes` — the tool's bare-repo proxy runs a disposable worktree pipeline, and only forwards the branch after every check passes.

The pipeline is sequential (nine steps): intent extraction, rebase, review, test, document, lint, push, PR creation, and CI monitoring. Each step produces structured JSON findings with severities and actions — auto-fix safe issues within limits, park risky ones for human approval, or log informational notes. Review auto-fix defaults to zero (the riskiest category requires explicit opt-in).

It supports six native agent backends (Claude, Codex, OpenCode, RovoDev, Pi, Copilot) plus ACP targets, with configurable fallback chains. Structured output parsing handles real-world model messiness via a three-tier strategy: direct JSON, code-fence extraction, and bare-object scanning.

Key architectural choices: SQLite for all durable state (zero setup), git worktree isolation rather than containerization, a trust boundary in config merging that ignores executing fields from pushed-branch `.no-mistakes.yaml`, and durable approval parking that survives daemon restarts. The combined document+lint pass avoids a second cold agent startup by stashing lint findings for later consumption.

Compared to traditional CI (validates after push), `no-mistakes` acts as a pre-push gate. Compared to pure linting, it combines deterministic rules with AI-driven review that catches things rules cannot (missing doc comments, edge-case explanations). The git-proxy pattern is the central insight: no new workflow to learn, just push to a different remote.

---
*Sources: [[raw/no-mistakes]]*
*Last updated: 2026-08-01*
