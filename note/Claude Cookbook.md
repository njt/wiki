# Claude Cookbook

Anthropic's official collection of 84 practical guides, notebooks, and examples for building with Claude — part developer documentation, part product roadmap, part onboarding funnel. The cookbook spans vision basics (August 2023) through async multi-agent orchestration (June 2026), and in that chronological arc you can watch Anthropic's product strategy unfold in real time.

---

## Structure

The cookbook is a flat, reverse-chronological index — no sections, no learning paths, no curated tracks. Each entry links to a dedicated page on `docs.anthropic.com`. The GitHub repo ([anthropics/claude-cookbooks](https://github.com/anthropics/claude-cookbooks)) accepts community contributions.

84 entries across these categories:

| Category | Count | Era |
|----------|-------|-----|
| Agent Patterns | ~12 | 2024–2026 |
| Claude Agent SDK | 8 | Sep 2025–May 2026 |
| Claude Managed Agents | 9 | Apr–May 2026 |
| Tools | ~11 | Apr 2024–Mar 2026 |
| RAG & Retrieval | ~8 | Jul 2024–Mar 2026 |
| Multimodal | 6 | Mar–Nov 2025 |
| Evals | 4 | Mar 2024–Jun 2026 |
| Integrations (3rd-party) | ~13 | Aug 2023–Nov 2025 |
| Skills | 3 | Oct 2025 |
| Thinking | 2 | Feb 2025 |
| Responses/Misc | ~8 | Mar 2024–Jun 2026 |

## Key Quotes

> "Practical guides and examples for using Claude effectively"

The tagline undersells what the cookbook actually is. This isn't a reference manual — it's a product strategy document disguised as documentation. Every recipe that lands signals what Anthropic wants developers to build, and what they want developers to stop building themselves.

> "Contributions welcome. Have an idea for a cookbook? We welcome community contributions."

The open contribution model (GitHub PRs) is smart: it turns the cookbook into a distribution channel for third-party integrations (ElevenLabs, Deepgram, Wolfram Alpha, Pinecone, MongoDB) while keeping Anthropic's own recipes front and center. It's developer relations as platform play.

## Key Themes

### #pattern — The Cookbook as Product Archaeology

Read chronologically, the cookbook tells a clear story:

- **2023**: "Here's how to call the API." Basic PDF upload, Wikipedia search. Claude is a model you prompt.
- **Early 2024**: RAG, tool use, structured JSON, evals. Claude is a component you engineer around.
- **Mid 2024**: Classification, summarization, contextual retrieval, prompt caching, batch processing. Claude is a production system you optimize for cost and latency.
- **Late 2024**: Basic agent workflows, evaluator-optimizer, orchestrator-workers. Claude is an agent you design patterns for.
- **2025**: Extended thinking, programmatic tool calling, context compaction, speculative caching, Claude Agent SDK, Skills. Claude is a platform you build on.
- **Early 2026**: Site reliability agents, vulnerability detection, session browsers, OpenAI SDK migration. Claude is infrastructure you migrate to.
- **Mid 2026**: Managed Agents launch, memory, outcomes, multiagent coordination, async orchestration, benchmark reproduction. Claude is a managed service you configure, not code.

That arc — model → component → system → agent → platform → infrastructure → managed service — is the Anthropic strategy, told in recipe form. The cookbook doesn't just document capabilities; it manufactures demand for the next tier of abstraction.

### #pattern — Claude Cookbook vs. OpenAI Cookbook

The [[OpenAI Cookbook]] covers similar ground but the emphasis differs sharply:

- **OpenAI** leads with code snippets and API mechanics. The recipes are "here's exactly how to call this endpoint."
- **Claude** leads with architecture patterns and design rationale. The recipes are "here's how to think about building this kind of system."

This isn't accidental. OpenAI has the market share; Anthropic has to compete on developer experience and architectural opinion. The cookbook is a vehicle for that opinion. Where OpenAI's cookbook says "you can do X," Claude's says "here's the right way to do X, and here's why."

### #tool — Managed Agents as the Strategic Pivot

The most significant section of the cookbook is the Managed Agents cluster (9 entries, April–May 2026). These aren't just recipes — they're the launch documentation for a product that abstracts away most of what the earlier cookbook entries teach you to build yourself:

- Agent creation, environment setup, session management
- File mounting and sandboxed execution
- Memory persistence across sessions
- Outcomes-based verification loops
- Prompt versioning with rollback
- Production webhook patterns for human-in-the-loop
- Multiagent coordination with scoped tool sets

The implicit message: "You could build all this yourself using the Agent SDK recipes from 2025, or you could use Managed Agents and skip to the part where your agent actually works."

### #concept — The Missing Learning Paths

The cookbook's flat structure is its biggest weakness. An engineer new to Claude in 2026 faces 84 entries with no guidance about where to start. The "featured" section highlights six, but they're a grab-bag: PTC, tool search, context compaction, crop tool, frontend aesthetics, Skills introduction.

A curated learning path exists implicitly — start with the vision basics (Mar 2024), learn tool use (Apr 2024), understand RAG (Jul 2024), grasp evals (Mar 2024), then move to agent patterns (Dec 2024) and the Agent SDK (Sep 2025) — but Anthropic doesn't surface it. The cookbook is a library, not a curriculum.

### #concept — Community Recipes as Strategic Content

The third-party integration recipes (ElevenLabs, Deepgram, Wolfram Alpha, LlamaIndex, Pinecone, MongoDB, LangChain) serve a dual purpose: they make Claude look well-integrated, and they offload integration maintenance to the partners themselves. Each integration cookbook is a commitment device — once Pinecone has a recipe in the official Claude Cookbook, they have a stake in keeping it current. Anthropic gets an ecosystem for the cost of a PR review.

## Critical Analysis

**The cookbook is Anthropic's most effective developer acquisition tool and it doesn't read like marketing.** That's the trick. Every recipe is genuinely useful, and every recipe also says "you should be using more Anthropic products." The progression from "Claude via API" to "Claude Agent SDK" to "Claude Managed Agents" is a masterclass in gradually escalating commitment — each tier solves problems the previous tier created.

**The chronological organization is a bug, not a feature.** A new developer doesn't need to know what Claude could do in August 2023; they need to know what to do today. The cookbook would benefit from task-based organization (Build a RAG System, Create an Agent, Deploy to Production, Evaluate Quality) rather than "here's everything we've ever published in reverse order." The fact that Anthropic hasn't reorganized suggests the cookbook is under-invested relative to its strategic importance, or that its primary audience is existing users tracking what's new, not new users finding their way.

**The Managed Agents recipes create a tension the cookbook doesn't acknowledge.** If you follow the Agent SDK path (build your own agent runtime), you'll invest weeks in infrastructure. If you follow the Managed Agents path, you'll be on a proprietary platform with a pricing model you don't control. The cookbook presents both as equally valid choices, but they represent fundamentally different bets about where your architecture lives. This isn't a cookbook problem per se — it's a platform strategy tension — but the cookbook's "all recipes are equal" framing obscures it.

**What's missing**: There's no recipe for cost management across these patterns (the Admin API cookbook is read-only analytics, not active cost control). No recipe for hybrid architectures (some Managed Agents, some self-hosted SDK agents). No recipe for migrating *away* from Managed Agents if the pricing changes. And nothing on the failure modes — what breaks when you compose these patterns in production. The cookbook teaches you to build; it doesn't teach you to operate.

**The cookbook is best understood as API documentation for a product that refuses to be just an API.** Each recipe is a thesis about what the right level of abstraction is. In 2023, the right level was HTTP endpoints. In 2026, it's a managed agent platform. The cookbook is the argument, made in code, that you should let Anthropic handle more of the stack. Whether that's good for you depends on whether you believe Anthropic's incentives will stay aligned with yours.

---
*Sources: [[raw/claude-cookbook]]*
*Last updated: 2026-07-25*
