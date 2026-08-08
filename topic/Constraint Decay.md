# Constraint Decay

The phenomenon where LLM coding agents fall apart as structural constraints accumulate: give an agent a loose spec and it thrives; mandate Clean Architecture, a database backend, and an ORM, and it loses ~30 percentage points of assertion pass rate. The most rigorous empirical study of how production engineering requirements break agentic code generation.

---

## The Core Finding

Dente, Satriani, and Papotti ran 80 backend generation tasks (8 frameworks × 10 constraint combinations) across 7 models and 2 agent scaffolds, totaling ~5 billion tokens. They held the functional spec constant (the RealWorld Conduit API's 19 endpoints) and layered on three non-functional dimensions: architectural pattern, database backend, and ORM.

The result is devastating for anyone hoping to throw Claude Code at a production backend:

> "Among eight capable configurations (L0 A% > 50%), A% dropped by an average of **30 percentage points** from L0 to L3."

That's a 40% relative loss. The worst configuration lost 45 pp — nearly two-thirds of its unconstrained performance. Even the best only dropped 17 pp.

The gap between A% (mean assertions passed) and pass@1 (all assertions passed in at least one run) tells an even starker story: the strongest L3 config hit 78.6% A% but only **8.3% pass@1**. Even at peak performance, agents almost never produce a fully working backend when constraints are present.

## The Data Layer Is the Kill Zone

The paper's most actionable finding is about which constraints actually cause the damage:

| Constraint | Marginal A% Drop |
|---|---|
| PostgreSQL | −19.3 pp |
| SQLite | −14.3 pp |
| Clean Architecture | −9.1 pp |
| SQLAlchemy ORM | −1.5 pp |
| Sequelize ORM | −0.6 pp |

Databases are the killer. Architecture patterns hurt but are survivable. ORMs barely register — and in some cases specifying an ORM *reduces* ambiguity and helps.

The root-cause analysis confirms this: **data-layer defects** (incorrect query logic + DB/ORM runtime errors) drive ~45% of logic failures. Agents can structure code, route HTTP, and follow dependency direction — but they routinely get database queries wrong, misconfigure ORM connections, and botch schema interactions.

This tracks with practical experience. An agent writing a REST endpoint understands the pattern. An agent writing a parameterized SQL query with correct join logic, transaction handling, and type coercion is doing something fundamentally harder — and current models aren't there yet.

## Convention-Heavy Frameworks Are a Trap

> "Express (51.4%), Koa (50.7%), Flask (49.3%) — minimal, explicit API surfaces with no implicit conventions. Bottom tier: Django (25.4%), FastAPI (24.2%), Hono (18.5%)."

This inverts conventional wisdom. Django and FastAPI are designed to make *humans* productive by providing batteries, conventions, and magic. For agents, that magic becomes a minefield. The agent doesn't know which Django middleware is active, doesn't understand FastAPI's dependency injection, can't trace Hono's edge-runtime compatibility layer.

Explicit frameworks (Express, Flask) win not because they're better frameworks — but because their API surface is flat and visible. There's nothing to infer. Every route, every middleware, every database call is right there in the code.

The implication: if you're building with agents, choose frameworks that minimize implicit behavior. Convention-over-configuration was optimizing for the wrong thing.

## The Scaffold Matters, But Not Enough

The paper tested Mini-SWE-Agent (~100 lines of bash) and OpenHands (full-featured ReAct framework). OpenHands + MiniMax-M2.5 was the most resilient combination. But the difference between scaffolds was small relative to the constraint effect — suggesting this isn't a scaffold problem that better tooling will fix. It's a fundamental capability gap.

## A Potential Escape Hatch: Deterministic Generators

[[BESSER]], an academically-led low-code platform, suggests an alternative architecture: instead of asking an agent to follow constraints (Clean Architecture, Postgres, ORM) from a natural-language prompt, encode the constraints in a formal model and use deterministic generators to produce the constrained code. The agent assists with *modeling* (image-to-model, text-to-model) but the model→code translation is a compiler, not an LLM call. If the model is valid, the generated code is constraint-correct by construction — there's nothing for the agent to get wrong. Whether this works at the scale and diversity of real applications is unproven, but it reframes the constraint decay problem from "make agents better at following rules" to "don't make agents follow rules at all."

This aligns with [[Honey I Shrunk the Coding Agent]]'s finding that scaffold redesign can yield dramatic improvements — but also with the broader lesson that scaffolds have limits. You can't scaffold your way out of a model that doesn't understand database transactions.

## Critical Assessment

**This paper is important because it quantifies what the industry has been feeling.** "AI writes good code" isn't wrong — it's incomplete. AI writes good code under conditions that don't resemble production engineering. Add the constraints that make software maintainable, and performance collapses. A parallel dynamic plays out in tabular prediction: [[Why LLMs Fail at Tabular Prediction]] shows that LLM in-context classification works fine in 2D but collapses as feature columns increase — the capability is real but bounded by a specific structural condition (dimensionality there, constraints here). Both papers share the rare methodological virtue of isolating *which* condition causes the collapse rather than just reporting that collapse occurs.

**The database finding is the one to act on.** If data-layer defects cause 45% of failures, the highest-leverage intervention isn't better prompts or better scaffolds — it's better database tooling. Give agents type-safe query builders instead of raw SQL. Auto-generate migration files. Let agents write queries in their native language and compile to SQL deterministically. The [[Layer-First Pattern — Keep Data Out of the LLM Context]] suggests one architectural answer: keep data operations server-side where they can be verified.

**The framework finding is actionable today.** If you're building with agents, pick Express over Django, Flask over FastAPI. Explicit over implicit. Flat over nested. The framework that's easiest for a human to learn is not necessarily the framework that's easiest for an agent to operate.

**The pass@1 gap is the real problem.** 78.6% A% sounds fine until you realize that means the *average* endpoint works, but the *system* doesn't. Cross-file consistency — getting imports right, matching type signatures across modules, keeping the ORM models aligned with the database schema — is where agents fail. This isn't a reasoning problem; it's a coordination problem. Agents don't maintain a global model of the codebase they're writing.

**One thing the paper doesn't address:** what happens when you give agents a reference implementation of one endpoint and ask them to follow the pattern? The benchmark uses OpenAPI specs alone, but real development always has existing code to learn from. The paper's feature-implementation sanity check (ablating features from existing repos) showed only GPT-5.2 exceeded 50% pass@1 — suggesting pattern-following helps but doesn't solve the problem.

**The paper is also, inadvertently, a case for spec-driven development.** If agents can be made reliable at the unconstrained level, and constraints cause the decay, the answer might be [[Specifications as the Product]]: keep the spec as the durable artifact, let agents generate the unconstrained code, and verify the constraints deterministically through linters, type checkers, and schema validators rather than relying on the agent to respect them.

---

## See Also

- [[Agent Coding Workflow]] — the practitioner's loop where this constraint decay plays out daily
- [[Honey I Shrunk the Coding Agent]] — scaffold redesign changes everything; same fundamental insight about model-scaffold interaction
- [[Components of a Coding Agent]] — "the harness matters more than the model"
- [[Agentic Testing]] — Slack's empirical study of agent-driven testing; similar rigor, similar sobering results
- [[Guardrails and Feedback Loops]] — deterministic enforcement over prompt-level pleading
- [[Specifications as the Product]] — if constraints cause decay, the spec is the durable artifact
- [[Layer-First Pattern — Keep Data Out of the LLM Context]] — architectural response to data-layer failures
- [[FrontierCode]] — Cognition's mergeability benchmark; another sobering look at agent code quality
- [[A New Era for Software Testing]] — antirez on agentic QA as compensation for lower-quality AI code
- [[Agentic Software Engineering (Hassan)]] — the comprehensive treatment of engineering with stochastic contributors
- [[Software Engineering at the Tipping Point]] — Bender on AI as 10× amplifier, not directed solution
- [[Lean Software Scaling Laws]] — Gwern predicts the constraint-decay slope should be *shallower* in languages where invariants are structural (Lean's type system) rather than bolted-on (Python + mypy); directly testable with this paper's methodology
- [[React Component Purity]] — not all constraints degrade performance. React's purity rules (idempotent renders, no side effects, immutable props/state) *reduce* the solution space rather than expanding it, eliminating entire categories of wrong implementations. The paper's framework finding (explicit > implicit for agents) and React's purity model converge on the same principle: make the rules visible and structural, not ambient and inferential

---
*Sources: [[summary/constraint-decay]]*
*Last updated: 2026-08-08*
