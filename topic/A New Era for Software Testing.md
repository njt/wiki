# A New Era for Software Testing

antirez (Salvatore Sanfilippo, creator of Redis) argues that while LLMs produce structurally mediocre code at high speed, they excel without compromise at QA and testing. His approach: give an AI agent a markdown QA checklist, point it at recent commits, and let it run integration tests, regression checks, and even UX audits that were previously manual and often skipped. The thesis is that automatic QA can raise the quality bar enough to compensate for the lower-quality code produced by automatic programming.

---

## Key Quotes

> "Covering all the lines of the code does not mean covering all the possible states."

The most compressed critique of line-coverage-as-quality I've seen. Traditional test suites are necessary but structurally incapable of covering the state space of any non-trivial system. antirez isn't dismissing unit tests — he's saying the gap between "tests pass" and "software works" is where QA lives, and that gap has been under-automated.

> The QA checklist should be reviewed "especially in light of the added commits," starting from inspecting the changes to understand what parts of the system are affected.

This is the key process innovation: the agent doesn't blindly execute a static checklist. It reads the diff first, identifies what might have broken, and *then* runs the tests. This is what a good human QA engineer does. The fact that an LLM can do it means we've been under-engineering QA automation — not because the tools didn't exist, but because nobody framed the problem as "give an agent a checklist and let it figure out the rest."

> The speed regression check does not require providing the agent with the numbers of a previous run.

This is a genuinely clever design choice. Hardcoding benchmark numbers into the checklist would create a maintenance burden (update the numbers every release) and a brittle test (slightly different hardware, different conditions). Instead, antirez lets the agent determine what constitutes a regression based on context. It's the same insight as [[Demystifying Evals for AI Agents]] — grade outcomes, not pathways.

> Asking the agent to find features that seem surprising, under-documented, or "generally sloppy from the POV of the user."

The extension of agentic QA into UX and psychology is the most underrated idea in the post. These were the checks that always got skipped in manual QA — not because they're unimportant, but because they're hard to systematize. An LLM agent doesn't get bored, doesn't have deadline pressure, and will methodically flag every rough edge. This is QA as user advocacy, automated.

> Automatic QA "maybe partially compensates for the lower quality of the code produced at high speed with the use of automatic programming."

The closing thesis, and it's characteristically honest. antirez doesn't claim AI-generated code is as good as hand-written code. He claims the *system* — fast generation + thorough automatic QA — can produce an acceptable outcome. It's [[Compound Engineering]] applied to the development lifecycle: add a QA system rather than slowing down generation.

---

## Key Themes

- **#concept Agentic QA**: The pattern of giving an AI agent a markdown checklist, letting it inspect recent changes, and running targeted integration/regression/UX tests. Distinct from traditional test automation in that the agent exercises judgment about *what to test* based on the diff, rather than blindly executing a fixed suite.
- **#pattern Checklist-Driven Agent Testing**: A specific implementation pattern: a single markdown file that serves as both the test plan and the agent's instructions. Minimal configuration (SSH endpoints, paths), no hardcoded benchmarks, designed for the agent to interpret contextually. This is the complement to [[Agentic Manual Testing]] — Willison focuses on making agents execute and observe; antirez focuses on making agents *design* the test plan.
- **#person antirez (Salvatore Sanfilippo)**: Continuing to develop his thinking on AI-assisted development. [[Automatic Programming]] established his stance on human-vision-driven coding; this post extends the argument to QA, completing the development lifecycle. He's building a coherent philosophy where AI is the accelerator and QA is the governor.
- **#tool DwarfStar (DS4)**: The inference engine antirez uses as his running example. [[DS4 (DwarfStar 4)]] covers the architecture in detail; this post shows how it's tested in practice — distributed inference across two MacBooks, GGUF compatibility verification, speed regression detection.

---

## Critical Analysis

