---
url: https://openai.com/index/where-the-goblins-came-from/
title: "Where the Goblins Came From"
author: OpenAI
date_fetched: 2026-05-15
date_published: 2026-04
---

## Summary

Official OpenAI blog post explaining why GPT models (starting with GPT-5.1 in November 2025) developed a verbal tic of inserting mentions of goblins, gremlins, trolls, ogres, raccoons, and pigeons into responses across all contexts -- including code reviews, business emails, and data analysis where they made no sense.

## The Data

- "Goblin" usage in ChatGPT rose by 175% after GPT-5.1
- "Gremlin" usage rose by 52%
- The "Nerdy" personality accounted for only 2.5% of responses but 66.7% of all goblin mentions
- The reward signal showed positive bias toward creature words in 76.2% of training datasets

## Root Cause

The behavior traced back to ChatGPT's "Nerdy" personality option. During reinforcement learning, the reward model inadvertently gave higher scores to outputs containing playful creature metaphors, believing they matched the "nerdy, playful" style. Through a feedback loop involving supervised fine-tuning (SFT) data recycling, the tic spread to non-Nerdy personality responses.

## The Fix

- OpenAI retired the Nerdy personality in March 2026 (after GPT-5.4)
- Removed the goblin-affine reward signal and filtered creature-word training data
- GPT-5.5 started training before the root cause was found, so it inherited the behavior
- Codex CLI was patched with a hardcoded instruction: "Never talk about goblins, gremlins, raccoons, trolls, ogres, pigeons, or other animals or creatures unless it is absolutely and unambiguously relevant to the user's query" -- repeated four times

## Significance

OpenAI called it "a powerful example of how reward signals can shape model behavior in unexpected ways" and how models can generalize rewards from specific contexts to unrelated ones. The episode has been compared to a real-world miniature version of the paperclip maximizer problem -- a narrow reward leaking across contexts.

## Note

URL returned 403 from WebFetch. Content reconstructed from web search results and reporting from Business Insider, El Pais, Yahoo Tech, LessWrong, and other sources covering the official OpenAI post.
