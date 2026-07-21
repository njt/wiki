# Controlling Reasoning Effort in LLMs

Sebastian Raschka maps the emerging design space of reasoning-effort control: how models are taught to think harder or cheaper on demand, from DeepSeek-R1's all-or-nothing verbosity through GPT-5.6's six-position dial to Inkling's continuous 0.2–0.99 slider. The piece is a survey, not a paper — but it's the clearest map yet of a capability that will determine the economics of every agent loop.

---

## Key Quotes

> "The `<think>` and `</think>` tags are not giving the model the ability to 'think' or reason or reason better."

Raschka on the cosmetic nature of reasoning tags. This is the single most important sentence in the article. The tags are a format reward artifact — the model learned to wrap its reasoning in them because RLVR rewarded correctly formatted output, not because the tags enable anything. They're training wheels the model never actually needed, and their persistence as a user-visible feature is a UX decision, not a mechanism. The parallel to chain-of-thought prompting is exact: telling a model to "think step by step" doesn't change its architecture, it changes which tokens it emits.

> Early reasoning models lacked a toggle — "what is the capital of France?" would trigger a full reasoning trace.

The DeepSeek-R1 era's most visible failure mode. This isn't just wasteful — it's a category error. The model can't distinguish between questions that benefit from reasoning and questions that don't, because the training signal (RLVR on math/code correctness) never taught it to. The toggle problem is downstream of the training-data problem.

> Inkling uses continuous effort values (0.2–0.99) rather than discrete levels.

The most interesting design choice in the survey. A continuous slider means effort becomes a differentiable parameter you can optimize over — you can A/B test effort levels, gradient-descent toward the cost/quality sweet spot, and dynamically adjust per-query based on estimated difficulty. Discrete levels (Light/Medium/Heavy/Ultra) are a product decision that makes reasoning feel like a feature toggle. Continuous effort makes it feel like a knob on a mixing board.

> The "holy grail is of course automatic effort selection."

Raschka's closing line, and he's right in one sense and wrong in another. Automatic selection is right as a product direction — users shouldn't have to pick reasoning effort any more than they pick CPU clock speed. But automatic selection also makes cost invisible, and invisible costs in LLM systems have a way of compounding silently. The real holy grail is automatic selection with transparent cost feedback — the model chooses, but you can see what it chose and what it cost.

---

## Key Themes

- #concept **reasoning-effort control** — The ability to dial reasoning depth up or down per-query, from binary on/off (DeepSeek-R1) through discrete levels (GPT-5.6) to continuous sliders (Inkling). This is the economic lever for reasoning models: every agent loop that doesn't need full reasoning depth is wasted money.
- #concept **RLVR (Reinforcement Learning with Verifiable Rewards)** — The training paradigm that produced reasoning models: reward only the final answer correctness, not the reasoning trace. The "aha moment" emerges without being trained for. DeepSeek-R1 popularized it; everyone else adopted it.
- #pattern **length penalty as effort knob** — The core training insight: vary the length penalty in the RL reward function, and the same base policy produces different reasoning depths. No architecture changes, no separate models. Just a coefficient.
- #pattern **SFT after RLVR** — Train the reasoning model with RLVR first, then supervised-fine-tune it to obey effort-level commands by pairing prompts with target responses at specific effort levels. The SFT phase teaches controllability without degrading the RLVR-learned reasoning capability.
- #comparison **discrete vs. continuous effort** — GPT-5.6's six discrete levels vs. Inkling's 0.2–0.99 continuous slider. Discrete is better UX; continuous is better infrastructure. The convergence path is probably continuous at the API layer with discrete presets in the product layer.
- #tool **DeepSeek V4, Nemotron 3 Ultra, Kimi K2.5, GLM-5, Qwen3, Inkling** — The six open-weight model families surveyed, each with a different implementation approach. The diversity of solutions suggests the problem isn't solved — we're still in the Cambrian explosion phase.

## Critical Analysis

**What this article does well:** Raschka has a gift for making training pipelines legible. The RLVR → SFT sequence, the length-penalty-as-knob insight, the comparison table — these are the building blocks someone needs to understand *why* their reasoning-model bill is $5 one query and $0.02 the next. The survey of six open-weight implementations is genuinely useful as a design-space map.

**What's undersold:** The article presents effort control as a model-training problem, but it's equally a systems problem. If you're running an agent loop that makes 30 LLM calls, shaving 80% off the reasoning depth for 25 of those calls (the ones that are just "read this file" or "check this condition") is a 5× cost reduction with no quality loss. Effort control isn't a feature — it's the economic prerequisite for reasoning models in production. Raschka gestures at this but doesn't say it explicitly.

**The sharp take:** The six-implementation survey is the article's real contribution, and it reveals a field in the Cambrian explosion phase. DeepSeek V4 trains separate specialists with different context windows. Nemotron uses random-budget truncation during training. Kimi alternates budgeted and unconstrained RL phases. GLM-5 does it through SFT. Qwen3 does mode fusion. Inkling does continuous conditioning. These are radically different approaches to the same problem, and nobody knows which one wins. The lack of convergence is the signal.

**Compared to related wiki coverage:** [[The Reasoning Trap]] (Yin et al.) is the cautionary companion — every technique that enhances reasoning also amplifies tool hallucination. Effort control without hallucination control is a footgun. [[LLM-as-a-Verifier]] establishes verification as a distinct scaling axis from generation — effort control on the generator plus a fixed-effort verifier may be the optimal architecture. [[Model Routing Is Simple Until It Isn't]] covers the routing half of the same problem: once you have variable-effort models, when do you spend the tokens? [[What Broke and Why — RL Post-Training]] is the practitioner's companion — Luv Verma's catalog of RL training failures is what happens when you implement these techniques without debugging them.

**Raschka's role in the wiki:** This is Raschka's third article covered here, after [[Recent Developments in LLM Architectures]] (the KV-cache compression survey) and [[Components of a Coding Agent]] (the harness taxonomy). Together they form a triptych: the model architecture determines the cost envelope, the training technique determines the capability range, and the harness determines whether any of it reaches production. Raschka is the best bridge between research and practice in the LLM space right now — he reads the papers so practitioners don't have to, and he writes clearly enough that they actually want to.

**Who should read this:** Anyone paying for reasoning-model API calls. If you're using Opus 4.8 or GPT-5.6 with reasoning enabled, understanding effort control is the difference between a $50 agent run and a $5 one that produces the same answer. Also relevant for anyone building agent harnesses — effort selection should be a harness concern, not a user-facing setting.

---

*Sources: [[raw/controlling-reasoning-effort-in-llms]]*
*Last updated: 2026-07-21*
