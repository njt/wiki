# No Escape, No Leak — Egor Kraev on Structured Objects

Egor Kraev's conference talk (preserved as a ytx transcription gist) argues that the most impactful production use of GenAI is making LLMs emit structured objects reliably, every time — and that every standard approach to guaranteeing this leaks. His fix is to make a deterministic validator the only exit from the agent loop ("there is no escape, there is no leak"), to keep objects outside the LLM and expose only semantically natural action tools, and to treat the LLM as "a little box surrounded by other boxes" rather than a conductor.

---

## The Argument

Kraev — ML "since the last millennium," ex-investment banking, built Wise's AI team from scratch to 30 people, now CTO of Motley AI — opens from a place of hype-cycle fatigue: "I don't care about building what's sexy, I care about building what works. What works every time?" Since downstream systems "expect objects of a very defined shape," the value of an LLM lies entirely in whether its output can be *guaranteed* valid: schema-valid, semantically correct, names and amounts matching the input.

The escalation ladder he climbs, each rung leaking into the next:

- **Prompt + parser + hope** — "probably will not work."
- **JSON mode** — valid JSON, wrong schema.
- **Function calling / `with_structured_output`** — schema guaranteed, custom validators and semantics not.
- **Retries** — "try again, try again, you might get lucky."
- **Validator-as-a-tool loop** — the classic agentic pattern, and "exactly what an AI agent is" — but the LLM gives up after 3–4 tries and returns something anyway, and even a success is only relayed by the LLM, so what comes back may not be what was validated.
- **`return_direct`** — the tool returns the validated result directly, bypassing the LLM. One leak closed; the give-up leak remains.
- **The validator as the only exit** — his Motley Crew agent flag: the agent *cannot* return except through the validator. Fail X times, get `None`. "At least you get a clear failure."

Beneath the loop fix sit three object-handling patterns. **Ship in a bottle**: the object lives outside the LLM, which acts on it only through tools like *add a series* or *change chart type* — because "if you just say, here's the original thing, change X, it will change X, it will also change Y, Z, A, B and C all over the place." **Semantic layers instead of SQL**: expose a small pydantic query surface over metrics and dimensions, convert to SQL with deterministic machinery that is "stupidly easy to make work." **The intermediate stripped-down object**: even when the final artifact is complex, the LLM produces a minimal object containing only the fields it should decide, and the validation function converts it into the real thing — used to turn a loose prose prompt into full SQLAlchemy schemas with foreign keys.

## Key Quotes

> "I don't care about building what's sexy, I care about building what works. What works every time?"

The framing of a man who has watched several hype cycles come and go. The talk's entire moral stance is compressed here: reliability over novelty.

> "You cannot trust the LLM to reliably relay its input to the tool. You just can't, because it's not what it does."

The recurring "law of nature" of the talk. This is why his tools get direct access to the agent's original input rather than an LLM-passed copy — wiring he thinks "should come as standard."

> "There is no escape, there is no leak."

The title pattern. The agent flag forces return through the validation tool, so a successfully returned object *is* a validated object, by construction rather than by hope.

> "It's pretty good at obeying instructions. It's just not good at not doing other stuff as well. And the tools enforce that."

His answer to why LLMs call tools fine but mangle everything else — the sharpest one-sentence diagnosis of LLM agency in the talk.

> "I laugh evilly whenever I see anybody declare reliable security SQL generation."

His bluntest dismissal. SQL's expressiveness is precisely what makes LLM-generated SQL unverifiable: "you could probably write a SQL query that could compose the works of Shakespeare if you tried hard enough."

> "You don't have atomic transactions as tools. You have meaningful semantically natural actions."

His prescription for MCP server design — the ship-in-a-bottle principle generalized to the tool-protocol layer.

> "The poor LLM has no choice but to return valid output. So as long as it's something released this year, it's probably going to work."

The side effect of validating everywhere: model choice stops mattering. Claimed, not demonstrated — see analysis below.

> "Then you get back into the stochastic country."

On LLM-as-judge validators for multimodal output: the whole edifice rests on *deterministic* validators, and substituting an LLM judge forfeits the only guarantee the talk sells.

