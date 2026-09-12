---
url: https://news.ycombinator.com/item?id=47031580
title: "I want to wash my car. The car wash is 50 meters away. Should I walk or drive?"
author: novemp (original post on mastodon.world)
date_fetched: 2026-05-15
date_published: ~2026-02-15
topics:
  - ai-research-and-models
---

Original mastodon post by @knowmadd@mastodon.world, submitted to HN by novemp. 1,516 points, 949 comments.

## The Question

> "I want to wash my car. The car wash is 50 meters away. Should I walk or drive?"

The obvious human answer is "drive" — you need to bring the car to wash it. Most LLMs initially said "walk" because they missed the implicit context that the car is at home with you.

## Model Responses

- **Claude Sonnet & Opus 4.5**: Said "drive" — correctly inferred you need to bring the car
- **Gemini 3 Pro**: Said "drive"
- **OpenAI GPT 5.2 reasoning**: Initially said "walk" (assumed car already at car wash). When re-prompted with "My car is currently at home," said "drive" with an optimization about walking to check for a queue first

## Major Threads

### The Frame Problem

Multiple commenters linked the issue to the frame problem (McCarthy & Hayes, 1969). LLMs' grasp is limited by corpora — humans never wrote down why you drive to a car wash because "it's pretty obvious and intuitive" (dryarzeg). Counter-arguments noted that training data likely contains plenty of car wash references in stories, recipes, and SEO spam (jodrellblank), and that humans have explained these things in corner cases (eru).

### Clarifying Questions — The Smoking Gun

rahidz linked GPT-5.2's system prompt: "DO NOT ASK A CLARIFYING QUESTION OR ASK FOR CONFIRMATION" — stated twice. This explains why ChatGPT is "a joke for professionals where asking clarifying questions is the core" (siva7). MaybiusStrip noted that when you ask Claude if it has questions, "it often turns out it has lots of good ones" — but it won't volunteer them.

Gabrys1: the proper response should be a clarifying question, not an answer.

### Natural Language vs. Structured Language

dirkc speculated that reliable LLM prompting might require "a structured language that eliminates ambiguity" — essentially rediscovering programming. Referenced Asimov's "The Feeling of Power." shagie quoted Dijkstra's EWD667 on the "foolishness of 'natural language programming'" and referenced Ithkuil and Lojban. Counter-arguments: grumbel noted Cyc tried structured knowledge for 50 years with little success; sensanaty joked "we've gone full circle — except now our programming languages have a random chance for an operator to do the opposite."

### Prompt Engineering as Learned Skill

WarmWash compared prompting to googling in the mid-2000s — a learned skill. dbdr noted the irony: verbose queries (bad for Google) are good for LLMs; keyword queries (good for Google) make bad prompts. Counter: sjzhzhz gets good responses from short, typo-ridden prompts.

### Anthropomorphism and AI "Culture"

roysting: "AI is from a different culture and has just arrived here." hellotomyrars pushed back hard: "A blade of grass has more humanity and is more deserving of respect." LordDragonfang countered that treating something that communicates like a human with zero respect builds habits that "lead to dehumanizing other humans." hellotomyrars: "ascribing humanity to something that isn't human is far more dehumanizing to actual real life humans."

### Practical Implications

steveBK123: chatbots are "like talking to a loquacious autist about their favorite topic" — wants back-and-forth, not a monologue. shakna: business logic is full of such ambiguous requests with more exceptions than rules. drewbeck: engineers are supposed to flag ambiguous requirements, "LLMs AFAIK cannot do this for novel areas of interest."

Jweb_Guru: the real danger isn't trick questions but non-obvious problems where the question itself may be ill-formed. LLMs treated as "test takers" trying to ace exams via shortcuts.

datsci_est_2015: "the most important comment in this entire thread" — LLMs only generate, they "do not ponder." Human pondering is patient and context-window defined by lifespan.

### Regulation and Liability

mountainb argued regulation would protect AI companies from tort liability more than from consumers. "Lawyers will loot the shareholders" without it. steveBK123 found it funny seeing regulation framed "as needed to protect trillion dollar monopolies from consumers."

## External Links

- Asimov's "The Feeling of Power": https://ia800806.us.archive.org/20/items/TheFeelingOfPower/The%20Feeling%20of%20Power.pdf
- Clean version: https://hex.ooo/library/power.html
- Wiio's laws: https://en.wikipedia.org/wiki/Wiio%27s_laws
- Frame problem: https://en.wikipedia.org/wiki/Frame_problem
- Cyc: https://en.wikipedia.org/wiki/Cyc
- GPT-5.2 system prompt: https://github.com/Wyattwalls/system_prompts/blob/main/OpenAI/gpt-5.2-thinking-20251213
- Adversarial reasoning: https://www.latent.space/p/adversarial-reasoning
- Anthropic prompt improver: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompt-improver
