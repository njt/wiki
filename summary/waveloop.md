---
url: https://neynt.ca/writing/waveloop/
title: "waveloop: what fable left me"
author: neynt
date_fetched: 2026-07-03
date_published: 2026-06
topics:
  - agent-coding-workflow
---

# waveloop: what fable left me

neynt, June 2026

Article about a music visualizer called Waveloop, built during two days of access to "Fable 5" (an AI coding model from Anthropic). The visualizer maps music onto a 12-TET chromatic circle — 30° per semitone, one revolution per octave — with pitch classes stacking as a spiral histogram. Different octave registers are color-coded via an Oklch color spiral: muted blues/greens for bass, fiery orange/red/violet for mid-tones, gold/sky for treble.

Key technical details:
- Built on 12-TET (¹²√2 ratio between successive semitones)
- Intervals readable as angles (m2 = 30°, M2 = 60°, etc.)
- Chord quality discernible from shape; transposing rotates, inversion preserves shape
- Common chord qualities: maj, min, dim, aug, sus4, sus2, dom7, maj7, min7
- Primarily offline (precomputed CQT), but also has live mic mode capable of identifying ukulele chords
- Chord detector uses NOTE_NAMES array, QUALITIES definitions, AGC normalization, scoring against known chord templates (threshold 0.5)
- Trail map stores sqrt(v / EMAX) in alpha with premultiplied RGB
- Radius is a concave function of age, so "material surges off the rim and decelerates"

The author contrasts Fable's code style with previous models. Prior models wrote "like a perfectly reasonable upwardly mobile engineer at a FAANG." Fable wrote more like Terry Davis: dense, technical, "maximally information dense recordings of intent." A lengthy comment block at the top of the generated file shows deeply technical prose referencing alpha premultiplication, CDF, FFT, AGC, and Oklch color theory — both technical and literary, describing how "noise *lingers*" and "material *surges* off the rim."

Explainer video process: three prompts. First attempt "hot garbage." After detailed feedback (atrocious TTS, requesting conversational tone like 3blue1brown, more illustrative visuals), second attempt "a LOT better." Third cleanup for typesetting and consistent voice. Final video "still not a fantastic video" but "engaging enough to capture my attention for all ten minutes."

AI usage disclaimer: "I used Claude to generate svgs for the diagrams. But all prose is mine."

Visualizer URL: saltblock.neynt.ca/waveloop.html

Fable was available for about one week before being removed. Author notes: "We've been without Fable for about week now."
