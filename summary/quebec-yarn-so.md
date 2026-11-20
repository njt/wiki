---
url: https://quebec.yarn.so/
title: "Yarn Quebec: We built a really good forced aligner"
author: Aman Javeri Kadri
date_fetched: 2026-11-20
date_published: 2026-11-13
topics:
  - software-engineering-craft
  - developer-tools
---

A write-up by Yarn's founding engineer on why they built their own forced aligner. Yarn is a video editor where you (or agents) edit by editing the transcript, so word timings are load-bearing: deleting a word clips its neighbour when the aligner is sloppy. None of WhisperX, ElevenLabs Cue, or the Montreal Forced Aligner was accurate enough — about a fifth of cuts came out wrong even with the best — so they built Quebec, which puts 77% of word edges within 20 ms of a human's label versus 54% for the next best, and nears the ~79% at which two humans agree.

The build is three stacked ideas. First, a "fence": the transcriber (ElevenLabs Scribe) has sloppy timings but is rarely wildly wrong, so MFA's beam search is boxed in to look only within 200 ms of the transcriber's guess, making the search small enough to solve exactly — a custom decoder replaced beam-search pruning, cutting unaligned words from 1-in-12 to 1-in-500. Second, a small boundary model: MFA's acoustic model can't decide where one overlapping sound hands over to the next, so a tiny convolutional net (first two layers of Whisper tiny, 19 MB) looks at 200 ms on each side of each boundary and places it in 5 ms steps, taking within-20ms from 60% to 77%. Third, training data: hand-labelled boundaries are mostly paywalled, so they generated labels with a single well-engineered Gemini call (decoder's guess plus two zoom levels of spectrogram plots plus a page of phonetic rules), reaching 69% within-20ms against human labels where its inputs scored 51% — and the labels helped training as much as human labels did, at under $200 for ~17,000 boundaries.

They also made it fast and cheap: recordings split at pauses into 2-minute chunks run in parallel on Lambdas, and the aligner costs $0.32/hour of audio with a documented no-signup API (an `llms.txt` version of the docs is provided, "Point your agent at this page"). Caveats are stated plainly: English only, built on MFA's acoustic model and Whisper's features rather than trained from scratch, and slower than ElevenLabs on short recordings.
