# AI for Product Management

Rian van der Merwe's opinionated system for using LLMs as product-thinking partners — not ghostwriters. A three-layer prompt architecture (system prompts, personal context, reference materials) composed on the fly via Windsurf's `@` mentions, grounded in company data through MCP servers. The author still writes their own PRDs and strategy docs; the AI challenges weak reasoning, spots missing criteria, and rehearses ideas before stakeholders see them.

---

## Key Quotes

> "the magic isn't in any single prompt—it's in how you combine them"

The composability insight. Most people hunt for the One Perfect Prompt; van der Merwe builds a library of small, sharp tools and layers them. Same pattern as Unix pipes vs monolithic programs.

> "an assistant that pushes back on bad ideas"

Deliberately designed into the prompts. This is the opposite of the sycophantic default most LLM products ship with. It costs more effort to get here — you have to write prompts that invite disagreement — but the output is actually useful rather than merely pleasant.

> "Context tells the model who you are, what you're working on, and what 'good' looks like. Constraints keep the model from going off the rails with generic advice."

The two-lever theory of LLM steering. Context is identity and standards; constraints are guardrails. Both are needed. Context without constraints produces plausible-sounding hallucination; constraints without context produces correct-but-irrelevant output.

> "turns the AI from a general-purpose assistant into something more like an expert who has access to your company's actual knowledge base"

This is the MCP thesis in one sentence. Grounding in real data is what separates a toy from a tool. The author's instruction to "always cite sources with links so I can verify" is the essential trust mechanism — without it, you're just hoping.

> "I still write my own PRDs, OKRs, and strategy docs"

The line he refuses to cross. AI handles research, critique, and rehearsal. The artifacts that represent decisions and accountability remain human-produced. This is a sharper boundary than most "AI for PM" content draws.

> "Less context is often more"

Counterintuitive but consistent with everything we know about prompt engineering. More context doesn't linearly improve output — it dilutes signal. Start minimal, add only what the model demonstrably needs. This is the same principle behind [[CLAUDE.md (Universal)]]'s six-rule brevity.

---

## Key Themes

- **#concept** "Sparring partner, not ghostwriter" — The AI's role is to challenge thinking, not produce artifacts. This inverts the typical sales pitch for AI productivity tools.
- **#tool** [[Windsurf]] — The IDE as prompt composition surface. `@` mentions let you build the assistant from parts: system prompt + context files + current document.
- **#pattern** Three-layer prompt architecture — System prompts (role/behavior), personal context (identity/standards), reference materials (formatting/domain rules). Layered, not monolithic.
- **#concept** MCP grounding — Connecting the AI to internal wikis, docs, and code turns it from generic oracle to domain expert. Citation requirements make the output verifiable.
- **#pattern** Record-keeping as first-class workflow step — Save good critique to a `work/` folder organized by topic. The conversation isn't the artifact; the extracted summary is.
- **#person** Rian van der Merwe — Product leader and Elezea blogger. Opinionated practitioner who's been refining this system over months of daily use.

---

## Critical Analysis

**What's genuinely useful:** The three-layer architecture is the right abstraction. Most people either write monolithic mega-prompts or ad-hoc one-shots. Van der Merwe's system recognizes that different jobs need different system prompts, but they all share the same personal context and reference materials. This is modular without being over-engineered. The `work/` folder practice — extracting and saving good output — is the habit most people skip and most need.

**What's undersold:** The author says "less context is often more" but doesn't give heuristics for deciding when. How do you know you've crossed the line from "enough" to "too much"? The follow-up post might address this, but in this article it's a hand-wave. Similarly, the prompt iteration advice ("regular updates based on what works") is correct but thin — what's the feedback mechanism? How do you know a prompt got *worse* rather than the model having an off day?

**What's missing:** There's no discussion of model selection. Do these patterns work equally well with Claude vs GPT vs Gemini? The Windsurf dependency is a single point of failure — if Windsurf changes its `@` mention behavior or pricing, the whole workflow breaks. A file-based approach (like [[Planning With Files]] or the [[Agent Coding Workflow]] pattern) would be more portable.

**The hard question:** This is a system built by someone who already knows what good PM looks like. Could a junior PM use this to *learn* good PM, or does it only amplify existing skill? The author's insistence on writing their own PRDs suggests the latter — the AI critiques, you improve, but you have to know what improvement looks like. This is the [[Radical Accountability]] problem: AI amplifies taste, it doesn't create it.

**The meta point:** This article itself is an example of the system working. Van der Merwe documented his approach, shared it publicly, and linked to a follow-up about how it evolved. The "record keeping" habit produced the blog post. That's genuinely elegant — the system that helps you think also helps you communicate what you thought.

---

## Related Pages

- [[Talking to Transformers]] — The prompting theory underneath: attention as budget, domain language as compression
- [[Writing a Good CLAUDE.md]] — The same "less is more" philosophy applied to AI instruction files
- [[CLAUDE.md (Universal)]] — Six rules that encode the "context and constraints" pattern in token-efficient form
- [[Building Agents for Production Systems with MCP]] — Anthropic's guide to the MCP grounding layer van der Merwe relies on
- [[Agent Memory and Context]] — Context management as the real engineering challenge
- [[Specifications as the Product]] — The artifact/human-boundary parallel: specs are durable, code is disposable
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, echoing the "sparring partner" role
- [[Agent Coding Workflow]] — Another practitioner's daily loop, from a developer's perspective
- [[Radical Accountability]] — AI eliminates the excuse of insufficient time; taste is all that's left
- [[Slowing the Fuck Down]] — Deliberate friction as a feature; the pushback-is-a-feature parallel
- [[How Boris Uses Claude Code]] — Creator's usage patterns, different tools but similar composability philosophy
- [[Feedback Loop is All You Need]] — Linters beat prompts; the constraint principle in a different domain
- [[Two Kinds of User Are Emerging]] — Van der Merwe is the archetypal power user: domain expert + AI fluency
- [[Cognitive Debt]] — What happens when you let the AI write the PRDs instead of just critiquing them

---

*Sources: [[summary/ai-for-product-management]]*
*Last updated: 2026-05-14*
