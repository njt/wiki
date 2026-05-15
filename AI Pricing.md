# AI Pricing

A free, unauthenticated JSON API that tracks per-token pricing for AI model APIs across 19 providers. No scraping, no API keys -- just `curl` a URL and get structured pricing data back. Built for agents and CLIs, not just humans in browsers.

---

## Key Quotes

> "Per-token price rows with provider, model, metric, billing tier, unit, currency, and latest observation timestamp."

The data model is the product. Each price is decomposed into its full context: who serves it, what tier, what metric, what currency, when last observed. This is pricing-as-structured-data, not pricing-as-screenshot.

> "Markdown API documentation for agents and command-line clients."

The site serves its own docs as markdown via `Accept: text/markdown`. An agent can read the API docs, understand the schema, and start querying prices -- all without a human in the loop.

## Key Themes

#tool #pricing #api #agent-infrastructure

- **Agent-native design** -- The entire API is built for programmatic consumption. No auth, no API keys, rate limits generous enough for scripting (60-120 req/min), markdown docs for LLM ingestion. This is what "agent-friendly infrastructure" looks like in practice: not an MCP server, just a clean REST API with good docs.

- **Price decomposition** -- Prices aren't a single number. They're broken into input tokens, output tokens, cached input tokens, cache write tokens, across billing tiers (standard, batch, flex, priority, turbo). The `model_group_key` lets you compare the same model across hosting providers. This granularity matters because the real cost of an agent session depends heavily on cache hit rates and I/O ratios.

- **19 providers, one schema** -- OpenAI, Anthropic, Google, DeepSeek, xAI, Groq, Fireworks, Together, DeepInfra, Cloudflare Workers AI, Novita, Cerebras, Nscale, DigitalOcean, Kimi, GLM, Qwen, MiniMax, Mistral. The normalization work -- canonical slugs, model group keys, vendor vs. serving provider distinction -- is where the real value lives. Raw pricing pages are easy to find; normalized cross-provider comparison data is not.

- **The terminology is revealing** -- The distinction between "canonical vendor" (who made the model) and "serving provider" (who hosts it) reflects the reality that open-weight models live on many platforms at different prices. The "offer" abstraction captures that the same model from the same provider can have multiple pricing tiers. This vocabulary alone is useful for thinking about AI economics.

## Critical Analysis

**What's valuable:** This fills a genuine gap. If you're building an agent that needs to select models by cost, or a dashboard that tracks spending, or a CLI that estimates session costs before running, you need exactly this kind of structured pricing data. The alternative is scraping individual provider pricing pages, which break constantly and aren't machine-readable. The OpenAPI schema and markdown docs mean an agent can bootstrap itself on this API without human intervention.

**What's missing:** No historical pricing data is exposed (the site says it tracks "current and historical" but the API only serves current snapshots). Price trends over time -- which providers are getting cheaper, which models are converging in price -- would be the killer feature for anyone making build-vs-buy decisions. Also missing: quality-adjusted pricing. Knowing that Haiku costs 1/30th of Opus means nothing without knowing which tasks Haiku can actually handle. A pricing API paired with eval data would be transformative.

**The deeper question:** This is infrastructure for a world where token prices matter. Right now, most teams treat AI costs as a rounding error or an unmanageable cloud bill. As agent workloads scale -- sessions running for hours, multiple agents in parallel, batch processing thousands of documents -- per-token pricing becomes a real engineering constraint. Tools like [[session-analysis]] show that cache tokens dominate actual API bills; this API provides the price-per-cache-token data needed to turn that insight into dollar estimates.

**Comparison to alternatives:** [[Self-Hosted LLMs]] tackles cost from the other direction -- running your own models to avoid per-token pricing entirely. [[stupidmeter]] benchmarks model quality but not cost. [[Computer Use is 45x More Expensive Than Structured APIs]] demonstrates why token costs compound (551k tokens for a vision agent vs 12k for an API agent -- at different per-token rates, the cost gap widens further). [[How to Buy Cheap Claude Tokens in China]] shows the grey market that emerges when official pricing creates arbitrage opportunities. None of these tools provide a clean, queryable pricing database. This one does.

---

*Sources: [[raw/ai-pricing-fyi]]*
*Last updated: 2026-05-14*
