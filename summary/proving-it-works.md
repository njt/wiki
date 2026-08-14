---
url: https://github.com/prime-radiant-inc/proving-it-works
title: Proving It Works
author: Prime Radiant, Inc. (Jesse Vincent)
date_fetched: 2026-08-14
date_published: 2026-08-12
---

# Proving It Works

A Claude Code plugin (MIT, by Jesse Vincent's Prime Radiant) for recording a narrated movie that proves software actually works — and for catching the defects that make such movies worthless before you hand one to anyone.

## What it is

A movie is evidence, and every way it fails is silent: no crash, no red text, just an artifact that looks fine to its maker and is obviously broken to the first viewer. The plugin was born from a measured failure — an agent that passed every per-frame check and still shipped a movie whose picture froze three seconds in while the narration talked for another twenty. Per-frame verification can't see a defect that lives *between* frames.

The skill `proving-it-works-with-a-movie` covers four routes: browser-driven motion, a filmed terminal, composited stills, and a reel rendered from a run's own log. Five Python scripts form the pipeline: `narrate` (one voice clip per scene, cloud or local Piper voice), `assemble` (scenes into a cut, each held to max(narration, visuals)), `make-subtitles` (an SRT timed to the measured clips), `burn-subtitles` (into the picture where libass exists, a soft track otherwise), and `check-movie` (the gate).

## The gate

`check-movie` samples picture and sound on one timeline and fails a movie when the action is crammed into the opening seconds while narration continues over a frozen picture, when the picture never changes, when the audio is silent, or when a narrated movie lacks subtitles (or its subtitles quit early). It writes a contact sheet the maker is expected to actually look at. It cannot tell you a movie is *right* — only that it isn't obviously broken.

The narration path has its own gate: `narrate` listens back to each clip with a local ASR and flags *missing or invented content* (not mispronunciations), because a dropped word scores more similar than two mangled ones.

## Requirements and packaging

Needs `ffmpeg`/`ffprobe`, `uv`, Chrome plus a driver for browser routes, and any TTS. It runs on a dozen harnesses (Claude Code, Cursor, Codex, Devin, Kimi, Gemini CLI, OpenCode, Pi, Hermes, and Agent Plugins 1.0 clients) — each reads the same skill from `skills/`. The stills and log-reel routes are adapted from `obra/superpowers` PR #1931 (MIT).
