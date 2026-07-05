---
url: https://github.com/alistaircroll/verbose-deployment
date_fetched: 2026-07-05
backfilled: true
---

A 10-phase deployment pipeline that checks your build, tests it, deploys it, and gives you a detailed report. Improves itself as it learns your environment. Inspired by the Superpowers methodology of composable skills.

Each phase is an independently useful skill with detect logic, collect instructions, and stop conditions. The pipeline adapts to your project — it detects your test runner, build tool, hosting platform, and browser automation framework rather than assuming a specific stack.

Executes 10 phases in sequence:

| # | Phase | What it does | 
|---|---|---|
| 1 | Project Inventory | Baseline: files, LOC, recency, git state | 
| 2 | Dependencies | Upgrade all packages, fix vulnerabilities, apply migrations | 
| 3 | Unit Tests | Run the test suite, collect pass/fail metrics | 
| 4 | Build | Production build + type checking (catches what tests miss) | 
| 5 | E2E Tests | Integration tests against local services or emulators | 
| 6 | Security Review | Secrets scan, audit, pre-commit hook installation | 
| 7 | Push to Remote | Git push to the deployment remote | 
| 8 | Verify Deployment | Poll hosting platform until READY (no sleep timers) | 
| 9 | Production Verification | Browser automation against the live deployment | 
| 10 | Deployment Report | JSON + HTML report with history swimlane | 

Each phase stops on failure. Fix the issue, then restart from Phase 1 with a clean slate.

`claude plugin install /path/to/verbose-deployment`Copy the entire `verbose-deployment/` directory to your Claude Code skills location:

```
# As a project skill (per-project)
cp -r verbose-deployment/ /path/to/your/project/.claude/skills/
# As a user skill (global)
cp -r verbose-deployment/ ~/.claude/skills/
```
The orchestrator skill is at `skills/verbose-deployment/SKILL.md`. Claude Code will discover it automatically and invoke sub-skills by reference.

Once installed, invoke the pipeline:

```
/verbose-deployment
```
Or invoke individual phases by name:

```
/project-inventory
/dependencies
/unit-tests
/build
/e2e-tests
/security-review
/push-to-remote
/verify-deployment
/production-verification
/deployment-report
```
```
verbose-deployment/
├── README.md                              # This file
├── LICENSE                                # MIT
├── lessons-learned.md                     # Cross-phase patterns
├── shell-portability.md                   # macOS/Linux command notes
└── skills/
    ├── verbose-deployment/SKILL.md        # Orchestrator (runs all phases)
    ├── project-inventory/SKILL.md         # Phase 1
    ├── dependencies/SKILL.md              # Phase 2
    ├── unit-tests/SKILL.md                # Phase 3
    ├── build/SKILL.md                     # Phase 4
    ├── e2e-tests/SKILL.md                 # Phase 5
    ├── security-review/SKILL.md           # Phase 6
    ├── push-to-remote/SKILL.md            # Phase 7
    ├── verify-deployment/SKILL.md         # Phase 8
    ├── production-verification/SKILL.md   # Phase 9
    └── deployment-report/                 # Phase 10
        ├── SKILL.md                       # Report generation
        ├── json-schema.md                 # JSON v1 schema reference
        └── history-tab.md                 # HISTORY swimlane spec
```
The pipeline detects your project's tools and skips phases that don't apply:

- No e2e tests? Phase 5 is marked N/A.
- No hosting platform? Phase 8 is skipped.
- No smoke tests? Phase 9 does a minimal curl check and flags the gap.

Every pipeline run may reveal a command that fails on a specific platform, a bypass that masks a real bug, or a missing verification step. The skill files are meant to be edited — fix commands, add checks, refine logic. Each run should be more thorough than the last.

- Superpowers methodology — the skill framework this is built on
- `lessons-learned.md`— hard-won patterns about HTTP 200 lies, build tool transforms, and test selector collisions
- `shell-portability.md`— avoid GNU-only commands that break on macOS
