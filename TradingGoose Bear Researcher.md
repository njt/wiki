# TradingGoose Bear Researcher

A Supabase Edge Function implementing the adversarial Bear Researcher agent in TradingGoose's multi-agent stock analysis debate system. The code is a production reference for position-aware AI prompting, multi-round agent debate with memory, coordinator-worker orchestration, and the kind of harness engineering that turns an LLM from a chatbot into a specialized analyst.

---

## What It Does

TradingGoose runs a team of specialist AI agents -- Market Analyst, Fundamentals Analyst, News Analyst, Social Media Analyst, Bull Researcher, Bear Researcher, Risk Analysts, and a Portfolio Manager. The Bear Researcher receives the output of peer agents, builds a prompt that's deeply tailored to the user's actual portfolio state (what you hold, at what price, how much profit or loss, position sizing relative to limits), and constructs a bearish thesis that either urges taking profits, cutting losses, or staying out entirely.

It then enters a structured debate with the Bull Researcher across multiple rounds, each side building on and countering the other's previous arguments. The coordinator decides when to stop.

---

## Key Quotes

> "YOUR BEARISH STANCE: Position has strong gains -- argue for taking profits before reversal"

The prompt engineering here is the product. The agent doesn't just ask for "a bearish analysis" -- it constructs a specific adversarial role based on concrete portfolio numbers. This is [[Smart Models Dumb Pipes]] in practice: the LLM provides judgment, the harness provides context and constraints.

> "Directly counter the bull's specific arguments from Round {{currentRound}}. Do NOT simply repeat your previous points. Reference specific bull claims and provide detailed rebuttals."

The debate protocol is explicit about iteration quality. This solves the common agent failure mode where multi-round systems devolve into restating the same points with different words. The requirement to introduce NEW risks each round forces progressive deepening, analogous to the "recursive audit loops" in [[recursive-mode]].

> "CRITICAL: Position is oversized AND profitable -- perfect time to reduce to target allocation... Explain why holding at these levels is risky and greedy"

The position-aware prompting is genuinely sophisticated. It's not just "you're bearish" -- it's "here's exactly how bearish the numbers say you should be, and here's your rhetorical brief." This is the kind of structured role-playing that makes multi-agent debate systems produce useful synthesis rather than performative disagreement.

---

## Key Themes

#multi-agent #debate #orchestration #prompt-engineering #supabase #edge-function #trading #position-aware

The code is a clean example of the **coordinator/worker pattern** from [[Agent Orchestration]]. A coordinator dispatches agent functions, each runs independently, results flow back through shared database state. This is the same architecture Cursor found worked at scale ([[Scaling Long-Running Agents]]) -- flat self-coordination fails, hierarchy works. Here the hierarchy is coordinator → analysts → debaters → portfolio manager.

The **harness is more sophisticated than the model call**. The AI provider call (`callAIProviderWithRetry`) is maybe 10 lines. The surrounding harness -- position context construction, debate round history injection, error categorization, atomic updates, cancellation checks, self-retry timeouts -- is 300+ lines. This validates the thesis from [[Honey I Shrunk the Coding Agent]] and [[Components of a Coding Agent]]: the scaffold matters more than the model.

The **position-aware prompt construction** is the most interesting engineering decision. Rather than passing raw portfolio data to the LLM and hoping it does the right math, the harness pre-computes thresholds (near max, above max, below min, near min), calculates room-to-reduce percentages, and injects explicit rhetorical stances. This is [[Harness Engineering]] in the feedforward mode: the system shapes what the LLM sees, not what the LLM outputs.

The **error taxonomy** is production-grade: `rate_limit`, `api_key`, `ai_error`, `data_fetch`, `database`, `timeout`, `other`. Each gets different handling. The system distinguishes between "AI couldn't respond" (retry) and "API key is bad" (stop, notify user). This is the kind of operational thinking that separates demo agents from production agents.

---

## Critical Analysis

**The debate pattern is under-specified.** The code requires the bear to counter the bull's specific arguments, but it truncates bull arguments at 800 characters in the prompt. At 800 chars you're debating a summary of a summary. If the bull made a nuanced argument about, say, the inflection point in a specific business segment's margins, you've lost the nuance by the time the bear sees it. The architecture is sound but the context window budgeting is aggressive.

**The "bear points" are hardcoded fallbacks.** The agent output includes `bearPoints` like "Valuation stretched at current levels" -- but these are the same defaults regardless of what the AI actually said. If the AI's analysis contradicts these, the summary and the analysis diverge. This is a known anti-pattern: [[Prefix Effects]] warns that early naming decisions create gravity. A hardcoded summary does the same.

**No model of the other side's model.** The bear receives the bull's output but has no explicit instruction about the bull's likely biases, blind spots, or rhetorical strategies. A truly adversarial system would model "what the bull will conveniently ignore" and probe there. This could be improved with a meta-prompt about common bullish cognitive biases (survivorship bias, recency bias, narrative fallacy).

**The position calculus is sophisticated but fragile.** Profit targets, stop losses, min/max position sizes, near-threshold percentages -- these are all configurable, which is good. But the prompt construction is a long chain of if/else branches in TypeScript. Adding a new position state (e.g., "covered call in place") would require surgery. This is the tension between [[Specifications as the Product]] (the prompt logic IS the spec) and [[Compound Engineering]] (the prompt logic needs systematic verification).

**The error handling is better than most, but not great.** The agent returns HTTP 200 with `{success: false}` on errors. This is the Supabase Edge Function pattern -- you want to avoid HTTP error codes because Supabase's retry behavior on non-200 responses is opaque. But the caller now has to check both HTTP status and response body. The [[Designing a Passively Safe API]] principle says: after any failure, land in a visible terminal state. This API lands in a middle state where the caller must parse the body to know what happened.

---

## Connections

- [[Agent Orchestration]] -- the coordinator/worker pattern this implements
- [[Scaling Long-Running Agents]] -- Cursor's finding that hierarchy beats flat coordination, validated here
- [[Smart Models Dumb Pipes]] -- the LLM as judgment machine, harness as execution
- [[Harness Engineering]] -- the feedforward prompt construction is textbook harness work
- [[Honey I Shrunk the Coding Agent]] -- the scaffold matters more than the model
- [[Components of a Coding Agent]] -- this is a well-factored harness, not a raw model call
- [[Elements of Agentic Systems Design]] -- the debate round tracking is Memory and Coordination
- [[Designing Agentic Loops]] -- the multi-round debate with progressive deepening
- [[Scaling LLMs to Larger Codebases]] -- prompt libraries as feedforward, this agent's prompt construction as a case study
- [[Building Agents for Production Systems with MCP]] -- production agent patterns (though this uses shared DB state, not MCP)
- [[Specifications as the Product]] -- the prompt IS the spec for this agent's behavior
- [[Designing a Passively Safe API]] -- the HTTP 200 + `success: false` pattern and its tradeoffs
- [[Prefix Effects]] -- the hardcoded bearPoints as a naming-decision-gravity trap
- [[Correct by Construction]] -- the position calculus would benefit from this approach

---
*Sources: [[raw/tradinggoose-bear-researcher]]*
*Last updated: 2026-05-15*
