---
url: https://interfaze.ai/blog/interfaze-a-new-model-architecture-built-for-high-accuracy-at-scale
hn_url: https://news.ycombinator.com/item?id=48097078
title: "Interfaze: A new model architecture built for high accuracy at scale"
author: Interfaze (yoeven on HN)
date_fetched: 2026-05-15
date_published: 2026-05-11
hn_score: 163
hn_descendants: 43
topics:
  - ai-research-and-models
---

# Interfaze: A new model architecture built for high accuracy at scale

## Source: Interfaze blog post

Interfaze is a hybrid model architecture merging DNN/CNN models with omni-transformers. The core premise: traditional LLMs are error-prone on deterministic tasks, while older DNN architectures (CNNs, CRNN-CTC) are highly accurate for specific jobs but inflexible. Interfaze combines both.

### Architecture

1. DNN/CNN encoders — task-specific deep neural networks for OCR, object detection, audio
2. A transformer decoder — reasoning/generalization layer
3. Task-specific adapters — modular components routing to different subnetworks via `<task>` tags
4. Built-in infrastructure — web index, scraping, code sandbox

The heterogeneous design allows partial model activation: `<task>ocr</task>` in the system prompt routes through only relevant subnetworks.

### Specs
- Context window: 1M tokens
- Max output: 32k tokens
- Input modalities: Text, Images, Audio, File
- Reasoning: Available (default off)

### Benchmarks (Interfaze vs. best competitor)

| Benchmark | Interfaze | Best Competitor |
|-----------|-----------|-----------------|
| OCRBench V2 | 70.7% | 55.8% (Gemini) |
| olmOCR | 85.7% | 81.9% (Grok) |
| RefCOCO | 82.1% | 75.5% (Claude) |
| VoxPopuli (WER) | 2.4% | 4.0% (Gemini) |
| Spider 2.0-Lite | 52.9% | 49.6% (Claude) |
| GPQA Diamond | 89.9% | 89.9% (Claude, tied) |
| MMMLU | 90.9% | 89.7% (Grok) |
| MMMU-Pro | 71.1% | 68.7% (Grok) |
| SOB Value Acc | 79.5% | 78.4% (Grok) |

Interfaze leads in 8/9 benchmarks.

### Pricing
$1.50/M input tokens, $3.50/M output tokens. OpenAI Chat Completions-compatible API at api.interfaze.ai/v1.

### Design Intent
Not meant to replace generalist LLMs (Claude Opus 4.7, GPT-5.5). Positioned as a specialist for deterministic tasks at lower cost and higher accuracy for high-volume production workloads.

---

## HN Discussion (163 points, 43 comments)

The HN thread is a snapshot of real-world testing on a launch-day model. Key themes:

### Positive signals
- **schanz**: OCR'd a typewritten DIN A4 page with distortion and inline corrections — "by far the most accurate so far." Cost was ~$0.25/page at full mode; "only OCR" mode cut cost 3x but degraded quality.
- **wood_spirit**: Interested in code extraction/manipulation use cases.
- **sareiodata**: Smaller models struggle with structured output; this could be valuable if it delivers.

### Skepticism
- **gok**: Accused them of "cheating" by comparing a specialized model's MMLU against general models.
- **icemaze**: Tested STT, found "it's worse than whisper."
- **nickserv**: Structured data extraction from images took 20-25 seconds for 5 fields — "unusable at scale." Pricier than Flash-Light.
- **woadwarrior01**: Noted the first detailed positive review came from a brand-new account created ~5 hours after the post — suspected astroturfing (schanz denied affiliation).

### yoeven (Interfaze rep) responses
- No fine-tuning on roadmap; "in most cases it should work out of the box"
- Code extraction not tested; code manipulation won't beat Claude Opus for code tasks
- On-prem deployment available for enterprises in certain regions
- Working on speed optimization; quality was the priority first
- Distinguished from Large Action Models: Interfaze focuses on "deterministic outputs that require high accuracy" where there's one right answer

### Standout exchange
**euroderf**: "Can models be chained like UNIX command-line programs?" — called this "so, so intuitive." This is the most interesting architectural question the thread surfaces and nobody really answers it. The `<task>` tag routing hints at composability but the API is currently monolithic.
