---
url: https://www.404media.co/hack-reveals-suno-ai-music-generator-scraped-youtube-deezer-and-genius/
title: "Hack Reveals Suno AI Music Generator Scraped YouTube, Deezer, and Genius"
author: Jason Koebler
date_fetched: 2026-07-25
date_published: 2026-07-15
site: 404 Media
---

A hacker breached Suno, the AI music generation tool, and shared stolen data with 404 Media. The breach exposed source code revealing the company's training data sources and also compromised user information for "hundreds of thousands" of Suno customers along with Stripe payment data.

The hacked source code—appearing to date from 2023 and 2024—contains scraping instructions revealing these specific data sources:

- **YouTube Music** — One file showed it had ingested over 2 million music clips, comprising roughly 113,879 hours of audio. This confirms RIAA accusations that Suno "ripped songs directly from YouTube."
- **Deezer** — Approximately 12,287 hours of music were scraped.
- **Genius** — Referenced as "genius_hq," accounting for about 17,615 hours of content.
- **Other sources** included Pond5 (62,117 hours), Jamendo (3,726 hours), Freesound (410 hours), the International Music Score Library Project (19,514 hours), Musescore lyrics (103 hours), and podcasts via RSS feeds.

A code comment indicated the pipeline would pull from "genius_hq, youtube_music, freesound, jamendo, imp, deezer, ytm_tagged" and noted that "non-music will be filtered out."

In total, the documented datasets amount to "at least decades worth of music."

Suno faces multiple major lawsuits from the record industry. The company previously acknowledged it trained on "essentially all music files of reasonable quality that are accessible on the open internet," totaling tens of millions of recordings. Suno has argued this constitutes fair use; one suit has already been settled. The article notes the hacked data is "a rare look at exactly how AI models and tools are built."
