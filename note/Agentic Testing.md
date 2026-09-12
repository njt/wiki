# Agentic Testing

Slack Engineering's empirical study of where LLM-driven testing fits in the E2E stack: 200+ runs across three execution models (Playwright MCP, Playwright CLI, generated Playwright tests) and two test flows, run against test workspaces with non-production data. The core finding: agentic testing adds a new exploratory layer on top of deterministic E2E tests — it doesn't replace them. The infrastructure matters as much as the model: MCP's persistent DOM view delivered near-zero failure on simple scenarios, while CLI's snapshot-per-step approach failed 12–20% of the time with the same underlying LLM. And the brutal reality check: AI-generated deterministic tests fail 48% of the time on complex workflows, breaking at the last interaction or assertion 70–80% of the way through.

---

## Key Quotes

> "Tests enforce journeys. Agents verify goals."

The article's crispest distinction. Traditional E2E tests validate a specific path through the UI (click → click → type → assert). Agent-driven tests validate whether a *goal* can be achieved, allowing the agent to adapt its path. This isn't just a semantic difference — it changes what "passing" means. A deterministic test passing means "the journey we specified works." An agentic test passing means "the goal is achievable via some valid path." Both are useful; neither replaces the other.

> Only about 20% of runs followed the exact same action sequence. Agents discovered different valid UI paths — different input methods, navigation patterns, and step ordering — while still reaching the same goal state.

This is simultaneously the best and most dangerous property of agentic testing. On the upside: agents find paths humans wouldn't think to test, surfacing real bugs that deterministic tests miss. On the downside: non-reproducible test results are a CI nightmare. If a test fails one in five runs and took a different path each time, debugging it requires replaying the agent's session, not just reading a stack trace. The article frames this as pure upside; I think the reproducibility tax is real and under-discussed.

> CLI failures were predominantly authentication and navigation issues — "failures were caused by the execution layer rather than the agent's reasoning."

This is the article's most important finding and it's buried in the results section. The same LLM (Claude Sonnet 4.5) produced dramatically different reliability depending on how it was connected to the browser. MCP's structured primitives and persistent DOM view beat CLI's string-of-shell-commands approach by 12–20 percentage points. This validates the [[Components of a Coding Agent]] thesis: the harness matters more than the model. If you're building agentic testing, invest in the execution infrastructure first, the model choice second.

> The majority of cost came from retransmission of previously seen content, not model reasoning.

$15–30 per run, dominated by context accumulation. CLI took 85 turns to MCP's 40–60 because each browser interaction was split across multiple shell commands, each carrying the full conversation history. Code Gen's context ballooned from test runner output with full error traces and DOM state on each retry. This is the same dynamic as [[Computer Use is 45x More Expensive Than Structured APIs]] — the cost isn't the model thinking, it's the model *re-reading*. Prompt caching and context compaction are the obvious optimization surface.

> Generated tests typically progressed through 70–80% of the flow before breaking on a final interaction or assertion.

