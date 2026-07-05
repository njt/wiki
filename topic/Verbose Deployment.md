# Verbose Deployment

Alistair Croll's 10-phase deployment pipeline implemented as composable Claude Code skills. Each phase is independently useful with detect logic, collect instructions, and stop conditions. The pipeline adapts to your project -- it detects your test runner, build tool, hosting platform, and browser automation framework rather than assuming a specific stack.

---

## Key Themes

#devtools #agentic-coding #sre #guardrails

The ten phases:

| # | Phase | What it does |
|---|-------|-------------|
| 1 | Project Inventory | Baseline: files, LOC, recency, git state |
| 2 | Dependencies | Upgrade packages, fix vulns, apply migrations |
| 3 | Unit Tests | Run test suite, collect pass/fail metrics |
| 4 | Build | Production build + type checking |
| 5 | E2E Tests | Integration tests against local services |
| 6 | Security Review | Secrets scan, audit, pre-commit hooks |
| 7 | Push to Remote | Git push to deployment remote |
| 8 | Verify Deployment | Poll hosting platform until READY |
| 9 | Production Verification | Browser automation against live deployment |
| 10 | Deployment Report | JSON + HTML report with history swimlane |

Each phase stops on failure. Fix the issue, restart from Phase 1. The pipeline skips phases that don't apply (no e2e tests? Phase 5 is N/A). Self-improving: skill files are meant to be edited as you discover platform-specific issues.

Inspired by the Superpowers methodology of composable skills (Jesse Vincent), which also influenced [[Trycycle]]. The approach embodies [[The Future of Software Engineering is SRE]] in skill form -- deployment is operations, and this pipeline makes operations systematic rather than heroic.

The JSON + HTML report with history swimlane is a nice touch -- it creates an audit trail that addresses the accountability gap [[Write Only Code]] identifies.

## Critical Analysis

The composable-skills architecture is the right call. Each phase as an independent skill means you can use the security review alone, or the deployment verification alone, without buying into the full pipeline. This follows the Unix philosophy that [[Systems Ideas That Sound Good]] implicitly endorses.

The self-improving claim is aspirational but honest -- "skill files are meant to be edited" is a polite way of saying "this won't work perfectly for your setup out of the box." That's fine; the structure for editing is what matters. The risk is the same as any multi-phase pipeline: phase 1-6 add 30 minutes to every deploy, which teams will skip under pressure.

---
*Sources: [[summary/verbose-deployment]]*
*Last updated: 2026-05-14*