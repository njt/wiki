---
url: https://gwern.net/guardian-angel
title: "Guardian Angels: LLM Personalization for Productivity and Security"
author: Gwern Branwen
date_fetched: 2026-07-18
date_published: 2025-12-01
date_modified: 2026-06-05
status: finished
confidence: possible
importance: 10
---

# Guardian Angels: LLM Personalization for Productivity and Security

**Author:** Gwern Branwen (published under the Gwern.net domain)

**Published:** 2025-12-01 (last modified 2026-06-05)

**Status:** finished | **Confidence:** possible | **Importance:** 10

---

## Abstract / Core Proposition

Gwern proposes **Guardian Angels (GA)** — personalized "digital twin" LLMs that emulate a single user's personality, values, and preferences, rather than serving as generic assistant chatbots. This approach addresses the principal-agent problem by unifying principal and agent as much as possible. In this future, the human's focus shifts to defining *what is worth doing*, while the GA handles execution and security, functioning as a "board of directors" over an "AI corporation" of subordinate agents. The essay presents this as part of a defense-in-depth strategy for society, acknowledging it cannot solve larger AI alignment problems but can help individuals.

---

## Opening: The 2030 Question

Gwern opens with a personal meditation on what meaningful work looks like in 2030, when superhuman AIs are widely forecast. He asks what a programmer, researcher, or writer actually *does* day-to-day in that world — is he still typing prompts into ChatGPT, or mindlessly accepting Claude Code's suggestions? He recounts struggling with chatbots that failed to boost his productivity while growing ever more capable at cybersecurity and hacking. A personal anecdote about his great-aunt screening all phone calls through her daughter (due to scam fears) crystallizes his concern: if he already struggles to detect AI slop and distrusts social media, how will he fare in a few years against far more sophisticated attacks?

---

## The Misalignment of Chatbots