**This is the companion piece that [[Automatic Programming]] needed.** That post argued that AI-assisted coding produces structurally weaker code but that human vision makes it viable. This post answers the obvious follow-up: "if the code is weaker, how do you keep quality from collapsing?" The answer — automated QA by the same class of AI that wrote the code — is elegant and a little unsettling. AI writes the code, AI tests the code. The human provides vision and the checklist.

**The checklist-as-interface is the stealth innovation.** Everyone's building MCP servers, custom tools, eval frameworks. antirez writes a markdown file and hands it to an agent. This is the [[Smart Models Dumb Pipes]] philosophy applied to QA: the model owns the judgment (what to test, what constitutes a regression), the "pipe" is just a markdown file with SSH credentials. It's aggressively simple in a way that makes most QA infrastructure look over-engineered.

**The uncomfortable implication:** If AI is both the code generator and the QA engineer, we're building a closed loop where AI checks AI's work. This works for regression detection and integration testing — things with observable failure modes. It works less well for the class of bugs antirez acknowledges exist: structural quality problems, complexity accumulation, architectural decay. An AI agent checking an AI's code for "structural quality" is like asking a student to grade their own essay for depth of thought. The checklist would need to encode architectural judgment, which is exactly the "vision" antirez says AI doesn't have (yet).

**Compared to [[Agentic Manual Testing]]:** Willison's approach is execution-as-verification — the agent runs the code and observes whether it works. antirez's approach is broader: the agent designs the test plan, executes it, *and* performs qualitative assessment (UX, documentation, "sloppiness"). Willison gives you a testing technique; antirez gives you a QA philosophy. They're complementary, not competing.

**Compared to [[Demystifying Evals for AI Agents]]:** Anthropic's framework is rigorous, structured, and designed for teams shipping agents in production. antirez's approach is a single markdown file and a `git log`. This isn't a criticism of either — they serve different scales. Anthropic is building evals for products with millions of users; antirez is shipping a personal inference engine. The insight is that agentic QA scales *down* better than traditional eval infrastructure. You don't need an eval harness when you can hand an agent a checklist.

**What's missing:** The post is a manifesto, not a manual. It doesn't describe what makes a good QA checklist, how to handle false positives (the agent flags something that isn't a bug), or how to iterate on the checklist as the system evolves. There's no discussion of token costs — running a full QA pass on every release consumes context and compute. And the claim that automatic QA "compensates" for lower-quality code is asserted, not demonstrated. It's a hypothesis worth testing, not a proven result.

**The meta-point:** antirez is one of the most influential systems programmers alive, and his development process now consists of: (1) write a vision in markdown, (2) let AI generate the code, (3) let AI test the code, (4) ship. This is the [[Agent Coding Workflow]] maturity spectrum at its most advanced: the human is architect, critic, and release manager. Everything else is automated.

---

## Related

- [[Automatic Programming]] — antirez's earlier essay on human-vision-driven AI coding; the thesis this post extends into QA
- [[DS4 (DwarfStar 4)]] — the inference engine used as the running example; architecture, key techniques, design tradeoffs
- [[Agentic Manual Testing]] — Willison's complementary approach: agents that execute and observe their own code
- [[Demystifying Evals for AI Agents]] — Anthropic's rigorous eval framework; the industrial-strength alternative
- [[Guardrails and Feedback Loops]] — hub page for quality assurance patterns in agentic development
- [[Agent Coding Workflow]] — the broader context: where agentic QA fits in the development lifecycle
- [[Smart Models Dumb Pipes]] — the architectural philosophy behind checklist-as-interface
- [[Compound Engineering]] — "add a system, not manual review"; agentic QA as a compound-engineering move
- [[Harness Engineering (OpenAI)]] — harness engineering as the discipline that makes agentic QA possible
- [[Polyglot Unit Testing Agent]] — Microsoft's open-source unit-test generation agent: quantitative validation of the workflow-over-model thesis across 152 tasks and 12+ languages, with a 63% failure reduction driven entirely by vague-prompt performance

---

*Source: [antirez.com/news/168](https://antirez.com/news/168)*
*Fetched: 2026-06-12*
