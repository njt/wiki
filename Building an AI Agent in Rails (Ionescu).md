# Building an AI Agent in Rails (Ionescu)

Catalin Ionescu's field report on adding an AI agent to a 7-year-old multi-tenant Rails monolith using RubyLLM's tool-calling DSL — and discovering the whole thing was surprisingly simple once authorization was moved from prompt engineering to the tool execution layer.

---

## Key Quotes

> "no model should ever know" a given user's private data from a SaaS service

The core insight, expressed in six words. The LLM doesn't need unrestricted data access — it needs tools whose execution is already scoped by the existing authorization system. This inverts the security model from "trust the model to behave" to "the model can't misbehave because the tools won't let it."

> I realized I could encode our authorization logic into specific function calls, giving the LLM data access without having to give it unrestricted access.

This is the [[Guardrails and Feedback Loops]] pattern applied at the agent design level: deterministic enforcement at the tool boundary, not prompt-level pleading. The Pundit policy scope becomes the guardrail. The LLM asks for data; the tool decides what it's allowed to see.

> GPT-4 was very prone to hallucinations — rushing to respond with made-up data instead of calling the necessary tools

Notable that GPT-4, not a smaller model, was the worst offender. Ionescu found GPT-4o struck the best balance of speed and correctness. Anthropic and Gemini remain untested. The model selection table is candid and practical — exactly the kind of field data that's more valuable than benchmark scores.

> The entire agent took 2-3 days of Claude-assisted development. AIs building AIs.

Two to three days from idea to working agent in a production Rails app with real authorization constraints. The author's surprise at the low complexity is itself the headline: "the tool service object is essentially an API controller action — pass inputs and get a JSON back."

> ActiveAgent had no built-in support for defining tools or having long-running conversations

Ionescu evaluated ActiveAgent (which moves prompts into view files) and rejected it. The tool-definition gap was the dealbreaker. This is a useful data point for anyone choosing a Rails AI framework: if it can't do tool calling and multi-turn conversations, it's not an agent framework.

## Key Themes

#tool #pattern #rails #case-study

- **Authorization at the tool layer, not the prompt layer.** Pundit scoping runs inside the tool, so the LLM only ever sees data the current user is authorized to access. This is the production-grade version of "don't prompt-engineer security."
- **Tool calling as the integration primitive.** RubyLLM's DSL (typed parameters, descriptions) makes function calling feel like writing a controller action. The LLM decides *which* tool to call; the tool decides *what* to return.
- **Brownfield Rails is viable.** This isn't a greenfield prototype — it's a 7-year-old multi-tenant monolith with sensitive data. The agent was bolted on without architectural upheaval.
- **Hotwire + Active Job + Conversations.** The UI stack (Turbo Streams broadcasts, Stimulus scrolling, background jobs for LLM calls) is a clean pattern for async agent interactions in Rails.

## Critical Analysis

**What's strong:** The authorization-through-tools pattern is the right answer to the "how do I let an LLM query my database without it leaking data" problem. Ionescu sidesteps the entire prompt injection / data leakage debate by never putting sensitive data in the prompt in the first place. The tool is the security boundary. This is the [[Smart Models Dumb Pipes]] principle applied to data access: the model makes the judgment ("search for X"), the pipe enforces the policy ("here's what you're allowed to see about X").

**What's missing:** No discussion of observability — what happens when the agent returns wrong results? How do you debug a tool-calling chain? No mention of rate limiting, cost tracking, or what the per-query token economics look like. The model comparison is GPT-only; the fact that Anthropic and Gemini are "not yet tested" is a confession that this is v0.1, not a mature system. The article also skips over the hardest part: what happens when the agent needs to *write* data, not just read it?

**The ActiveAgent rejection is telling.** Moving prompts into view files is a Rails-idiomatic instinct that makes the wrong tradeoff — it optimizes for template cleanliness while ignoring the actual requirements of agent design (tool definitions, multi-turn state). This is a miniature case study in why framework choice matters and why "Rails-native" isn't always the right answer.

**The 2-3 day timeline is both impressive and incomplete.** Claude-assisted development compressed what might have been weeks into days, but the article describes a read-only search agent — the simplest possible agent architecture. The real test is what happens when the agent needs multi-step reasoning, writes to the database, or handles ambiguous user intent. Part 2 (if it exists) will tell us whether the pattern holds up.

## Related Pages

- [[Guardrails and Feedback Loops]] — Deterministic enforcement at the tool boundary is a guardrail, not a prompt
- [[Smart Models Dumb Pipes]] — The LLM as judgment machine, tools as deterministic execution
- [[Minions — Stripe's One-Shot Coding Agents]] — Another brownfield Ruby codebase + AI agents, at Stripe's scale
- [[Building Agents for Production Systems with MCP]] — Anthropic's guide to agent-to-production integration
- [[Elements of Agentic Systems Design]] — Design taxonomy: this article demonstrates Agency, Context, and Memory in practice
- [[Harness Engineering]] — Feedforward (tool definitions) vs. feedback (authorization enforcement) in agent design
- [[Compound Engineering]] — The tool-as-security-boundary pattern is compound engineering: add a system, not manual review
- [[Semantic Kernel]] — Microsoft's enterprise function-calling middleware; RubyLLM is the Rails equivalent

---
*Sources: [[raw/building-ai-agent-rails-part-1]]*
*Last updated: 2026-05-14*
