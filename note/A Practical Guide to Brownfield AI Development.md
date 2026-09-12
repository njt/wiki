# A Practical Guide to Brownfield AI Development

Daniel Pupius's field guide to making AI coding agents productive in legacy codebases — the problem everyone has but few write about honestly. The core argument: brownfield systems lack the structural guardrails agents need (tests, clear boundaries, documented intent), but you can build those guardrails incrementally. The end state isn't a rewrite; it's a legacy system an agent can safely modify.

---

## Key Quotes

> "Real systems carry a lot of invisible context."

The article's thesis in eight words. This is why the opening anecdote — an 8-year-old Django monolith where Claude's "clean, confident patches quietly broke integrations" — resonates. The patches were syntactically correct and semantically wrong because the agent couldn't see the invisible contracts. Pupius's entire framework is built around making that context visible.

> "Agent autonomy requires structure, but legacy systems don't have it."

The brownfield bind. Greenfield projects give agents clear module boundaries, type systems, and fresh tests. Brownfield projects give them accumulated quirks, undocumented data flows, and the engineer who understood the auth system left three years ago. Pupius's move is to treat this not as a reason to abandon AI but as a requirements list: what structure do we need to add?

> "AI agents excel at local transformations but lack global context."

This is the architectural justification for tests-as-system-boundaries. The agent can change a component locally; the test suite verifies the global invariants still hold. Tests are the negative feedback mechanism preventing local changes from becoming global regressions. Pupius's 120+ Playwright tests aren't enthusiasm — they're the minimum viable safety net.

> "The fundamentals remain annoyingly fundamental."

On why AI speed doesn't replace refactoring discipline, citing Fowler and Beck. The temptation with agents is to let them loose on the whole codebase at once. Pupius's counter: break work into discrete, reversible phases. Bounded problems with clear success criteria. The speed of AI makes you more dangerous, not less — the discipline has to keep pace.

> "AI models default to 'best practices' when you need 'what actually works.'"

The compromise-as-strategy insight. Agents want to refactor everything into clean abstractions. Legacy systems need surgical changes that preserve invisible context. `window.handleUpload = handleUpload;` is ugly but functional. The skill isn't generating clean code — it's directing the agent toward pragmatic, contained changes that don't disturb things they don't understand.

> "The faster you can fix things, the bolder you can be about breaking them."

The counterintuitive conclusion. Structure that feels like overhead — tests, docs, phased plans — is actually what enables speed. When you know a broken integration will be caught within seconds, you can experiment freely. This inverts the instinct to skip testing to go faster.

## Key Themes

#concept #pattern #tool #person

- **Tests as system boundaries** — Not test coverage for its own sake, but tests as the negative-feedback mechanism that lets agents work safely in codebases they don't fully understand. The 120+ Playwright tests are the article's most concrete deliverable.
- **Documentation as context engineering** — CLAUDE.md is table stakes; brownfield needs architecture overviews, integration maps, and a `/learn` command that turns agent failures into institutional memory. Documentation that agents consume, not just humans.
- **Incrementalism as risk management** — Tooling → structure → migration → integration. Four reversible phases. Bounded problems, clear success criteria. The agent runs fast but the human runs the phases.
- **Compromise hierarchy** — Security and data integrity are non-negotiable. Existing functionality must not break. Ugly patterns are acceptable between steps. This hierarchy is the missing piece in most "vibe coding" disasters.
- **Structure enables speed** — The article's deepest insight: guardrails don't slow you down, they let you go faster. This is the engineering version of "go slow to go fast" applied to agent-assisted development.
- **Agent autonomy as a metric** — Pupius tracked the ratio of agent turns to human prompts (60%–95%). This framing — autonomy as something you measure and improve, not a binary — is more useful than "is my agent autonomous?"

## Critical Analysis

**This is the best brownfield AI piece in the wiki.** Most writing on this topic is either "Copilot helped me rename some variables" (see [[Refactor Legacy Code with Copilot]]'s shallow syntax-modernization advice) or "rewrite everything from scratch" ([[Simplicity in the Age of AI-Assisted]]'s demolition-first approach). Pupius occupies the harder middle: the legacy system isn't going away, we can't rewrite it, and we still need agents to be productive in it. The framework is coherent, tested in real migrations, and honest about limitations.

