# Slop Score

EQ-Bench's quantitative metric for measuring how much a text exhibits AI-typical writing patterns — the "slop smell" that readers feel before they can articulate. Higher scores mean more stereotypically AI-like prose, measured through word frequency, trigram patterns, and rhetorical structures over-represented in LLM outputs versus human writing.

---

## Methodology

The Slop Score is a weighted composite of three signals, normalized across all models on the leaderboard:

- **60% Slop Words** — individual words occurring unnaturally often in LLM outputs (per 1,000 words)
- **25% Not-x-but-y Patterns** — contrast constructions like "not just X, but Y" that are AI over-tells (per 1,000 characters)
- **15% Slop Trigrams** — three-word phrases over-represented in AI text (per 1,000 words)

The word and trigram lists come from the `slop-forensics` toolkit, which compares outputs from 10 language models on essay and creative writing prompts against human-authored text. The tool is explicitly **not an AI detector** — it identifies over-used patterns rather than classifying text. This distinction matters: a low slop score doesn't mean human-written, just that the writing doesn't smell like the AI default.

The leaderboard uses standardized prompts across models, with 150 samples per model. The site also offers a text analysis tool where you paste writing and get a slop score with detailed breakdowns.

---

## Leaderboard Highlights

The leaderboard reveals sharp differences in how models write:

| Model | Slop Score |
|-------|-----------|
| Gemma 3 4B | 85.2 (worst) |
| Gemini 2.5 Flash | 77.6 |
| DeepSeek R1 | 67.9 |
| ChatGPT-4o latest | 47.7 |
| GPT-5 Nano | 34.1 |
| OpenAI o3 | 31.8 |
| Claude Sonnet 4 (May 2025) | 27.3 |
| Claude Sonnet 4.5 | 19.5 |
| Human baseline | 10.4 |

The spread is enormous — Gemma 3 4B is over 8× the human baseline, while Claude Sonnet 4.5 is within 2×. The Google family (Gemma, Gemini) dominates the high-slop end. Anthropic's models are the clear winners at the low-slop end among frontier models, which makes intuitive sense to anyone who's read Claude's prose side-by-side with ChatGPT's.

A fascinating datapoint: `sam-paech/gemma-3-27b-it-antislop` scores 20.8 — nearly identical to Claude Sonnet 4.5. This is a fine-tuned Gemma model that the benchmark creator himself modified to reduce slop, validating that the metric measures something trainable, not just model-family destiny.

---

## The Slop Lexicon

The word list is a rogues' gallery of AI creative writing tics. Here are the top offenders:

> silas, unsettling, elias, profound, subtly, meticulously, relentless, scent, flicker, echoes, blackwood, insistent, elara, frantic, thorne, clung, shimmering, faint, intricate, shadows, profoundly, perpetually, fleeting, lyra, glow, agonizing, obsidian, unwavering, whispered, whisper, gaze, undeniably, fueled, swirling, whispers, etched, suffocating, tapestry, dread

The trigrams are even more damning because they capture *narrative gestures*, not just word choice:

> something else entirely, small intricately carved, intricately carved wooden, faint almost imperceptible, young woman named, air hung thick, dust motes danced, resonated deep within, voice barely audible

What's striking is how many of these come from genre fiction — fantasy names (Silas, Elara, Lyra, Thorne, Blackwood), gothic atmospherics (shadows, flicker, obsidian, dread, clung, suffocating), and the sensory-overload prose style that workshop fiction prizes (meticulously, profound, shimmering, tapestry). These aren't random artifacts; they're the ghost of a specific kind of writing that dominates the training corpus.

The "not-x-but-y" pattern list is the most revealing. It's not a word; it's a *thought structure* — the compulsion to undercut every assertion with a more nuanced one, to perform intellectual depth through negation. A human writer does this occasionally. An AI does it constantly, and the effect is exhausting:

> "It wasn't a smile of defiance, not in the aggressive sense. It was a smile of quiet understanding."
>
> "The silence wasn't a void; it was a space for healing."

