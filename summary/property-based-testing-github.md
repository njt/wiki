---
url: https://github.com/trailofbits/skills/tree/main/plugins/property-based-testing/skills/property-based-testing
title: "Property-Based Testing Skill (Trail of Bits)"
author: Trail of Bits
date_fetched: 2026-10-18
date_published: 2026 (undated)
topics:
  - software-engineering-craft
  - claude-code
---

Trail of Bits' agent skill for property-based testing (Hypothesis, and Echidna/Medusa for EVM contracts). The skill does four jobs: spotting PBT opportunities (codec pairs, validators, normalizers, pure functions with wide domains, contract invariants), writing property tests (strategy design, edge-case pinning, settings), reviewing existing property tests (tautologies, vacuous `assume()`), and triaging shrunk failures (real bug vs wrong property vs ambiguous spec).

The visible page is largely about the engineering around the skill rather than the skill content itself. Three reference files were deliberately deleted with recorded rationale (syntax like `st.integers` is knowledge models already have; paying context for it crowds out judgment), while `refactoring.md` was restored in trimmed form after an initially wrong cut — with the principle that "a deletion with no rationale is indistinguishable from an oversight." Eval harnesses deliberately live outside the skill directory so the model never reads 900 lines of bash and deliberately-broken fixtures.

The bulk is a careful trigger-rate and effectiveness methodology: 45-session sweeps (~$36) run by hand, a free 9-second `--self-test` in CI to prove the harness still discriminates, and measured parameters (TIMEOUT_S=600 from an observed 449s, TURNS=200 from a truncating cap of 14). Two hard-won methodological corrections stand out: a failed session is not a non-trigger (crashes were being absorbed into the pass floor), and an invocation is final evidence — a Skill call stands regardless of later exit status. An earlier table recording recall 5/7 was corrupted by a classifier that checked exit status *before* checking for a Skill call, discarding 9 of 12 sessions that had in fact invoked the skill. Baseline comparison (`NOPLUGIN=1`) is mandatory because Opus already writes competent Hypothesis suites unaided; effort was swept downward to `low`, which found the planted fixture defect 4/4.
