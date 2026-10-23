# Humanizer 12B

A 12B Gemma-4 fine-tune that rewrites AI-generated drafts so they read human-written, running locally — notable less for what it does than for how honestly it does it: no detector in the training loop, per-file quality tables, byte-exact prompt specs, and a published dataset.

---

humanizer (jialinyyzz) is a 12B model fine-tuned from google/gemma-4-12B under Apache 2.0, with a single narrow job: take an AI-written draft and rewrite it to read like a person, keeping every number, unit, date, name and quote intact. It runs entirely locally, from a 3.9 GB 2-bit GGUF on 8 GB machines to full bf16. Evaluation: 95% of English rewrites judged human by Originality.ai at its strictest setting, 376/420 with no factual problem from a strict LLM judge — and, unusually for this genre of project, no AI detector anywhere in the training pipeline.

## How it was built

Three stages, all disclosed: 28,598 SFT pairs where the AI side is a frontier-model draft and the human side is *always real human writing*; 3,918 DPO pairs chosen only on fact fidelity and copying; and three GRPO rounds (500 steps) with an LLM fact-judge reward plus a penalty on 5-gram and syntactic-skeleton reuse. The key design point:

> **No AI detector was used anywhere in training**: not as a reward, not as a filter, not to pick a checkpoint.

That's the interesting methodological bet. Optimising against a detector is Goodhart waiting to happen — you'd learn detector-satisfying prose, not human prose. Instead they optimise against a fact judge and a copying penalty, and use the detector purely as an external audit. Every training row is published as a dataset, which is more openness than most lab blog posts manage.

## The engineering discipline

> Reproduce it byte for byte: the first 16 hex characters of `sha256(prompt for the draft "X")` must be `cc51d66b4c593fbe`.

Text completion, not chat: one fixed instruction, temperature 1.0 / top-p 0.95, stop on EOS only — and the card warns that llama.cpp's *defaults* (top-k 40, min-p 0.05) silently break it. The quantisation table carries per-file fact-check and detector numbers rather than just size, and QAT + distillation lifts a 2-bit build from 70% to 87% agreement with bf16. Even the card's limitations section is unusually candid: "Detector results change over time. Nothing here guarantees any detector outcome."

## What it means for the slop problem

This is the strongest counterpoint yet to the "you can't fix AI slop, only humans can" position. Where [[Fixing LLM Writing with Distribution Fine Tuning]] argues the fix is distribution-level post-training but offers no verifiable artifact, humanizer ships open weights, a published dataset, and a self-contained evaluation — and outperforms a prompting-only baseline (the popular `blader/humanizer` skill) 4/60 vs 60/60 on detector flags. It also nuances [[Slop Score]]: that metric measures how much text *smells* AI-like, while this project attacks the underlying distribution directly and measures the outcome with an external detector instead.

The honest framing for practitioners: this is a post-processing pass, not an author. The failure modes it admits — 28/420 dropped a format element, occasional wrong-language answers, slang drift in casual genres — mean it slots in as a verification-bearing step after drafting, which fits the pattern in [[LLM-as-a-Verifier]] inverted: here a judge model gates the output, and the human still reads numbers, dates and names before sending.

## Fair criticisms

- Detector numbers are a snapshot: Originality.ai v3 on one date. The authors say so, but the headline "95%" will still be quoted as a stable property.
- The fact judge is one model voting once; the card's own audit (Opus re-reading flags) shows ~27% of "serious" flags were minor or non-errors, so both clean and dirty numbers have error bars.
- The genre most people care about — templated social posts — is still the weakest (3/16 flagged, and 0/60 for the prompting baseline implies the skill comparison favoured the easiest genres less than the headline suggests).
- Verifying your setup requires byte-for-byte prompt reproduction with a sha256 check, which is admirably rigorous and also a symptom of how brittle single-checkpoint models are: change one newline and the magic degrades.

## Tags

#tool #concept #ai-research #local-inference

---
*Sources: [[raw/humanizer]], [[summary/humanizer]]*
*Last updated: 2026-10-24*
