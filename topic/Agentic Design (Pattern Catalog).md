# Agentic Design (Pattern Catalog)

A free, searchable architecture catalog for AI builders by KORTEXYA SAS: 280+ agent design patterns, 280+ techniques, 900+ use cases, and 280+ interactive demos. A connected catalog — not a list of buzzwords — that lets engineers compare, inspect, and apply design patterns for building reliable AI agents. The freemium model ("start with the library, pay when the tools save you work") adds a Prompt Optimizer, Eval Lab, and red-team audits in the Pro tier.

---

## Key Quotes

> "A connected catalog, not a list of buzzwords."

The site's self-description, and the thing that distinguishes it from the flood of AI reference sites that are really just SEO landing pages. The claim is that patterns are connected — you can trace from a constraint (reliability, latency, cost, safety) through relevant patterns to implementation options. This is the [[A Pattern Language (Christopher Alexander)]] model applied to agent architecture: each pattern names a recurring problem and its proven solution, and patterns compose.

> "Start with the library. Pay when the tools save you work."

The freemium thesis in one sentence. The entire catalog is browsable for free; you pay for the Pro tools (Prompt Optimizer, Eval Lab, red-team audits) that operationalize the patterns. This is a smarter pitch than most AI tooling — it bets that the patterns themselves are marketing for the tools, and that engineers who learn the catalog will pay to automate what they now understand.

> "Design AI agents with patterns you can inspect, compare and apply."

The verb choices matter: *inspect* (not just read), *compare* (side-by-side trade-offs, not isolated descriptions), *apply* (practical, not academic). This is engineering documentation written for builders, not researchers.

> "Choose how an agent should reason, use tools and fail safely."

The production framing: agent design isn't about capability, it's about reliability. The hard part isn't making an agent that works; it's making one that fails gracefully in production. This aligns with [[Building Agents That Don't Break Themselves]] and the broader harness-engineering thesis.

## Key Themes

#agent-architecture #pattern-catalog #design-patterns #tool #freemium

### The Pattern Language Revival

Christopher Alexander's 1977 *A Pattern Language* gave architects 253 composable patterns for buildings. Agentic Design applies the same model to AI agent architecture — and it's one of the first to do so at scale. The 280+ patterns aren't just a list; they're connected, so you can navigate from a concrete constraint to the relevant design patterns to implementation techniques. This is what distinguishes a catalog from a wiki: the connections are curated, not just hyperlinked.

### 4-Step Design Workflow

The site's workflow — Frame → Compare → Inspect → Practice — is a useful mental model even without the product. **Frame** forces you to name the constraint (reliability, latency, cost, or safety) before reaching for a pattern. **Compare** puts patterns side by side on mechanism, trade-offs, and use cases — the missing feature in most architecture documentation. **Inspect** dives into implementation with interactive examples. **Practice** closes the loop with learning paths and labs. This is a better design process than most teams follow for any architecture decision.

### The Freemium Bet

The free tier is genuinely useful — full catalog access, interactive demos, an AI expert, sandboxed code execution. The Pro tier adds operational tools: Prompt Optimizer, Eval Lab, red-team audits. This is a calculated bet that (a) the catalog is valuable enough to attract a large free audience, (b) a fraction of that audience will pay for tools that automate what the catalog teaches, and (c) the catalog itself improves as more people use it. The bring-your-own-key option (Anthropic/OpenRouter) is smart — it avoids the margin-crushing token markup model.

### What's Missing

The site claims 280+ patterns but the homepage doesn't name a single one. There's no sample pattern, no taxonomy overview, no sense of what "pattern" means in this context. Is a pattern a code template? A design decision tree? An architecture diagram? The 4-step workflow is clear, but the catalog's actual shape is invisible from the landing page. This might be a deliberate free-tier tease, or it might be that the patterns are thinner than the count suggests. Either way, you can't evaluate a pattern catalog without seeing a pattern.

### Where It Fits

The agent design space is filling in. [[Elements of Agentic Systems Design]] gives you the ten-element taxonomy. [[The Agentic Product Standard v2.0]] gives you the production standard and Claude Code skills to apply it. [[Agent-Native Architectures (Every)]] gives you the five principles for building applications around agents. Agentic Design gives you the pattern catalog — the connective tissue between these frameworks, organized by constraint rather than taxonomy. It's the reference manual in a stack that already has the textbook and the field guide.

### The KORTEXYA Question

The site is produced by KORTEXYA SAS, a French company with no other visible products. Is this a startup building toward a paid platform? A consultancy's lead-gen tool? A labor of love by someone who got tired of explaining the same agent architecture patterns over and over? The domain name (agentic-design.ai) is strong, the catalog scale is ambitious, and the freemium model is coherent. But there's no About page, no team page, no blog. The absence of author identity is conspicuous in a field where the best resources are personally signed.

---
*Sources: [[raw/agentic-design]]*
*Last updated: 2026-08-01*