**The test-writing strategy is novel and under-discussed.** Having Claude inspect HTML/JavaScript to identify structural markers, then write Playwright tests verifying those markers, then using those tests as the safety net for agent-driven refactoring — this is a concrete, repeatable pattern. It's not "write tests first" — it's "use the AI to bootstrap the test suite, then use the test suite to constrain the AI." A bootstrap loop.

**The compromise hierarchy is the most immediately useful takeaway.** Every team struggling with agent-assisted legacy work needs explicit rules about what can be ugly and what can't. Without this hierarchy, agents either refuse pragmatic compromises (defaulting to over-refactoring) or accept everything (producing [[Write Only Code]]). The hierarchy gives you a language for telling the agent "this is fine for now, that is not."

**What's missing:** Pupius mentions CLAUDE.md but doesn't engage with the full context-engineering literature in this wiki — [[Harness Engineering]]'s feedforward/feedback axes, [[Scaling LLMs to Larger Codebases]]'s one-shotting metric, or [[Agent Memory and Context]]'s context-degradation problem. The `/learn` command is clever but hand-waved; there's no implementation detail on how agents analyze their own failures and propose useful doc updates. Also missing: the economic argument. Writing 120+ Playwright tests and architecture docs is expensive. What's the break-even point vs. just doing the migration by hand?

**The "fundamentals remain annoyingly fundamental" line is doing more work than it admits.** Citing Fowler and Beck is correct but insufficient. The real question is which fundamentals change and which don't. Test-driven development was designed for humans writing code line by line — does it apply unmodified to an agent generating entire components? Pupius says yes but doesn't argue it. This is the tension the whole field is wrestling with: which pre-AI disciplines survive and which become ceremony.

**The autonomy metric (60-95%) is the most honest thing in the piece.** Most writers talk about agent autonomy as a yes/no — either the agent does everything (dark factory) or it's just autocomplete. Pupius's range reveals that autonomy is contextual, task-dependent, and improvable. This is more useful than either extreme and deserves its own page in the wiki.

## Cross-Links

- [[Refactor Legacy Code with Copilot]] — The shallow version of this article; Copilot prompt tips vs. Pupius's full engineering framework
- [[Scaling LLMs to Larger Codebases]] — Gill's guidance/oversight split maps onto Pupius's docs/tests split; read together for the complete picture
- [[Building an AI Agent in Rails (Ionescu)]] — Another brownfield success story: bolting an agent onto a 7-year-old Rails monolith with authorization at the tool layer
- [[Guardrails and Feedback Loops]] — The synthesis page for "linters beat prompts"; this article is a case study in building guardrails for legacy code
- [[Harness Engineering]] — Böckeler's feedforward (docs) / feedback (tests) framework is the engineering theory behind Pupius's practice
- [[Feedback Loop is All You Need]] — Tests as the self-tightening feedback loop; Pupius's bootstrap (AI writes tests, tests constrain AI) is a concrete instance
- [[Compound Engineering]] — The `/learn` command is compound engineering: add a system (doc updates from failures) rather than manually documenting
- [[CLAUDE.md (Universal)]] — Referenced as table stakes; Pupius argues brownfield needs deeper documentation than a single context file
- [[Writing a Good CLAUDE.md]] — The instruction-budget case for brevity; Pupius pushes in the opposite direction for brownfield
- [[Make the Easy Change Hard]] — Same Kent Beck refactoring discipline, applied to agent-assisted rather than human-only work
- [[Slowing the Fuck Down]] — Deliberate friction: the phases, tests, and docs are friction that enables speed
- [[Cognitive Debt]] — The risk when autonomy outruns comprehension; Pupius's compromise hierarchy is a partial defense
- [[Simplicity in the Age of AI-Assisted]] — The counterpoint: maybe the right answer is demolition, not brownfield adaptation
- [[The Mythical Agent-Month]] — Agents attack accidental complexity but generate new; Pupius's framework is about managing this dynamic
- [[Agent Coding Workflow]] — Where this article fits on the maturity spectrum: compound engineering territory, well past vibes
- [[Agent Memory and Context]] — Context engineering is the real challenge; the `/learn` command is a memory strategy
- [[Designing Agentic Loops]] — Willison's meta-skill: choosing tools, guardrails, and success criteria for agent work
- [[Write Only Code]] — The endpoint Pupius is trying to avoid: code generated without comprehension or safety nets

---
*Sources: [[summary/a-practical-guide-to-brownfield-ai]]*
*Author: Daniel Pupius*
*Date published: 2026-02-04*
*Last updated: 2026-05-15*
