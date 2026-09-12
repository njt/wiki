---
url: https://news.ycombinator.com/item?id=49373456
title: "Piano Autocomplete — On-Device Music Copilot (Show HN)"
author: unknown (Show HN)
date_fetched: 2026-09-04
date_published: unknown
site: news.ycombinator.com
topics:
  - ai-research-and-models
---

# Piano Autocomplete — On-Device Music Copilot (Show HN)

A Hacker News Show HN post describing a **125M-parameter transformer** that autocompletes piano performances in real time — roughly **108 notes/second on an iPhone 15**, running entirely on-device via Core ML.

The pitch is explicit: **GitHub Copilot or Tabnine for music**. Instead of prompting with code, you prompt by playing a few notes on a MIDI piano, and the model continues what you played. The app is free, and the author offers to answer questions about the model, training, Core ML, or "the many things that didn't work."

The intellectual frame is historical rather than hype-driven. The author points to Robert Gjerdingen's *Gebrauchs-Formulas* — the galant-schema / partimento tradition in which 18th-century composers treated music as a library of reusable patterns — and to a recording of four Russian composers (including Rachmaninoff) playing a pattern-recognition-and-generation game at a dinner party in the late 1800s. Composers of that era could do this from sheet music alone, audiating without a piano.

The post is a compact artifact: one model, one modality, one device, and a claim that next-note prediction is the musical analogue of next-token prediction.
