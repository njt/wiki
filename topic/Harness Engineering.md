# Harness Engineering

Birgitta Böckeler's framework for systematically building confidence in AI-generated code, published on Martin Fowler's site. The core equation is **Agent = Model + Harness**, where the harness is everything except the model itself. She decomposes it into two axes: feedforward vs. feedback (guides that steer before the agent acts, sensors that correct after), and computational vs. inferential (deterministic tools vs. AI-based review). The result is a 2x2 taxonomy that finally gives the "linters beat prompts" intuition a proper engineering vocabulary. #concept #person

---

## Key Quotes

> "A well-built outer harness serves two goals: it increases the probability that the agent gets it right in the first place, and it provides a feedback loop that self-corrects."

This is the thesis. Two goals, not one. Most teams fixate on feedback (catch mistakes after they happen) and neglect feedforward (prevent them from happening). Böckeler insists you need both or you get pathological loops: feedback-only agents repeat mistakes endlessly; feedforward-only agents encode rules they never verify.

> "A coding agent has none of this: no social accountability, no aesthetic disgust at a 300-line function."

The sharpest line in the piece. Human developers are regulated by social mechanisms — code review shame, team norms, career incentives — that agents simply lack. The harness must replace what socialization provides for humans. This reframes the problem: you're not just adding tooling, you're replacing an entire social control system with an engineering one.

> "A good harness should not necessarily aim to fully eliminate human input, but to direct it to where our input is most important."

Against the "fully autonomous agent" fantasy. The harness isn't about removing humans — it's about routing human attention to the decisions that actually require judgment. This is the mature position, and it's the opposite of the "dark factory" vision where humans disappear entirely.

> "Separately, you get either an agent that keeps repeating the same mistakes (feedback-only) or an agent that encodes rules but never finds out whether they worked (feed-forward-only)."

The pathology diagnosis. This explains why teams that only add lint rules (feedforward) or only add CI checks (feedback) plateau. You need the closed loop.

> "An LLM-based coding agent can produce almost anything, but committing to a topology narrows that space, making a comprehensive harness more achievable."

Architecture constrains the agent's output space, which makes harnesses tractable. This is the argument for strong opinions in your codebase: the more constrained the design, the fewer failure modes the harness needs to cover.

## Key Themes

### The Two Axes

**Feedforward vs. Feedback** maps to prevention vs. detection. Guides (AGENTS.md files, skills, bootstrap scripts, OpenRewrite recipes) steer before generation. Sensors (tests, linters, ArchUnit hooks, AI code review) catch problems after. The insight is that you need both — and that most teams are lopsided toward feedback. #concept

**Computational vs. Inferential** maps to deterministic vs. probabilistic. Computational controls (linters, type checkers, structural tests) are cheap, fast, and reliable. Inferential controls (LLM-as-judge, AI code review) are expensive, slow, and non-deterministic — but they catch semantic issues that no linter can. The recommendation: computational sensors on every change, inferential sensors reserved for post-integration. #concept

### Three Regulation Categories

1. **Maintainability Harness** — the most mature. Linters, type checkers, tests. This is where most teams already have infrastructure. #pattern
2. **Architecture Fitness Harness** — enforces structural properties via fitness functions. ArchUnit, dependency rules, topology constraints. Underinvested by most teams. #pattern
3. **Behaviour Harness** — functional correctness. The hardest and least solved. AI-generated tests are not trustworthy enough as the sole mechanism. Böckeler is honest that "we still have a lot to do" here. #pattern

### Harnessability and Ambient Affordances

Not all codebases are equally harness-friendly. Strongly typed languages, clear module boundaries, and opinionated frameworks create what Ned Letcher calls "ambient affordances" — structural properties that make the environment legible and tractable to agents. Greenfield projects can embed harnessability from inception; legacy systems face steeper challenges. This concept connects directly to [[Components of a Coding Agent]]'s finding that "context quality > model quality" — harnessability IS context quality, from the infrastructure side. #concept

### The Steering Loop

Humans improve the harness iteratively: observe agent failures, add controls, test whether they work. The agent itself can help — generating custom lint rules, structural tests, documentation. This is [[Compound Engineering]]'s compounding loop expressed as a control theory concept, and it maps to [[Feedback Loop is All You Need]]'s self-tightening loop (Agent -> Rules -> CI -> Observability -> Tasks -> Agent). [[DSPy — Programming Not Prompting]] applies the same steering loop to a narrower surface: its optimizers automate the observe→tune→test cycle for prompt engineering, treating the prompt as a search space rather than a craft artifact — a feedforward computational control that removes the human from the prompt-tuning loop entirely.

### Ashby's Law

The sidebar references the law of requisite variety: a controller must have at least as much variety as the system it regulates. For coding agents, this means the harness must be as complex as the code the agent can produce — which is a sobering constraint, because agents can produce nearly anything.

## Critical Analysis

