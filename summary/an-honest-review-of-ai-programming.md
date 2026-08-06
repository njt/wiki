---
url: https://mropert.github.io/2026/08/04/an_honest_review_of_ai_programming/
title: An Honest Review of AI Programming
author: Mathieu Ropert
date_published: 2026-08-04
date_fetched: 2026-08-06
---

Mathieu Ropert, a C++ game developer and consultant, spent three months using Claude and other LLMs for programming work and came away with a nuanced, contrarian assessment: LLMs are genuinely useful as research assistants and codebase-exploration tools, but consistently mediocre at writing code. His review is structured around four domains — search, hallucinations, coding, and sustainability — and his thesis is that the tool's real value is in *finding and summarizing information*, not generating it.

**Search is the killer app.** Ropert finds LLMs excellent at answering pointed natural-language questions, effectively automating the "run N searches, skim top links, synthesize" loop. His most novel observation: LLMs are uniquely good at searching internal company knowledge bases (wikis, Slack, Confluence) whose built-in search is so broken that even exact-title queries fail. The LLM's ability to generate synonym-rich queries bridges the gap between what you type and how the information was actually written — a capability that enterprise search tools should have had decades ago but didn't.

**Hallucinations are structural, not fixable.** Ropert catalogs three failure modes beyond the standard "invented API" problem: (1) the self-reinforcing loop where connectors let an LLM cite your own work-in-progress as supporting evidence, (2) the telephone game where an LLM trusts another LLM's summary over primary sources, and (3) the "reasonable-sounding feature that doesn't exist" when asking niche questions about CMake, Vulkan, or Xcode. His advice: always verify against primary sources, especially in domains you don't know well.

**LLMs are bad at writing code.** Ropert's attempts at code generation produced over-engineered, OOP-pattern-heavy solutions that ignored his explicit instructions. For game development specifically, the training data problem is acute: the last AAA game to be open-sourced was Doom 3 (2004, 22 years ago), so models are trained on hobby projects, game jam entries, and tutorial demos. Custom engines with bespoke scripting languages fare even worse — the only training data comes from mods. Output tokens cost 5–10× more than input tokens, making code generation economically worse than summarization.

**The coding assistant framing is the right one.** Ropert invokes Stafford Beer's management cybernetics to argue that dropping into a new codebase requires processing more information than a human can absorb — and LLMs as *assistants* that help you pull on threads and explore code are genuinely valuable, provided you double-check everything. This is convergent with the "advisor" pattern many teams have landed on.

**Sustainability is dubious.** AI companies are all losing money; tokens make a marginal profit but overall operations run at a loss. Hardware isn't getting cheaper, models need constant retraining, and Ropert expects prices to rise. He contrasts management's willingness to mandate $20+/month LLM subscriptions for everyone with their historic refusal to approve a €20/month profiler license for engineers who asked for it — a symptom of top-down AI mandates disconnected from engineering reality.