## Key Themes

- #concept **Guaranteed validation over best-effort output** — the difference between "the LLM thinks it's valid" and "it is valid" is the difference between customer care ("you get shitty customer care, but your company can survive it") and regulator-facing reports ("your company may not survive it").
- #pattern **Validator-as-only-exit** — return through the tool or return `None`; clear failure beats plausible garbage.
- #pattern **Ship in a bottle** — objects outside the LLM, semantically natural action tools, same principle for MCP servers.
- #pattern **Off the critical path** — LLM produces a validated complex object once; the object, not the LLM, serves production traffic. "Squishy inputs, but hard outputs."
- #tool Pydantic, LangChain's `with_structured_output`, Motley Crew (his open-source framework), headless Claude Code for code generation.
- #person **Egor Kraev** — the anti-hype productionist voice: "AI is just a marketing term."

## Critical Analysis

This is the most articulate statement in the wiki's corpus of the **LLM-as-unreliable-messenger** position, and it is refreshingly concrete: instead of exhorting models to behave, it changes the control flow so misbehaviour cannot escape. The escalation ladder is a genuinely useful diagnostic — most teams are stuck on rung three and mistake schema validity for correctness. The semantic-layer argument independently converges with [[Beyond the Warehouse — Data Stacks That Actually Work]], which reaches the same conclusion (narrow interface, deterministic machinery beneath, agents never touching raw data) from the data-engineering side — two sources, two industries, one pattern is a strong signal.

But the talk undercuts itself at its most important point. A talk *titled around guarantees* concedes that "there is no guarantee that the validation loop will ever conclude," and covers the gap with "so far, restarting the loop has always worked" — an anecdote standing in for the very guarantee being sold. The gist's own omissions list is unsparing, and correct: zero numbers (no iteration counts, failure rates, latency, cost), no distinction between validity and *quality* (a validator-passing object can still be semantically wrong in ways the validator doesn't check), no security story (he mocks "reliable secure SQL generation" yet never addresses what happens when attacker-influenced data feeds the semantic layer), no human-in-the-loop discussion for the regulator-facing use case, and a conflict of interest left unexamined — the central solution is his own framework, with LangGraph's equivalent capability conceded only in passing during Q&A. He also admits he hasn't tried [[DSPy — Programming Not Prompting]] or Guardrails, which weakens the novelty claim for a pattern other ecosystems plausibly implement.

The "model doesn't matter" claim is asserted, not shown, and sits in tension with widespread experience that weaker models struggle with tool-calling discipline. And multimodal output is punted honestly — "I'm not sure how you would do this" — which is really an admission that the entire approach is bounded by the availability of deterministic validators. That boundary is the talk's most interesting unresolved edge: everything outside it is "the stochastic country."

## Related Pages

- [[OpenAI Structured Outputs]] — Kraev's ladder begins at exactly what that page treats as the end state: schema-guaranteed output is rung three, not the summit; structured outputs guarantee shape, and the whole talk is about the guarantees that begin where that guarantee ends.
- [[Parse Don't Validate]] — Alexis King's parse/refine distinction, applied to LLM output: Kraev's intermediate stripped-down pydantic object plus a validation function that *converts* it into the real artifact is parse-don't-validate performed at the LLM boundary.
- [[Beyond the Warehouse — Data Stacks That Actually Work]] — independent convergence: both sources prescribe a semantic layer with deterministic machinery between the LLM and the database, and both treat the narrow interface as the point rather than a limitation.
- [[Text-to-SQL in the Real World]] — Stonebraker and Chen's 10% accuracy on real warehouses is the empirical bill of goods behind Kraev's evil laugh at "reliable security SQL generation"; both argue the freeform SQL interface is the problem.
- [[Guardrails and Feedback Loops]] — the validator-as-only-exit loop is this topic's purest instance: a deterministic check gating the only path to production, with failure surfaced as `None` instead of plausible garbage.

---
*Sources: [[raw/3ef64cb852dadd4e90234c69b82252bb]], [[summary/3ef64cb852dadd4e90234c69b82252bb]]*
*Last updated: 2026-09-13*