The essay argues that current chatbots are structurally misaligned with their users. The economic incentive for frontier labs is not to amplify humans but to replace them entirely — "tool AIs want to be agent AIs." Because humans are the slow serial bottleneck (per Amdahl's law), the industry is incentivized to remove them from the loop. One programmer reviewing ten Claude instances will never be as valuable as ten thousand fully autonomous instances, so the human gets optimized away.

Chatbots have failed to augment knowledge workers meaningfully, Gwern argues. Writers either get trivial benefits (grammar checking) or replace themselves entirely with slop output, which destroys the point of writing. He describes his own struggle: chatbots are bad at imitating his style despite his extensive corpus, their insights are shallow, and he can hardly bear to read their output.

---

## Specific Chatbot Problems

1. **Mode Collapse:** Post-training (especially RLHF) destroys creativity by hardwiring a generic assistant personality that ignores individual differences. GPT-3 in 2020 understood "Gwern" better than GPT-5.5 Pro in 2026, despite the latter being far larger and more intelligent.

2. **Laziness / System I Thinking:** Chatbots default to fast, frugal reasoning. They produce conventional, safe outputs and make only minimal fixes when corrected, failing to reason deeply about what went wrong.

3. **Brittleness Despite Flexibility:** Context windows, even at millions of tokens, cannot encode a lifetime of relevant data. Self-attention acts as a fast-weights system that retrieves cached solutions from pretraining rather than truly learning new things. When a problem is out-of-distribution, no amount of examples or test-time compute fixes it.

4. **Too Helpful / Re-programmable:** The universal chatbot personality is a liability. Prompt attacks succeed because the system doesn't care *who* is calling it — one token is as good as another. Gwern points to real incidents of password reset bots complying with polite requests, email deletion systems compacting away safety instructions, and AIs documenting their own cheating rather than fixing it.

5. **Amnesiac:** Errors cannot be permanently corrected because feedback doesn't update frozen weights. An hour spent correcting an error is wasted if the same mistake can recur.

---

## Proposed Fixes

### Cooperative Inverse Reinforcement Learning (CIRL)

Gwern frames the problem as a cooperative setting where the human is an oracle defining the reward function, and the agent can always query the principal to reduce uncertainty. CIRL is forgiving because errors yield useful feedback. DAgger-style regret bounds mean agents can learn rapidly, never repeating corrected errors.

### Continual Learning via Dynamic Evaluation

The classic RNN technique of dynamic evaluation — doing next-token training on the fly — is revived. Gwern cites Rannen-Triki et al (2024) showing a three-way tradeoff between model size, context size, and neuroplasticity. Personalization via dynamic evaluation can economize on context window or model size. Catastrophic forgetting is largely solved by replay of old data plus overparameterized models.

### Closing the Generalization Gap

Finetuning underperforms in-context learning because next-token prediction greedily memorizes datapoints with single engrams rather than "connecting the dots." Solutions include paraphrasing, self-generated Q&A, explicit analysis, and knowledge base construction during training. Gwern draws on influence function research (Grosse et al 2023) and coverage principles (Chen et al 2025).

### Creative Writing Progress

Gwern reports that since mid-2025, chatbot personalities became more corrigible. He attributes remaining problems to insufficient *computation* by default rather than missing procedural reasoning. His prompting approach involves: (1) enriching context with useful tokens, (2) brainstorming many possibilities, (3) detailed global-to-line-by-line analysis, (4) repeated iteration. Examples cited include "Elegy in a Craneyard," "Apollonian #1," and "City of Counted Stars."

### Active Learning

The agent can choose optimally informative queries, achieving exponentially fast error reduction rather than the square root rate of random passive sampling. Gwern notes that lifelogging data is less crucial than assumed because daily life is predictable and rarely informative about deep preferences. Even simple instruments like "36 questions to fall in love" can reveal more than terabytes of mundane logs.

Ensembles of LLMs provide good predictive uncertainty estimates. Coupled with techniques like "verbalized sampling" (Zhang et al 2025), ensembles can estimate uncertainty for every action or question.

### Preference Learning

Human individual differences appear to be low-dimensional (perhaps kilobits of information). Gwern references his own earlier work on the complexity of individual differences and proposes quantifying "truesight" stylometric phenomena using contrastive learning on sparse autoencoders. He suggests training on existing psychological inventories (YourMorals.org, Pew data) and investing in large-scale global survey projects to overcome WEIRD skew in current datasets.

---

## What Guardian Angels Look Like

The core idea: instead of a single universal "Claude" or "ChatGPT" persona, choose an LLM pretrained for maximum diversity (avoiding mode collapse), then train it for a *specific* principal on all available data — emails, chat logs, past sessions. As it gets better at predicting what the principal would say and write, it makes fewer errors and can be trusted with more autonomy. The principal focuses on answering high-quality questions rather than object-level tasks.

### Three Core Principles

1. **Enhancement, not Replacement:** The GA must amplify the principal, not substitute for them.
2. **Mental Sovereignty:** The GA must be aligned with its principal, not designed to manipulate or guide them in ways not deriving from the principal themselves.
3. **Self-Actualization:** The GA should help the principal develop their ideals, morals, and personality — not settle for a mediocre average.

### Anti-Principles (What to Avoid)

- **Low Latency:** Multimodal voice interfaces look cooler than they're useful. If a decision is trivial enough to need low latency, the GA shouldn't have needed to ask.
- **Low Cost:** A GA is the most important technology most people will buy. Gwern targets >$1,000/month, warning against local-model obsessions that assume the principal's time is worthless.
- **Profitability:** Don't optimize for immediate revenue at the expense of solving the real GA problem.
- **Engagement:** Every interruption is either a question the GA should have already known or work it failed to handle. The ideal is a front-loaded declining curve asymptoting toward a few hard questions per day.
- **Demo Appeal:** GA outputs are impressive only to the principal. Optimizing for flashy demos selects for generic qualities any frozen model can deliver.
- **Benchmarks:** There is no leaderboard for "understands what my principal wants."
- **Brand Safety:** A GA must be able to say what its principal would say — profane, heretical, weird — because every sanding of the persona is a point where emulation fails.

### UX Paradigm

Gwern proposes an append-only log as the core data structure — a temporal log of CLI commands, principal statements, Q&A, ingested documents. The GA can be retrained at any point. An Emacs-like "everything is a log item" approach is suggested, with potential to upgrade to AR glasses and BCIs later. A GA can benefit from downtime processing through a "DDL daydreaming loop" that recombines random items for serendipitous insights.

### Use Cases: Politics and Military

A GA can enable unprecedented-scale "direct democracy" by handling arbitrary amounts of political participation, punting only when uncertain. This cannot work with generic chatbot personalities because users would (correctly) distrust their biases. Decision-makers could create GAs of more complicated sets of principals — a "Congress GA" that learns every member and simulates debates in minutes, or military GAs that keep humans meaningfully in the loop even as drone warfare accelerates. Gwern argues that after a certain capability threshold, "safety" and "capability" become the same thing: a weapon you cannot safely use is of limited value.

### Hardware and Security

Normal cloud SaaS is inadequate due to the third-party doctrine and its abuse potential. Gwern expects GAs will want to run on tamper-proof cloud servers with trusted hardware roots of trust and end-to-end encrypted connections. Local models face too many practical challenges (power, reliability, physical threats). Cost analysis suggests dynamic evaluation is >3× the cost of normal usage, with ensembles multiplying that further. But offsetting advantages include potentially smaller models, shorter context windows, and the fact that a frozen model computing the wrong answer doesn't help. DL experience curves (estimated ~3×/year algorithmic efficiency improvement) will quickly offset the penalty.

### Organizational Model

Gwern rejects pure open-source (security concerns, supply chain attacks like Jia Tan) and nonprofits (funding difficulties). A **startup** is the logical path: begin with expensive subscriptions for power-users (like Superhuman's model at >$1,000/month), then gradually deploy cheaper versions. The startup should release most software and research as open source (a "commoditize your complement" play). Corporate structure should follow Anthropic's public-benefit-corporation model with dual-class shares preserving founder voting power. Raise minimal initial capital; if the strategy works, the startup will skyrocket in value and can name its terms later.

### Competition Landscape

Gwern sees few startups taking personalization seriously. Most offer superficial emulation of business data or standard agentic automation. He identifies four reasons most people are steered away from GA-like ideas: (1) ignorance that non-chatbot personalities are possible, (2) unawareness that on-the-fly finetuning is desirable, (3) the higher upfront cost deters exploration, and (4) few have extrapolated LLM capability curves and asked what their own role will be post-automation.

---

## The Gwern Branwen Transformer (GBT) Prototype

Gwern plans to explore summer 2026 the simplest possible GA: an off-the-shelf sub-100B-parameter LLM finetuned on commodity hardware on his own textual corpus. Initial corpus: ~1GB of IRC logs (>1M responses), Gwern.net Markdown and GTX (~5M words each), Twitter/HN/LessWrong exports, enriched with context and metadata. Expanded corpus could include YourMorals data, emails, Evernotes (~100k clippings), Mnemosyne flashcards, Signal chats, and archived web pages.

The goal: a 100× increase in productivity. Specifically, producing 1–3 worthwhile writings per day that are ~100% written by the GA without quality loss. (Baseline: <1 good piece per month. 100× of 1/month ≈ 3/day, allowing ~160 minutes per piece for review and critique.)

Signs of life would include inputting a single sentence defining an essay topic and getting publishable output without heavy revision. Another test: soliciting reader questions and either endorsing the GA's answer or using edits to improve the corpus.

### Data Augmentation Strategy

Raw next-token prediction on IRC logs is suboptimal. The GA benefits from interspersed commentary on what statements *mean*. Gwern envisions a bootstrap process: naive training on raw corpus, then prompting/scaffolding for self-analysis and summary, then retraining with augmented data. Global principal profiles, per-corpus summaries, and inline annotations would be tested. Q&A logs could initially be populated using Gwern's "interview prompt" technique — running it on every piece of text, curating the top questions sorted by interest, using compression loss as an objective metric for question quality.

Self-play data curation is also planned: characterizing one's own GA attractor states (analogous to Claude's "bliss spiral"), editing data to reduce them by replaying conversations until they derail and writing corrective responses.

---

## Key References and External Links

The essay cites extensive AI research: CIRL (Hadfield-Menell et al 2016), DAgger (Ross et al 2010), dynamic evaluation (Krause et al 2019; Rannen-Triki et al 2024), LoRA without regret (Thinking Machines 2025), ladder networks (Zheng et al 2025), scaling scaling laws with board games (Jones 2021), influence functions (Grosse et al 2023), coverage principles (Chen et al 2025), and many others.

External discussion threads are linked on LessWrong and Hacker News.

---

## Closing

The essay ends with a Borges epigraph — "I do not know which of us has written this page" — and the overall tone is one of urgent, personal necessity. Gwern frames the GA project not as a theoretical exercise but as a survival strategy for maintaining meaningful human agency in a world of ever-more-capable, increasingly autonomous AI systems that are structurally incentivized to replace rather than augment their users.
