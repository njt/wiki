---
url: https://moisesai.org/
title: "Moises — AI Music Separation & Creation Platform"
author: Music AI (Geraldo Ramos, CEO)
date_fetched: 2026-05-22
date_published: various (founded ~2019, AI Studio launched 2025-08-20)
fetched_via: web_search + moises.ai/newsroom (WebFetch returned 403 on main domain; product pages fetched successfully)
topics:
  - ai-research-and-models
---

# Moises.ai

Moises is a music AI platform built by Music AI, claiming 65+ million users across 190 countries. Started as a stem separation tool (vocal removal, instrument isolation) and expanded into AI Studio — a browser-based AI music workstation launched August 2025 that generates individual editable stems rather than full mixed songs.

## Core Features

### Stem Separation
- 4-track separation: Vocals, Drums, Bass, Other
- Premium adds: Piano, Strings, Lead & Rhythm Guitar, Acoustic & Electric Guitar, Main & Background Vocals
- Pro adds: Hi-Fi models, Drum Kit Parts (kick, snare, hi-hat, toms, cymbals), Multimedia stems
- Claims to be the only AI capable of separating background vocals from lead vocals
- 4x faster separation speed in current web app
- Automatic Key & BPM detection on upload

### AI Studio (August 2025)
- Browser-based AI music workstation ("simplified DAW")
- Stem-by-stem generation using three conditioning signals:
  1. Audio Context — infers tempo, phrasing, key, structure from user stems
  2. Style/Content Conditioning — audio reference, text prompt, or genre preset
  3. Harmony Adherence — controls strictness of chord/scale matching
- Independent weights per conditioning axis
- Models trained on isolated, high-quality instrumental stems (not mixed recordings)
- AI Voice Conversion with 50+ voices
- AI Auto-Mixing & Mastering
- Full in-browser audio editing

### Additional Tools
- AI Chord Detection (beginner/intermediate/advanced modes, auto-transpose)
- Smart Metronome (auto-syncs to song beat)
- AI Lyric Transcription (English, Spanish, Portuguese, French, Italian)
- Speed Changer (50%+ slowdown without pitch change)
- Pitch Changer (all 12 keys)
- VST Plugin (Pro tier)

## Pricing Tiers

| Plan | Price | Key Limits |
|------|-------|------------|
| Free | $0 | 5 separations/month, 1 min/file, MP3/M4A |
| Premium | ~$2.33-5.99/mo | Unlimited, 20 min/file, WAV export |
| Pro | ~$11.66-29.99/mo | Hi-Fi models, drum parts, 180 min/upload, VST, unlimited AI Studio |

## Recognition
- Google Play Best App for Personal Growth (2021)
- Apple iPad App of the Year (2024)
- Apple Design Awards Finalist (2025)

## Key Quotes

CEO Geraldo Ramos: "AI should empower creators" and positions AI as a "co-creator" bandmate.

"Full-song generators are typically trained on mixed recordings and output a finished stereo file that is hard to edit" — Ramos on the Moises vs. Suno/Udio distinction.

"Moises currently has the only AI that is able to separate background vocals" — from the web app announcement.

"The same foundation that lets us take music apart now lets us build it back up" — on the stem separation → stem generation pipeline.

## Architecture Notes
- Cloud-based processing (no local inference option)
- Browser-based web app + mobile apps (iOS/Android)
- AI Studio generates at the stem level, not full songs — users layer, edit, and mix generated stems