Every one of these follows the same cadence: assertion → negation → more-profound-alternative. It's the rhetorical equivalent of a magic trick where you can see the wires.

---

## Key Themes

#tool for benchmarking AI writing style quantitatively rather than vibes-based

#concept for "slop" as a measurable property of text — distinct from "AI detection" and more honest about what's being measured

#pattern in the "not-x-but-y" rhetorical structure as perhaps the single most reliable AI tell in creative writing

#comparison across model families reveals genuine stylistic differences: Google models are the sloppiest, Anthropic models the cleanest, with a 4× spread between best and worst

---

## Critical Analysis

### What This Gets Right

**Transparency is the killer feature.** Unlike black-box AI detectors that return a probability with no explanation, Slop Score publishes the exact word lists, trigram lists, and formula. You can check its work. When it flags "profound" as a slop word, you can decide whether that's fair or whether your genre naturally uses that word. This is how all AI metrics should work — *show your cards*.

**The "not an AI detector" framing is the right posture.** Most "AI detection" tools are snake oil — unreliable, biased against non-native English speakers, and sold with false confidence. Slop Score sidesteps this entirely by measuring something different: not "was this written by AI?" but "does this writing exhibit AI-stereotypical patterns?" The difference is subtle but crucial. You can write with AI and score low (Claude's outputs do); you can write without AI and score high (if your style happens to overlap with slop words). The tool measures a stylistic fingerprint, not authorship.

**The benchmark exposes model "voices."** The human baseline at 10.4 is fascinating — it's not zero. Some slop detection is noise, and some humans legitimately write in ways that overlap with AI patterns. This is the benchmark's honesty on display. The wide spread between models also confirms something writers have felt intuitively: different models produce prose with different *voices*, not just different levels of factual accuracy.

### Where It Falls Short

**Genre confound.** The slop word list is heavily weighted toward fantasy and literary fiction vocabulary. If you're writing in those genres, your slop score will be inflated regardless of authorship. A fantasy writer using "obsidian" and "shimmering" descriptively isn't writing slop — they're writing fantasy. The benchmark conflates "genre convention" with "AI artifact" because the training data that created the AI's voice was, itself, genre fiction.

**Creative writing only.** The tool is explicit about this — it's optimized for creative writing and essays — but the leaderboard will inevitably be used to make claims about general writing quality. Technical documentation, business prose, and academic writing have entirely different stylistic norms. A low slop score in creative writing tells you nothing about how the model handles a spec document.

**Style isn't quality.** This is the category error at the heart of the project. "Not sounding like AI" is not the same as "writing well." Human authors produce bad prose, repetitive structures, and clichéd patterns all the time. The metric rewards models that sound *human-like*, and human-like includes boring, clumsy, and derivative. Claude's low score might mean it writes well, or it might mean it defaults to a register that happens to avoid the slop word list — a compositional accident, not a virtue.

**The anti-slop fine-tune raises uncomfortable questions.** The `gemma-3-27b-it-antislop` model proves you can fine-tune to lower your slop score. If slop becomes a widely-used metric, expect model providers to optimize against it directly — teaching models to avoid specific words and patterns without necessarily improving the *quality* of the prose. This is the Goodhart's Law problem: when a measure becomes a target, it ceases to be a good measure. We've seen this movie with every benchmark in AI, and slop score won't be different.

### The Meta Point

The most valuable thing about Slop Score isn't the numbers — it's the word lists. Reading through the slop words and trigrams is an education in AI's default voice: overwrought sensory detail, fantasy-name-dropping, the compulsive "not X, but Y" structure. Once you see these patterns, you can't un-see them. The benchmark is a diagnostic tool for developing taste, not just a leaderboard. In that sense, it's more useful to human writers trying to clean up their own prose than to model evaluators comparing APIs.

---

*Sources: [[raw/slop-score]]*
*Last updated: 2026-07-05*
