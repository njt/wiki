---
url: https://arxiv.org/abs/2604.03136
title: "StoryScope: Investigating idiosyncrasies in AI fiction"
author: Jenna Russell, Rishanth Rajendhran, Chau Minh Pham, Mohit Iyyer, John Wieting
date_fetched: 2026-06-22
date_published: 2026-04-03
topics:
  - ai-research-and-models
---

# StoryScope: Investigating idiosyncrasies in AI fiction

Russell et al. (University of Maryland / Google DeepMind) introduce StoryScope, a pipeline that automatically extracts 304 interpretable discourse-level narrative features across 10 dimensions (plot, agents, temporal structure, etc.) from fiction. Applied to a parallel corpus of 10,272 writing prompts each written by a human and five LLMs (Claude, DeepSeek, Gemini, GPT, Kimi) — 61,608 stories total, ~5,000 words each — they show that narrative features alone achieve 93.2% macro-F1 for human vs. AI detection, retaining 97% of performance of models that also include stylistic cues. A compact set of 30 core features captures the signal: AI stories over-explain themes, favor tidy single-track plots, and render emotion through bodies rather than naming it; human stories are temporally complex, morally ambiguous, and acknowledge the reader. Per-model fingerprints emerge (Claude: flat escalation, no dreams; GPT: gossip-driven plots; Gemini: external character description). AI stories cluster in a shared narrative region; human stories are rarer and more dispersed. The features survive stylistic editing (only 1.6pp F1 drop after LAMP artifact removal), suggesting narrative structure is a more durable detection signal than surface prose patterns.

## Key Findings

### Detection performance

- Narrative features alone: 93.2% macro-F1 (human vs. AI binary detection)
- Adding style features: 96.0% macro-F1
- 30 core features alone: 84.8% macro-F1
- 6-way authorship attribution (narrative only): 68.4% macro-F1 (chance: 16.7%)
- Human F1 in 6-way: 88.5% (most distinctive); Claude: 77.1%; Kimi: 55.0% (least distinctive)
- Length doesn't matter: narrative model unchanged (93.2%) after length-matching human/AI stories
- Topic doesn't matter: no significant detection variation across fiction genres
- Robust to stylistic editing: only 1.6pp F1 drop after LAMP artifact removal (93.9% vs 95.5%)

### Core human-AI differences

**AI over-explains themes.** AI stories score ~20% higher on thematic explicitness (1-5 scale). Narrators explicitly state the theme 77% of the time vs. 52% for humans. AI dialogue serves philosophical debate (59% vs. 34%). AI references to other works are vague allusions (72% vs. 50%) rather than specific named references.

**AI renders emotion through bodies.** AI conveys emotion through physical sensations 81% of the time vs. 38% human. AI uses smell-based imagery more (82% vs. 57%), treats setting as psychological mirror more heavily. Human authors use explicit emotion labels 29% of the time vs. just 8% for AI.

**Humans subvert linearity.** Human stories have more time jumps, flashbacks, nonlinear structure. AI favors single-track narratives with tight causal chains (4.20 vs. 3.92 on causality scale), protagonist-driven resolutions (69% vs. 46%), and far fewer subplots (79% "no subplots" vs. 57%). AI resolutions favor internal understanding/acceptance (47% vs. 27%).

**Humans engage the outside world.** Humans reference specific texts/authors at nearly double the AI rate (47% vs. 24%). Humans break the fourth wall more often (67% vs. 39%), address readers directly more (28% vs. 7%). Humans use balanced mix of explicit/implicit references (37% vs. 16%).

**Human stories are rarer and more diverse.** Mean rarity percentile 0.71 human vs. 0.49 AI (Cohen's d = 0.83). 24.7% of human stories fall in the top 10% rarest stories corpus-wide, vs. 7.1% of AI stories. At the prompt level, the human story is the rarest of all six versions 57.8% of the time.

### Per-model fingerprints

- **Claude**: flat event escalation, uniform narrative voice, reverent/continuist approach to literary tradition (62%), favors epilogues, avoids dream sequences. Most distinctive AI model.
- **GPT**: gossip and rumor as plot mechanism (64%), distant retrospective framing, subverts expectations more (41%), ensemble-heavy social networks, ambiguous reconciliations.
- **Gemini**: tidiest endings, extended denouements, bleakest settings (88% tagged bleak/oppressive), protagonist social trajectory expands. Forms confused cluster with DeepSeek and Kimi.
- **DeepSeek**: front-loads crucial context other sources leave until later, emotional expression via behavioral cues, plot-over-atmosphere orientation.
- **Kimi**: fewest fingerprints, lowest F1 — sits at the generic center of the AI distribution with no distinctive narrative choices.

### AI convergence

The five AI models occupy overlapping regions of narrative feature space, well-separated from human stories. Mean human-AI centroid distance is 1.6× the mean AI-AI centroid distance (6.6 vs. 4.3). Even the closest human-AI centroid pair is farther apart than the most distant AI-AI pair. Human stories are more dispersed: mean distance to human centroid is 22% greater than average AI radius.

## Pipeline Architecture

1. **Structured narrative representations**: Each story → JSON template along 10 NarraBench dimensions (Agent, Social Network, Event, Plot, Structure, Setting, Time, Revelation, Perspective, Style). GPT-5.1 extraction with zero-shot prompt + detailed JSON schema. Templates abstract away surface wording so downstream stages reason over narrative content, not style.

2. **Cross-source comparison**: Discovery pool of 600 stories (100 prompts × 6 sources). For each prompt, all six templates presented to GPT-5.1 (high reasoning effort) → structured comparative analysis with per-source dimension notes, cross-source divergences, and executive summary.

3. **Feature discovery**: 10 specialized expert prompts (one per NarraBench dimension) → GPT-5.1 proposes discriminative features as closed-form questions with discrete answer choices. Three discovery runs, union = 408 candidates, deduplicated via embedding clustering (F2LLM-4B, cosine threshold 0.85) → 304 features across 5 response types (124 categorical, 59 ordinal, 45 scale, 44 binary, 32 multi-select).

4. **Feature assignment**: Gemini 3 Flash (minimal thinking) assigns all 304 features to all 61,608 stories. High reliability: Krippendorff's α = 0.88 across 5 independent runs, mean human-model Cohen's κ = 0.84.

5. **Classification**: XGBoost + SHAP for feature importance decomposition. Bootstrap SHAP analysis (B=50 iterations, prompt-level resampling) assigns features to roles: 30 core (binary separation), 75 fingerprint (source-specific), rest excluded.

## Costs and Scale

- Story generation across 5 LLMs: ~$2,800 USD
- Feature extraction over full corpus: ~$1,600 USD
- Total: ~$4,400 USD
- 61,608 stories, ~5,000 words each, 304 features per story
- Code, 10,272 prompts, and 51,336 AI-generated narratives released (human stories withheld for copyright)

## Limitations and Caveats

- Human stories from Books3 (copyright concerns); only AI stories released
- Feature extraction uses LLMs (Gemini 3 Flash) — introduces annotator-model bias
- ~0.70% of generated stories flagged for memorization risk (public domain classics)
- Feature taxonomy induced by LLM, not hand-crafted by narrative theorists
- Detection results may not generalize to highly-edited AI text or future models
- 6-way attribution much harder than binary detection as AI models converge
