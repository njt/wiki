# OpenAI Structured Outputs

OpenAI's Structured Outputs guarantees model responses conform to a user-supplied JSON Schema — no more retry loops for malformed JSON or prompt-engineering "please respond in valid JSON." Two API surfaces: function calling for tool connections, `response_format`/`text.format` for shaping user-facing output. SDK-native via Pydantic (Python) or Zod (JavaScript) auto-generating schemas from typed objects. Available since `gpt-4o-2024-08-06`.

---

## Key Quotes

> "Reliable type-safety — no need to validate or fix incorrectly formatted responses"

This is the headline promise. JSON mode guaranteed valid JSON but not the right JSON. Structured Outputs closes that gap by enforcing the schema at the protocol level, not the prompt level. This is the API equivalent of [[Feedback Loop is All You Need]]'s thesis: deterministic enforcement beats pleading in natural language.

> "Explicit refusals — safety-based model refusals are programmatically detectable"

Previously, if a model refused to answer for safety reasons, you got unstructured text back and had to pattern-match your way to detecting it. Now the `refusal` field on the message object makes refusals a structured signal your code can handle — no parsing, no heuristics.

> "Simpler prompting — consistent formatting without strongly worded prompt engineering"

The quiet admission: before Structured Outputs, getting reliable JSON required threatening the model. The API now handles it so your prompt doesn't have to.

> "A well-built outer harness serves two goals: it increases the probability that the agent gets it right in the first place, and it provides a feedback loop that self-corrects."

Not from this document — from [[Harness Engineering]] — but Structured Outputs is the purest feedforward harness OpenAI ships. The schema constrains the output space before the model generates a single token.

---

## Key Themes

#tool #pattern #concept

- **Schema as protocol, not suggestion** — The `"strict": true` flag and mandatory `"additionalProperties": false` turn the schema from a hint into a contract. Every property must be in `"required"`, every object rejects extra keys. This is the API equivalent of a linter that fails the build rather than printing a warning.
- **SDK types as the single source of truth** — Pydantic and Zod schemas auto-generate the JSON Schema. You define your types once in your language's idiom; the API consumes them. This eliminates the dual-maintenance problem where your API schema and your application types drift apart. [[Swamp Club]] uses the same Zod-typed pattern for agent workflows.
- **Recursive schemas for nested structures** — `z.lazy()` (Zod) and forward-referenced Pydantic types with `model_rebuild()` support recursive UI trees and other nested patterns. This is how [[json-render]]'s component catalog pattern works: a `UI` node can contain `children: List[UI]`, constrained by an enum of allowed component types.
- **Two surfaces, one constraint engine** — Function calling and `response_format` share the same underlying schema enforcement. The rule of thumb (function calling for tools, `response_format` for user-facing output) is pragmatic API design, not a technical distinction.
- **The first-request tax** — Fine-tuned models pay a one-time latency cost per unique schema as the API processes it. Subsequent requests skip it. This is a caching detail that matters in production: warm your schemas.

---

## Critical Analysis

**What's strong:** The SDK integration is the real product here. `client.chat.completions.parse()` returning typed objects (Python) or `zodResponseFormat()` generating schemas from Zod types (JS) means developers never touch raw JSON Schema unless they want to. This is the DX moat — Anthropic's API has structured outputs via tool use, but the SDK ergonomics around Pydantic/Zod are less polished. OpenAI understood that the feature isn't the schema engine; it's the developer never having to write `json.loads()` and pray.

**What's weak:** The "not all JSON Schema features supported" caveat is doing a lot of work. The page links to a supported-schemas section but doesn't surface the restrictions prominently. If you're migrating from JSON mode and your existing schemas use unsupported keywords, you won't know until the API rejects them. This is the classic "check the fine print" API design — powerful when it works, opaque when it doesn't.

