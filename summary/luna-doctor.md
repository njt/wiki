---
url: https://github.com/jesserobbins/luna-doctor
title: "luna-doctor"
author: Jesse Robbins
date_fetched: 2026-11-02
date_published: 2026-10-28
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

`luna-doctor` (v0.3.0-alpha.3) is an offline Python CLI that diagnoses why coding-agent sessions on the LunaRoute gateway stall, hang, or get 429s. LunaRoute bills **concurrent requests, not tokens**, so every symptom is the account's concurrency width being exceeded — but the gateway hides this: it waits, demotes requests to a background tier and holds the stream open with keep-alives while producing nothing, and the harness then kills the stall with an error string of its own making. A grep for 429 finds only part of the picture.

The tool reconstructs every model request from the log stores the machine already has (pi, claude-code, codex, swamp), classifies each outcome against *measured* gateway behaviour (admission deadlines are per-model — 120.2s on glm-5.3, 60.3s on deepseek-4.1-flash — never hardcoded), and attributes each refusal to a wall: "you", "your org", "the gateway", "not concurrency", or "unknown". Commands: `discover` (what log stores exist), `analyze` (window analysis with `--baseline prev` before/after comparison), `explain` (one bad request in forensic detail), `watch` (live incident monitoring), `fix` (per-agent copy-paste config that the tool never applies itself), `chart` (a self-contained SVG stacked-area graph of concurrency), and `analyze --for-issue` (a redacted block for support tickets).

Everything carries provenance labels: `measured`, `declared`, `catalog`, `account`, `configured`, `unknown` — and the tool never borrows a number from one source for another, never sums a local lower bound with an account figure, and never substitutes a default for an unknown. Read-only and offline by default; the only opt-in fetches are `capacity --online` and the model catalog, using the key `lunaroute login` already stored.

The repo also ships an agent skill (`skills/luna-doctor/`): a router SKILL.md plus five playbooks and four catalogues, with invariants like "run the tool before theorising" and "never open concurrent requests of your own to test a theory". That agreement between skill prose and CLI is machine-checked: `tests/test_skill_package.py` parses every subcommand's `--help` and fails if a skill file names a flag the parser doesn't carry.
