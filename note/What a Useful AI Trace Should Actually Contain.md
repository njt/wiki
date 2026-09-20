# What a Useful AI Trace Should Actually Contain

Nikolay Iliev's Progress/Telerik post is an implementation walkthrough with a thesis attached: standard OpenTelemetry observability, designed for request latency and error codes, cannot answer the questions that matter when an AI agent misbehaves. A useful trace must carry the prompt, the completion, every tool call with inputs and outputs, token counts, cost, and a quality signal — and producing that requires deliberate instrumentation, not auto-magic.

---

## The argument in one paragraph

The article claims that the six-element trace (prompt, completion, tool I/O, tokens, cost, quality score) is the minimum unit of debuggability for an agent, and that no amount of standard OTel telemetry substitutes for it — a request can return HTTP 200 with zero errors and still have given a customer completely wrong product information, and without AI-specific context in the trace that failure is invisible until a user complains. This is falsifiable: if teams could reliably debug agent failures from timing spans, log files, and error rates alone, the six elements would be redundant. The article's own framing — that vendors across the market keep adding span filters because isolating relevant spans is an "ongoing operational challenge" — is offered as evidence that the gap is real and industry-wide, not a Progress sales angle.

## Key quotes

> If your AI trace doesn't explain decisions, it's just noise.

The sharpest line in the piece, and the one that generalizes beyond the vendor's platform: a trace that confirms infrastructure health is answering a question nobody asked. The interesting failures are decision failures.

> Standard OpenTelemetry answers none of these. It captures span timing, parent-child relationships and error codes—necessary, but not sufficient for debugging agent behavior.

Note the careful phrasing — "necessary, but not sufficient." The article is not anti-OTel; it builds on OTel plumbing and enriches at the collector. That's a more defensible position than the usual "telemetry is broken" take.

> **Trace volume** is easy to produce. **Trace hygiene**—capturing the right signals, in the right structure, with the right context—requires deliberate design.

This is the article's actual contribution, buried under the SDK tutorial: the industry's observability problem for agents is not under-instrumentation but undifferentiated instrumentation. Auto-instrument everything and you get a haystack.

> Cost calculation happens in the collector, not in your application. This matters more than it sounds: model providers change pricing with a blog post and a short notice period.

An underrated operational point. Pricing volatility is a fact of the current market, and embedding cost logic per-service turns every provider price change into a fleet-wide deploy. Centralizing it is an architecture decision, not a convenience.

> Without AI-specific context baked into the trace, every one of these failures is invisible until a user reports it.

The strongest argument for quality signals tied to traces: your users are currently your only output-quality detector, which means your worst incidents are discovered by the people least equipped to describe them.

## Critical analysis

The non-obvious insight here is the trace noise problem, and the article is honest that it's structural, not incidental: OpenTelemetry auto-instrumentation was *designed* to instrument everything, so LLM spans arrive pre-diluted among auth flows and HTTP client calls. The vendor-neutral implication is that agent observability needs a semantic layer — spans that reflect agent logic (`@agent`, `@workflow`, `@tool`, `@task`) rather than library calls — and that this layer must be designed, because no generic standard will infer your agent's intent. The decorator taxonomy is quietly the best idea in the piece: it makes the trace mirror the agent's own decomposition, which is what lets you tell the retrieval call from the answer-generation call without counting spans and guessing.

What's weak: this is, ultimately, a product tutorial, and it shows. The "six questions" framework is asserted via a link to the companion post rather than argued here; the quality-signal element — arguably the hardest and most interesting of the six — gets two sentences and a pricing note ("2 units consumed per evaluated span") with no discussion of what the evaluation actually measures or how much to trust it. The tradeoff table is refreshingly candid about the PII/debuggability tension, but the resolution ("toggle per deployment") sidesteps the harder question of how to debug production quality issues when `trace_content=False` blinds you exactly where production failures live. Redaction or sampling of content is never mentioned.

What's left out: any mention of open standards for agent telemetry (OpenTelemetry's GenAI semantic conventions, or trajectory formats like the ones [[atifact]] normalizes), any comparison with the wider observability ecosystem, and any treatment of trace cost at volume — capturing full prompt/completion content on every span is itself a storage and cost decision the article doesn't price. The serverless span-loss warning is genuinely useful operational detail, though, and rare in vendor posts.

## Related

- [[The Three Pillars of Observability]] — this source directly complicates that page's framework: metrics, logs, and traces built around request health are exactly the "necessary but not sufficient" layer the article argues fails for agent behavior, where the failure mode is a confident wrong answer behind an HTTP 200.
- [[atifact]] — a complementary take on the same problem from the opposite direction: atifact normalizes existing agent session logs into a standard trajectory schema post-hoc, while this article argues for enriching spans at instrumentation and collector time; together they bracket the question of whether agent observability should be retrofitted from logs or designed into the trace.
- [[AgentPulse]] — strengthens the same thesis from the analysis side: AgentPulse does drift detection and root-cause investigation over captured agent runs, which only works if the underlying traces carry the content and structure this article insists on — the two tools are the capture and investigation halves of one stack.
- [[Building an Advanced Agentic Harness]] — that tutorial includes a tracer as one of seven harness primitives motivated by failure modes; this source independently corroborates the claim that tracing is a first-class harness component, and adds the collector-side cost-attribution pattern the harness tutorial doesn't cover.

---
*Sources: [[raw/what-useful-ai-trace-should-actually-contain-how-to-build]], [[summary/what-useful-ai-trace-should-actually-contain-how-to-build]]*
