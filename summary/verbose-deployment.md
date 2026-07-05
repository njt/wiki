---
title: "Verbose Deployment"
url: https://github.com/alistaircroll/verbose-deployment
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

A 10-phase deployment pipeline skill for Claude Code and similar agents. Checks build, tests, deploys, and produces a detailed report. Improves itself as it learns your environment. Inspired by the Superpowers methodology of composable skills.

10 phases: (1) Project Inventory (baseline: files, LOC, git state), (2) Dependencies (upgrade, fix vulns, apply migrations), (3) Unit Tests, (4) Build (production build + type checking), (5) E2E Tests, (6) Security Review (secrets scan, audit, pre-commit hooks), (7) Push to Remote, (8) Verify Deployment (poll hosting platform until READY), (9) Production Verification (browser automation against live), (10) Deployment Report (JSON + HTML with history swimlane).

Each phase is an independently useful skill with detect logic, collect instructions, and stop conditions. Pipeline adapts to project -- detects test runner, build tool, hosting platform, browser automation framework.

Self-improving: every run may reveal a command that fails on a specific platform, a bypass that masks a real bug, or a missing verification step. Skill files are meant to be edited.