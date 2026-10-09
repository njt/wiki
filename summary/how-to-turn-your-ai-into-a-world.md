---
url: https://www.lennysnewsletter.com/p/how-to-turn-your-ai-into-a-world
title: "How to Turn Your AI Into a World-Class Designer"
author: Anshu Chimala (guest post on Lenny's Newsletter)
date_fetched: 2026-10-10
date_published: 2026-10-06
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Anshu Chimala, who led R&D design teams at Apple for 12 years, argues that LLMs are bad designers not because they lack capability but because next-token prediction plus human-feedback training makes them choose the most predictable option at every step — "design by committee." The fix is to push the model beyond its defaults with an explicit process he adapts from the Double Diamond: Discover, Define, Deliver.

In **Discover**, go broad before deep. Two techniques: (1) seed strings — have the model generate a random alphanumeric string (via shell) and derive the creative direction from it, importing true randomness from outside the model since it cannot act randomly itself (String Seed of Thought, from Sakana AI); (2) much more ambitious prompts where you inject your own taste, developed by iterating on deliberately vague AI idea lists, reacting to visualized directions, and steering — ending with prompts that sound terrible but often work.

In **Define**, give the design a personality. The core technique is a design-critic subagent: a cheap model implements while a big model (e.g. Claude Fable 5) reviews screenshots in a fresh context, scores against a studio-quality bar, and the implementer loops until the critic independently says 9/10 — the critic accounts for under 10% of tokens. Criteria must be objective (rank against professional examples, penalize obviously-AI patterns), with example images as moodboard, careful stopping criteria, and the right model per role. Then enrich with image generation (gradients and shapes are AI giveaways) and video generation via fal.ai for looping chroma-keyed animated graphics and fluid scroll-scrubbed transitions between product states.

In **Deliver**, polish by subtraction: AI adds but rarely removes, and deleting is risky for a risk-averse model, so the human must push — strip gradients, glows, custom controls that look worse than native components, and redundant labels. His calorie-tracker redesign went from cluttered glow to an image-centric, Apple-native minimalist grid, "much better" because it lets the visuals speak.