The 48% failure rate on complex workflows is a gut check for anyone who thinks "just have the agent write the tests" is a solved problem. These aren't failures early in the flow where the agent is confused — they're failures at the last mile, where "variability in UI state and abstraction mismatches" (the article's phrasing) cause the generated code to make incorrect assumptions about what the DOM will look like. This is the same class of problem that makes [[Webwright]]'s approach (code-as-action with self-verification) so effective — the agent sees the actual state, not its prediction of the state.

---

## Key Themes

- **#concept Agentic Testing**: Goal-oriented testing where an LLM agent explores the UI to verify that a goal is achievable, rather than following a predetermined path. Complements deterministic E2E tests; does not replace them. Currently best suited for debugging, exploration, and reproducing production bugs — not CI.
- **#pattern MCP for Browser Automation**: Microsoft's Playwright MCP server provides structured browser primitives that dramatically outperform shell-based Playwright CLI for agent-driven testing. The key advantage is persistent DOM context and tighter tool-calling integration. This is the same pattern as [[Building Agents for Production Systems with MCP]] — MCP as the standard agent-to-tool integration layer.
- **#pattern Test Generation with Iterative Refinement**: The "generated Playwright tests" model: AI writes deterministic test code, executes it, inspects failures, and iteratively refines until passing. Works for simple flows (~8% failure) but degrades on complex ones (~48% failure). Fast once built (~3 min total, ~32-45s raw execution).
- **#tool Playwright MCP**: Microsoft's Model Context Protocol server for Playwright. The most reliable execution model in the study, with near-zero failure on simple scenarios. Persistent DOM context and structured tool definitions beat ad-hoc shell commands.
- **#tool Playwright CLI**: Command-line interface for Playwright. Less reliable than MCP due to snapshot-per-step approach and authentication/navigation timing issues. Still useful for simpler automation tasks.
- **#person Sergii Gorbachov**: Staff Software Engineer at Slack, on the Frontend Test Frameworks team. One of the few practitioners publishing rigorous empirical comparisons of agentic testing approaches rather than anecdotal reports.

---

## Critical Analysis

This is the best empirical study of agentic testing I've read — not because the conclusions are surprising, but because they're *quantified*. 200+ runs across a controlled experiment matrix beats the usual "we tried it and it seemed good" blog post by a mile. The Slack team deserves credit for doing the work.

**The MCP vs CLI finding is the article's lasting contribution.** Same model, same tasks, 12–20% reliability gap driven entirely by execution infrastructure. This should kill the "just use a better model" reflex dead. If Slack — with their engineering resources and internal MCP servers — sees a 12% failure rate on *simple* flows with CLI, your startup's bash-script-and-pray approach is going to be worse.

**The 48% generated-test failure rate is under-analyzed.** The article says failures were driven by "variability in UI state and abstraction mismatches" but doesn't dig into what kinds of variability or what specific abstractions broke. This is where the practitioner needs operational detail — are we talking about timing, DOM structure, data state, or something else? The answer changes how you'd mitigate it.

**The reproducibility gap is real and the article mostly sidesteps it.** "Only 20% of runs followed the same path" is presented as a feature (agents discover novel paths!). But for debugging a production issue, you need to know *why* the agent did what it did, and whether the path it took is even relevant to the bug you're trying to reproduce. Agentic testing for reproduction needs session replay and explainability — the article acknowledges execution boundaries exist but doesn't propose solutions.

**The cost analysis is honest but the optimization path is speculative.** Prompt caching and context compaction are mentioned as promising avenues, but the article doesn't test them. The real question is whether you can get agentic testing costs down to the point where it's viable for pre-merge checks, not just ad-hoc debugging. At $15–30/run, the answer is no. At $1–3/run with cached context? Maybe. The article doesn't give us that data point.

**Comparison to related work:** This is the complement to [[A New Era for Software Testing]] — antirez argues for agentic QA as compensation for lower-quality generated code; Gorbachov provides the rigorous empirical data on what agentic testing actually costs and where it fails. Together they make the case that agentic testing is real and useful, but not free and not a replacement for deterministic tests. [[Agentic Manual Testing]] focuses on agents executing and observing their own code; this article extends that to agents exploring *other people's* UIs. [[Accordant]] takes the opposite approach — spec-driven deterministic test generation — and the contrast is instructive: Accordant's approach works when you can formally specify behavior; agentic testing works when you can't.

The article's proposed four-layer testing pyramid (unit → integration → E2E → agentic) is sensible but incomplete. The missing layer is *observability-driven testing* — using production telemetry to generate test scenarios. If you're going to have agents explore your UI, they should be exploring the paths your users actually take, weighted by frequency and error rate. That's the synthesis neither this article nor the related work has made yet.

**A deeper layer: validating the tests themselves.** [[Test Validation and the Trustworthiness of Tests]] argues that the next testing bottleneck isn't generation or execution — it's knowing whether the tests in your suite are trustworthy. Typemock's thesis converges with Slack's empirical finding from the opposite direction: Slack shows that AI-generated tests fail 48% of the time on complex workflows; Typemock argues that even passing tests can be fragile, duplicated, or verifying the wrong behavior. Together they make the case that the test suite itself — especially an AI-augmented one — needs its own quality audit layer. Runtime analysis (what does this test actually *do*?) complements Slack's execution-layer analysis (what path did the agent actually *take*?).

AMD's [[Execution-Free Agentic Program Repair]] takes the opposite approach for a different problem: when tests don't exist and can't be run (industrial C++ with multi-hour build cycles), they replace test-based validation with CppCheck static analysis + an LLM judge. The execution-free pattern is complementary to agentic testing — one handles the case where tests exist but you need exploratory coverage; the other handles the case where tests don't exist at all.

Microsoft's [[Polyglot Unit Testing Agent]] (`code-testing-generator`) fills the unit-test generation layer of this stack: an agent that learns repo conventions before writing tests, verifies the build system can find them, and performs lightweight mutation testing to catch vacuous assertions. Its 92.1% vs. 78.9% completion rate on 152 tasks is the quantitative counterpart to Slack's qualitative finding that agentic testing adds a new layer on top of deterministic tests — the agent doesn't replace existing test infrastructure, it makes it easier to build.

---

*Source: [Slack Engineering Blog](https://slack.engineering/agentic-testing-where-agents-fit-in-the-e2e-testing-stack/), Sergii Gorbachov, 2026-06-11. Fetched 2026-06-21.*