This is the most intellectually rigorous piece on agent quality control I've seen. Where [[Feedback Loop is All You Need]] gave the rallying cry ("your linter isn't a suggestion"), Böckeler gives the engineering framework. The feedforward/feedback and computational/inferential axes aren't just taxonomic neatness — they expose real gaps in how teams think about agent governance. Most teams have computational feedback (linters, tests) and inferential feedforward (CLAUDE.md instructions), but almost nobody has computational feedforward (code generation templates, OpenRewrite recipes) or systematic inferential feedback (LLM-as-judge in CI).

The article's biggest strength is its honesty about what's unsolved. The behaviour harness — proving that agent code does what it should — remains wide open. AI-generated tests are circular (the same model that wrote the code writes the tests), and human-written test suites can't scale to agent velocity. This is the actual hard problem, and Böckeler doesn't pretend she's solved it.

The weaknesses are minor but real. The control theory metaphors (feedforward, feedback, Ashby's Law) give the piece intellectual heft but occasionally obscure rather than illuminate — "sensor" and "guide" would work fine without the cybernetics framing. The real-world examples (OpenAI, Stripe, Thoughtworks) are mentioned but not deeply explored; you don't get enough detail to reproduce what they did.

The connection to [[Compound Engineering]] is tight: Klaassen's 50/50 rule (half your time on system improvement) is exactly how you fund the steering loop Böckeler describes. The connection to [[Guardrails and Feedback Loops]] is that Böckeler provides the theory that the synthesis page describes in practice. And [[Pre-Commit Lint Checks]] is the most concrete existing implementation of the maintainability harness.

The most comprehensive open-source harness implementation is [[Oh My Pi (omp)]], which instantiates nearly every quadrant of Böckeler's 2×2: computational feedforward (hashline content-hash editing to prevent patch corruption, AST structural rewrites), inferential feedforward (system prompts tuned per model, time-traveling stream rules that inject guardrails mid-turn), computational feedback (in-process lint/format/test execution), and inferential feedback (the advisor/watchdog pattern — a second model reading every turn and injecting notes).

[[Morningprint]] is a compact case study in dual-environment harness design: a Node.js `test-print.mjs` script loads the exact production bundle (`dist/main.gs`) via `vm.createContext`, so `art` mode runs the identical renderer that deploys to Apps Script. `art:live` mirrors the full Anthropic client (same `buildArtRequestBody`, same `pause_turn` resumption, same web-search-tool fallback). The `--dry` flag previews exact ESC/POS bytes before printing — computational feedback at the byte level. The one deliberate divergence (the `server-side-fallback` beta header in local testing but not production) is feedforward design: production refusal fails loudly to alert email; local testing degrades gracefully.

Where this article goes beyond existing wiki coverage is the **feedforward** dimension. The wiki's quality pages are heavily skewed toward feedback (linters, CI, evals). Böckeler's argument that feedforward controls — guides, templates, bootstrap scripts, code mods — are equally important is a genuine gap in the current conversation. The idea that you should shape the agent's output space before it acts, not just catch problems after, is the piece's most original contribution.

The "harnessability" concept deserves its own future page. The idea that your choice of language, framework, and architecture determines how effectively you can govern agents is a design-time decision with massive downstream consequences — and it's barely discussed anywhere else.

**The research frontier**: Böckeler's framework describes what practitioners do *today*. Lilian Weng's [[Harness Engineering for Self-Improvement]] surveys where this is headed: harness design itself becoming an optimization target, with systems like Darwin Gödel Machine and Agentic Harness Engineering evolving harness code through search rather than handcrafting it. The same feedforward/feedback taxonomy Böckeler uses maps naturally onto the research literature — ACE and MCE are feedforward context engineering; the propose-evaluate-accept loop in Self-Harness is feedback-driven harness evolution. The meta-question Weng raises is whether the steering loop Böckeler describes (human observes failures → adds controls → tests) can itself be automated.

## Cross-Links

- [[Guardrails and Feedback Loops]] — the synthesis page this enriches with proper theory
- [[Feedback Loop is All You Need]] — the self-tightening loop as one instantiation of the steering loop
- [[Compound Engineering]] — the 50/50 rule funds the steering loop
- [[Components of a Coding Agent]] — Raschka's "harness matters more than model" is the same thesis
- [[Pre-Commit Lint Checks]] — concrete implementation of the maintainability harness
- [[claude-ctrl]] — computational feedforward + feedback in one tool
- [[Demystifying Evals for AI Agents]] — eval frameworks as part of the behaviour harness
- [[dotnet Slopwatch]] — agent-specific sensors for the maintainability harness
- [[Fresh Eyes]] — inferential feedback via cross-model review
- [[Write Only Code]] — what happens when the harness is absent
- [[The Lifecycle of LLM-as-a-Judge]] — inferential feedback running in production at Netflix: an LLM judge serving as both quality gate and self-reflection critic, with Phase IV drift monitoring (a ±2σ band below the average human rater) as the steering loop that re-tunes the sensor itself

---
*Sources: [[summary/harness-engineering-for-coding-agent-users]]*
*Last updated: 2026-05-14*
