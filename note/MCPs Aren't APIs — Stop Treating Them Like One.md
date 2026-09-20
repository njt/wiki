# MCPs Aren't APIs — Stop Treating Them Like One

This piece reframes MCP server design as a context-economics problem rather than an integration problem. Its central claim: the common pattern of mirroring a REST API endpoint-for-endpoint in MCP tools is actively harmful, because tool definitions and tool outputs are paid for in context tokens on every single request, and bloated tool surfaces measurably degrade model performance.

---

## The argument in one paragraph

Because MCP tool definitions are injected into the context window on every request and tool outputs are loaded back into it after every call, an MCP server's design is a token-budget decision, and the industry's default — wrapping each REST endpoint as a tool — produces servers that are expensive, slow, and worse at their job. The author claims a concrete threshold (performance degrades noticeably beyond 30–40 tools), a concrete cost (13,000 tokens from two servers), and a design alternative (small, domain-scoped servers exposing capabilities, not endpoints, returning model-shaped output). This is falsifiable: if tool count above 40 had no effect on task performance, or if raw API passthroughs performed as well as transformed summaries, the argument would collapse.

## Key quotes

> Every time you send a request, **your MCP tools and their parameters are loaded into the context window**, before your prompt even gets processed.

The mechanism the whole article rests on: tool schemas are a per-request tax, not a one-time setup cost. Most teams reason about MCPs as if they were installed once.

> A real example that happened to me: adding just 2 MCP servers injected **13,000 tokens** into the context.

The number that makes the argument land. 13,000 tokens of schema overhead is not a rounding error — it's a meaningful fraction of most context windows, spent before any actual work begins.

> **Beyond 30-40 tools, performance degrades noticeably**. The model struggles with choice paralysis, and context bloat leaves less room for your actual data and instructions.

The strongest empirical claim in the piece, and the least defended — no benchmark, no task, no baseline. It is asserted from experience, which is either the article's credibility or its weakness depending on how much you trust the author's sample size of one.

> Think of your MCP as a **service layer**, not a pass-through proxy. The model doesn't need to orchestrate 7 API calls, it needs to accomplish a task.

The design inversion at the heart of the piece: the tool surface should be shaped by the agent's tasks, not by the backend's resource structure. This is the same insight that separates good CLIs from raw SDK bindings.

> The principle: return only the information the model needs to answer the question. Nothing more.

Output shaping gets half the article's attention but is arguably the more novel half — input bloat is at least visible in token counts, while a single unbounded tool response silently flooding the context is not.

## Critical analysis

The non-obvious move here is treating the MCP server as an *interface designed for a model* rather than a transport for an existing API. The `manage_user` example is genuinely instructive: a tool that aggregates creation, updates, and permission changes into one call returns "a clean summary" instead of forcing the model to orchestrate seven calls and hold their intermediate results in context. That's not just token savings — it removes failure modes where the model mis-sequences calls or misinterprets partial states. The article's best instinct is that both directions of the pipe cost tokens: schemas going in, responses coming back, and the output side is where the silent disasters live (a raw response "so large it could overwhelm the context window almost immediately").

What's weak: the numbers are anecdotal. "13,000 tokens from 2 servers" and "performance degrades beyond 30–40 tools" are plausible and consistent with wider reports of tool-selection degradation, but the article offers no measurement methodology, no model names, no task comparison. The do's-and-don'ts table is presented with more confidence than the evidence supports. It also underplays the trade-off it's making: coarse capability-level tools like `manage_user` push complexity into the server's implementation and can make tool *descriptions* do heavy lifting that a well-named set of narrow tools would have done structurally. There's a real tension between "fewer, bigger tools" and the model's ability to predict what a big tool will do — the article picks a side without acknowledging the cost.

What's left out: any mention of tool search beyond a parenthetical ("unless your agent app supports tool search (many don't)"), which is the ecosystem's main answer to exactly this problem and deserves more than one clause. Nothing on how dynamic tool registration or per-user tool filtering (as in authenticated, tailored MCP deployments) interacts with the domain-splitting advice. And nothing quantifies the actual dollar cost despite the article's own opening tease — "do you know how much your MCP is actually costing you?" — which it never answers in currency, only in tokens.

## Related

- [[MCP and Tool Protocols]] — This source adds a cost-and-context lens to the topic's coverage of how agents reach tools: where most MCP material asks what a server can expose, this asks what exposure costs per request, and answers with concrete design limits.
- [[Control Plane MCP Server]] — Directly complicates that page's admiration of a "feature-complete" 80-tool MCP implementation: by this article's thresholds, 80+ tools is deep in the degradation zone, and the AI Plugin curation layer on top is less a bonus than a necessary patch for a tool surface that should have been split by domain.
- [[How AI Coding Agents Actually Use Your Technology]] — Strengthens Mastykarz's AX argument from the server side: he traces how SDK and MCP design silently degrades agent behaviour downstream; this piece gives the concrete mechanism (per-request schema injection and unbounded outputs) and prescriptive thresholds for the tool provider.
- [[Stateless MCP]] — Nuances Willison's auditability argument for explicitly-declared MCP tools: this article agrees declared tools are controllable but adds that every declared tool is also a standing token tax, so the declaration surface itself needs curation, not just auditability.

---
*Sources: [[raw/mcps-arent-apis-stop-treating-them-like-one-11420]], [[summary/mcps-arent-apis-stop-treating-them-like-one-11420]]*
