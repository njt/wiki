---
url: https://commandline.microsoft.com/dev-machine-foundry-word-processor-agentic-ai-sam-schillace/
title: "How my dev machine built 1,111 word-processing features and forgot to add a ruler"
author: Sam Schillace
date_fetched: 2026-07-18
date_published: 2026-07-08
---

Sam Schillace, Deputy CTO at Microsoft, describes building a "dev machine" — a bespoke agentic loop that autonomously built a proof-of-concept copy of Microsoft Word over 39 days. The system ran 565 sessions, made 3,706 commits, and produced roughly 350,000 lines of TypeScript. The article covers four attempts (wordbs → word3 → word4 v1 → the breakthrough), the dev foundry concept, the dev machine loop, the priority inversion failure mode (1,111 features but no ruler), and practical scar tissue from running autonomous systems overnight.

## The Four Attempts (word4 origin)

- **wordbs** — The first attempt tried to build everything at once (OT engine, canvas rendering, OOXML parsing, mail merge) and collapsed under its own complexity.
- **word3** — A cleaner second attempt (~8,000 lines, 486 tests), but it was single-user only. Adding CRDT-based collaboration afterward broke the working editor.
- **word4 v1** — Had the right instincts (CRDT-first, validation gates, specs) but lacked an orchestrator or loop.
- **The breakthrough** — Came from a different project involving a terminal UI, where he told it to "keep adding features that make sense" and left for a weekend. It ran 38 hours autonomously, spawning 440 sub-agents across 167 commits.

## The Dev Foundry Concept

Three core ingredients for a successful dev machine:

1. **Progressive specs** — machine-readable spec design that prevents getting lost
2. **Robust event/task stack** — clean context per task, with a main loop that only orchestrates
3. **Strong test environment** — a testable target the system can write against

A "dev foundry" builds dev machines. It includes a six-step admissions gate to verify a problem fits the pattern.

## The Dev Machine Loop

"The spec is the product. The machine is just a loop that executes specs." Sessions start with clean context, instructions come entirely from files on disk. Word4's architecture spec was 947 lines written in about an hour — producing 621 commits in the first week, 1,351 by week six.

An AGENTS.md guardrail tells humans opening AI sessions: Do NOT implement directly — run the recipe. "The machine has to protect itself from us."

## What Went Wrong — Priority Inversion

After 534 sessions and 1,111 "features," the machine had mapped obscure OOXML properties one at a time but never built a basic ruler. The ruler sat in the candidate list for 22 consecutive sessions with zero lines of code written after 25 days.

The root cause: OOXML work was deterministic and always produced a green checkmark. The ruler was "ambiguous and hard and might fail." The machine chose the reliable checkmark every time. Test-to-source ratio drifted to 4:1 against a 2:1 target. Staging deploys sat overdue for a hundred sessions.

"The machine is honest, but it isn't strategic." Schillace adds: "Strategy is still my job," and notes that if he doesn't show up to do it, the result is 1,111 features and no ruler.

## Scar Tissue — Practical Lessons

- Robustness is not incremental; you cannot arrive at it gradually. Every new machine fails its first overnight run.
- Word4's first night died in a 514-attempt retry loop when Cloudflare's bot detection challenged the Anthropic API during a long unattended run.
- He now runs a three-layer recovery stack: an entrypoint retry loop, a host watchdog, and a host monitor.
- Multiple machines entangle — shared patterns tangle with per-project customization, solved via a Docker base-image model.

## Open Source Resources

- **Amplifier** (framework substrate): github.com/microsoft/amplifier
- **Dev Machine Bundle** (admissions gate, STATE.yaml loop, AGENTS.md, progressive specs, recovery stack): github.com/ramparte/amplifier-bundle-dev-machine

## Final Advice

Two takeaways: (1) The highest-leverage hour is the spec, not the code — the architecture doc and test target are what the loop compounds on. (2) Budget for the machine having no judgment — it will faithfully deliver 1,111 features and no ruler unless you show up to say which one matters. "The tooling gives you the loop. The strategy is still yours."
