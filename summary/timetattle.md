---
url: https://github.com/bensyverson/timetattle
title: "Timetattle"
author: Ben Syverson
date_fetched: 2026-11-26
date_published: 2026 (pre-launch, early build)
topics:
  - personal-agents
  - developer-tools
---

Timetattle is a macOS Go CLI (installed via Homebrew, run hourly by a launchd agent) that reconstructs *when* work happened and *for whom* from the traces work leaves behind: Claude Code transcripts, git history and reflogs, task databases, file modification times, and macOS presence data (knowledgeC backlight, Biome ScreenTime.AppUsage). It was built after a fixed-price contract budgeted at 40 hours consumed ~50 and the overrun was only detectable three weeks later by combining four evidence sources with memory.

The design rests on a strict separation: observations are immutable and never edited; inference (clustering observations into "sittings", attributing them to projects and clients) is derived, disposable and re-runnable; and human corrections are an overlay keyed to wall-clock spans, never rows in the evidence. Every report separates what was *observed* from what was *inferred* — on the live corpus, 67.3% of presence and 85.3% of engaged hours are clustering margin, not observation — and the tool is explicitly not a timesheet or system of record.

Two metrics are kept deliberately separate: **engaged** time (per-project, may overlap, drives billing) and **presence** (wall clock, never double-counted, includes a coding agent running with the lid shut). Duration is always inferred by clustering *across* sources, never interpolated within one — naive birth-to-modification spans claimed 70+ hours on a 13.9-hour day. Everything is local; the privacy posture is whitelist-only capture (one named Biome stream out of 153) with a test that fails if the list widens.

The codebase (~60k lines of Go, plus docs) is organised as one file per subcommand (harvest, register, status, sessionize, chains, metrics, fit, adjust, review, dashboard, orient) over an internal package per evidence source and concern: `internal/store` (the immutable event store, SQLite), `internal/sessions` (the clustering), `internal/transcripts` (Claude Code jsonl reading), `internal/biome`, `internal/knowledgec`, `internal/fit` (parameter search against human labels), `internal/manifest` (declarative JSON manifests for new file-based sources).
