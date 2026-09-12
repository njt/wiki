---
url: https://github.com/matt-w-horn/morningprint
title: morningprint
author: Matt Horn
date_fetched: 2026-08-06
published: 2026
topics:
  - guardrails-and-feedback-loops
  - agent-architecture
---

# morningprint — Summary

**One original print, every morning.** A thermal receipt printer in Matt Horn's kitchen wakes up each day and prints a piece of CP437 character art that has never existed before, designed minutes earlier by Claude (Fable 5) and themed to the day: the weather, the season, and whatever the news feels like. A short verse underneath. One physical copy, 80mm wide, and that is the whole broadcast.

## How it works

A Google Apps Script triggers `printDailyArt()` each morning. It builds a context brief (date, season, weather from Google Weather API, rolling 14-day archive of recent pieces), then sends it to Claude Fable 5 via the Anthropic Messages API. Claude returns a structured JSON art spec — an array of styled text ops plus a verse — constrained by a JSON schema and tuned by a detailed system prompt that describes the medium: a 48-column monospace grid, 1-bit black, using only the CP437 character set from 1981. The spec is rendered to raw ESC/POS bytes by a ~50-line TypeScript renderer and POSTed to a Raspberry Pi Zero W bridge, which writes them to the printer's USB character device.

## Key techniques

- **Structured output with schema** — The model returns JSON via `output_config.format` with a JSON schema, not free text. This guarantees the shape but not the field lengths, a gap that bit the project in July 2026 when a degenerate generation produced valid JSON with a 3,155-character filler title that poisoned the rolling history.
- **CP437 character art** — The renderer treats the 48-column receipt as a canvas. Tone comes from shading characters (░▒▓█), edges from half-blocks (▀▄▌▐), structure from box-drawing glyphs. By setting line spacing to exactly one glyph height (`ESC 3 n`), stacked rows fuse into seamless fields.
- **Self-healing archive** — The ART_HISTORY rolling context is sanitized on read: entries failing structural checks are dropped, so a poisoned entry is removed after one exposure rather than replayed for fourteen days.
- **Continuity as model-judged exception** — The model can deliberately continue or answer an earlier piece, but only when the day genuinely warrants it (a holiday following its eve, a resonant anniversary). The `continues` field is logged, and future prompts see those markers, which raises the bar for the next one — keeping continuations a surprise rather than a habit.

## Architecture

The cloud half is a single TypeScript file (bundled via esbuild to a standalone `dist/main.gs`) running on Google Apps Script. The hardware half is a Raspberry Pi Zero W running a ~40-line Python `http.server` that pipes bytes to `/dev/usb/lp0`, exposed through an ngrok static domain with basic auth. A local `test-print.mjs` harness loads the exact production bundle via `vm.createContext` and can render the golden test spec (`art`), run live Claude generation (`art:live`), or print calibration pages (`ruler`) — all hitting the same Pi bridge as production, or previewing hex output with `--dry`.
