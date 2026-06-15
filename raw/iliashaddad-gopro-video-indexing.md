---
url: https://iliashaddad.com/blog/i-indexed-669-gb-of-my-gopro-videos-using-my-m1-max-computer
title: "I indexed 669 GB of my GoPro videos using my M1 Max computer and local ML models"
author: Ilias Haddad
date_fetched: 2026-06-15
date_published: 2026-06 (approximate; original URL returned 503, content reconstructed from HN discussion, GitHub repo, and cnblogs analysis)
---

# I indexed 669 GB of my GoPro videos using my M1 Max computer and local ML models

Ilias Haddad is a long-distance cyclist from Morocco who built **edit-mind**, an open-source, local-first video indexing system to make his 2,207 GoPro cycling videos searchable by content.

## The Problem

After years of cycling with a GoPro, Haddad had accumulated 2,207 videos totaling 669 GB and 15+ hours of footage. The files were organized only by filename and date — there was no way to search for specific moments, faces, or scenes without manually scrubbing through everything.

## The Pipeline

1. **Audio extraction** via ffmpeg → **Whisper (large-v3)** for transcription
2. **Frame extraction** at 1fps via ffmpeg → **YOLOv8** (object detection) + **DeepFace** (facial recognition against custom dataset) + **PaddleOCR/Tesseract** (on-screen text) + shot-type classifier (wide/close/selfie)
3. **Qwen2.5-VL-7B-Instruct** generates scene descriptions (captioning) — optional "Advanced mode"
4. **Three parallel embedding types** — text, visual, audio — loaded into local vector database (ChromaDB)
5. **Chat agent** enables natural-language search, can send matched clips to DaVinci Resolve timeline

## Performance

- **Input**: 628 videos processed, 668.68 GB, 15h 13m 18s duration
- **Total compute**: 67h 40m 42s (0.22× realtime)
- **Frames analyzed**: 57,537 (at 1fps, downscaled to 720p)

Stage breakdown:
| Stage | Time | % |
|---|---|---|
| Whisper transcription | 25h 13m | 37.3% |
| Frame analysis (YOLO + DeepFace + OCR + captioning) | 24h 56m | 36.8% |
| Visual embedding | 11h 49m | 17.5% |
| Audio embedding | 4h 18m | 6.4% |
| Scene structuring | 48m | 1.2% |
| Text embedding | 37m | 0.9% |

Whisper + frame analysis consumed 74% of total time. Per-frame processing: ~4.2 seconds.

## Key Engineering Decisions

### Desktop App Over Docker
Docker couldn't access the M1 Max GPU (MPS backend fails inside containers). Native macOS desktop app required for GPU acceleration.

### M1 Max vs. RTX 3060
RTX 3060 (12GB VRAM) was faster for raw compute, but M1 Max's unified memory (32-64GB) could hold all four models simultaneously — something the 12GB card physically cannot do.

### Advanced Mode
Enabling Qwen2.5-VL-7B for scene captioning slows indexing but dramatically improves semantic search for complex queries like "bicycle + mountain pass."

## The Open-Source Project: Edit Mind

- GitHub: [IliasHad/edit-mind](https://github.com/IliasHad/edit-mind) — 1.6k stars, MIT license
- Monorepo: pnpm workspaces (TypeScript 89%, Python 9.8%)
- Stack: React/Vite/Electron frontend, Node.js/Express/BullMQ backend, Python AI layer (OpenCV, PyTorch, Whisper, DeepFace)
- DB: ChromaDB (vectors) + PostgreSQL/Prisma (relational)
- Deployment: Docker Compose + native desktop app
- Latest release: v0.22.0 (2026-05-18)

## HN Discussion Highlights

- DaVinci Resolve 21 has built-in AI IntelliSearch that processes locally (no face tagging yet)
- Adobe Premiere Pro equivalent processes in the cloud
- Multiple similar projects mentioned (Framedex)
- Docker + Apple GPU possible via podman/runkit/vllm-metal (Haddad planned to add support)
- Qwen2.5-VL can analyze multiple frames (5 frames at 720p) to understand actions like falling

## Unresolved Questions

- License ambiguity (GitHub API returns `other` — non-standard SPDX)
- No quantified RTX 3060 speedup multiplier
- No isolated timing for Qwen2.5-VL-7B advanced mode
- DaVinci Resolve integration mechanism not fully explained
- Incremental indexing behavior unclear (reuse existing embeddings or full re-index?)

Source reconstructed from: HN discussion (item 48528029), GitHub repo IliasHad/edit-mind, and Chinese technical analysis on cnblogs.com/ninghg/p/20526919. Original URL returned HTTP 503 on all fetch attempts (2026-06-15).