The limitation list (no `oneOf`/`anyOf`, restricted `pattern` support, limited numeric constraints) means Structured Outputs covers the 80% use case well but falls apart for complex validation that requires conditional sub-schemas or regex pattern matching. Teams with elaborate schemas should test against the supported subset before committing.

**Missing from the docs:** No guidance on schema design for LLMs specifically. Some schemas are easier for models to produce than others — flat structures beat deep nesting, descriptive enum values beat cryptic codes, field descriptions matter more than you think. The docs treat the schema as a purely technical constraint when it's also a prompting surface: the schema's field names and descriptions shape what the model generates.

**How it connects:** The 45x cost gap documented in [[Computer Use is 45x More Expensive Than Structured APIs]] is exactly what Structured Outputs exploits — structured data is cheaper, faster, and more reliable than pixel-parsing. Structured Outputs makes it easy to add structured endpoints to your own applications, which is the supply-side answer to the demand-side argument that computer vision is the fallback for systems you can't modify.

The [[Harness Engineering]] connection runs deeper: Structured Outputs is a feedforward computational control — deterministic, fast, applied before generation. It belongs in the same toolbox as type checkers and lint rules, not in the same bucket as "please format this correctly" prompts. The engineering insight is that you should push as much constraint as possible into the schema layer so your prompts only have to handle what's left.

For [[Building Agents for Production Systems with MCP]], Structured Outputs is complementary: MCP provides the transport and tool definitions; Structured Outputs ensures the agent's responses conform to the expected shape. The two together are the structured-output stack.

DSPy ([[DSPy — Programming Not Prompting]]) is the input-side analog of Structured Outputs: where Structured Outputs enforces output shape at the API level, DSPy replaces hand-written prompts with typed signatures that the framework compiles into optimized prompts. Together they define a stack where neither prompt engineering nor output parsing is the developer's problem — the specification layer handles both boundaries.

Courtney Yatteau's dynamic content gamification demo ([[AI-Powered Gamification for the Web]]) is the cleanest public example of why structured output matters for UI-facing AI features. Her schema defines exactly three fields (headline, fact, reward label) that map directly to different UI elements — "the response shape really matches what the interface actually needs." Without structured output, each field would require parsing from free text; with it, the response is immediately consumable. The insight generalises: when AI output feeds a UI, the schema is more important than the model's prose quality.

[[Morningprint]] is the Anthropic-ecosystem equivalent in production — `output_config.format` with a JSON schema driving daily thermal-receipt art — and its July 2026 title-poisoning incident surfaces a gap neither ecosystem documents: structured output guarantees JSON shape, not field length, and a model that fills a string field with 3,155 characters of filler can poison a rolling context archive. The fix (`validateArtSpec`) is a content-validation layer on top of schema validation, applied before the spec reaches storage or paper.

---

## Cross-Links

- [[Computer Use is 45x More Expensive Than Structured APIs]] — the economics that make Structured Outputs a hard requirement, not a nice-to-have
- [[Harness Engineering]] — Structured Outputs as feedforward computational control: deterministic, pre-generation, schema-level constraint
- [[Feedback Loop is All You Need]] — schema enforcement as the API equivalent of "linters beat prompts"
- [[Guardrails and Feedback Loops]] — structured output as a guardrail mechanism: constrain the output space, don't plead with the model
- [[json-render]] — consuming structured outputs for progressive UI rendering; the same schema-constrained pattern
- [[Swamp Club]] — Zod-typed models as the agent-native pattern for structured data workflows
- [[Smart Models Dumb Pipes]] — schemas as the dumb pipe: models make decisions, schemas enforce the contract
- [[Building Agents for Production Systems with MCP]] — MCP + Structured Outputs as the structured-output stack for agent-to-application communication
- [[Spec-First Development at Benchling]] — schemas as the single source of truth, consumed by multiple consumers including LLMs

---
*Sources: [[summary/openai-structured-outputs]]*
*Last updated: 2026-05-14*
