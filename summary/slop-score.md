---
url: https://eqbench.com/slop-score.html
title: "Slop Score — Emotional Intelligence Benchmarks for LLMs"
author: Sam Paech
date_fetched: 2026-07-05
date_published: 2025
topics:
  - ai-research-and-models
---

# Slop Score

A leaderboard and analysis tool from EQ-Bench that computes how much a given text "smells like AI slop" by measuring the frequency of words, trigrams, and rhetorical patterns over-represented in LLM outputs compared to human writing.

## Methodology

The Slop Score is a weighted composite metric:
- **60% Slop Words**: Frequency of individual words that appear unnaturally often in LLM outputs
- **25% Not-x-but-y Patterns**: Frequency of contrast patterns like "not just X, but Y" overused by AI
- **15% Slop Trigrams**: Frequency of 3-word phrases over-represented in AI text

Slop word and trigram lists are produced using the slop-forensics toolkit, which identifies words/n-grams statistically over-represented in LLM outputs compared to human writing, computed from 10 models' outputs on essay and creative writing prompts.

The tool is explicitly NOT an AI detector — it looks for glaringly over-used patterns rather than classifying text as AI or human.

## Leaderboard (higher = more slop)

| Model | Slop Score |
|-------|-----------|
| google/gemma-3-4b-it | 85.2 |
| google/gemini-2.5-flash | 77.6 |
| google/gemma-3-12b-it | 70.6 |
| google/gemma-3-27b-it | 69.5 |
| deepseek/deepseek-r1 | 67.9 |
| qwen/qwen3-4b | 62.2 |
| openai/chatgpt-4o-latest | 47.7 |
| openai/gpt-5-nano | 34.1 |
| claude-sonnet-4-20250514 | 27.3 |
| claude-sonnet-4-5 | 19.5 |
| human-baseline | 10.4 |

## Key Slop Words

Top words from the slop list: silas, unsettling, elias, profound, subtly, meticulously, relentless, scent, flicker, echoes, blackwood, insistent, elara, frantic, thorne, clung, shimmering, faint, intricate, shadows.

## Key Slop Trigrams

"something else something," "small intricately carved," "intricately carved wooden," "faint almost imperceptible," "something else entirely," "young woman named," "air hung thick," "dust motes danced."

## Additional Metrics

The tool also provides writing style analytics: Flesch-Kincaid grade level, average sentence/paragraph length, dialogue frequency, lexical diversity (MATTR-500), and unique/total word counts.

## Source Code

https://github.com/sam-paech/slop-score
