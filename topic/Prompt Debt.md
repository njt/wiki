# Prompt Debt

Drew Breunig names a specific form of technical debt that's unique to AI-native systems: the natural language prompt degrades from asset to liability as edge cases, model-specific fixes, and repeated instructions accumulate. The diagnosis is sharp because it explains several familiar pain points — brittle prompts, model lock-in, illegible system instructions — as one structural problem rather than separate nuisances.

---

## The Trap

Breunig describes a three-stage progression:

**Stage 1 — Slowing iteration.** Users flag errors, edge cases surface, and "hot fixes" get added to the prompt. Instructions are repeated with increasing severity. Soon one-line fixes regress earlier instructions, and the development cycle slows to a crawl.

**Stage 2 — Incapacitated team.** The prompt becomes illegible — a thicket of edge cases, all-caps threats, and repeated instructions that only the original author can navigate. Teams mitigate by breaking prompts into templated segments, but each segment evolves independently into its own thicket.

**Stage 3 — Model lock-in.** Fixes tuned for GPT-4o fail in new ways on GPT-5.4-mini. The team stays on an aging model, ignores deprecation emails, and forgoes cheaper/faster/better alternatives. Breunig cites Datadog data showing GPT-4o remains the most-used model in observed traffic — a signal that prompt debt is keeping enterprises pinned to old models.

> "Any one of these issues is a nuisance, but together they are the difference between a glorified prototype and a product that can grow with you, your customers, and your business."

## Why It Happens

Two structural forces combine to create prompt debt:

**Imprecision × probability.** Natural language is inherently imprecise; probabilistic models amplify that imprecision. The same intent expressed in different words yields different outputs. Breunig cites a study where the same clinical question, rephrased in a physician's voice vs. a patient's voice, flipped Opus from declining all ten times to answering all ten.

**Spurious correlations.** Seemingly unrelated statements in the same prompt affect results. In a Harvard study, merely stating which NFL team the user rooted for changed how often the model refused to answer sensitive questions. This means an additional instruction to fix one error can silently affect how the model interprets a separate instruction that worked yesterday.

**Fighting the weights.** When desired behavior is at odds with a model's training, prompt authors resort to repetition. Breunig documents this across production systems:

> "Every coding agent system prompt we analyzed featured repeated instructions, stern warnings, and all-caps demands. Claude Code tells Opus *seven times* to return multiple tool calls in a single response. And even the most advanced models force prompt authors to fight the weights: Fable's leaked system prompt restates one specific copyright rule six times."

This is the most vivid section of the piece. Reading it, you realize that every serious system prompt is a palimpsest of battles fought against the model's training distribution — and every new battle makes the prompt more brittle.

## The Prevention Strategy

Breunig draws on coding-agent best practices — programmers using AI sit at the leading edge of model capability, and they've evolved practices that deliver maintainable, modular software:

### 1. Measurement, not prose

> "Specify your system's behavior with measurements, not prose. When the model's output is probabilistic and language is imprecise, we build hard edges to constrain them: evaluations, metrics, and typed specifications."

This echoes [[Guardrails and Feedback Loops]]'s "linters beat prompts" thesis, but Breunig extends the argument from code quality to the entire specification surface. It's not just that lint rules catch agent mistakes — it's that the spec itself should be a measurable artifact rather than a prose document.

> "The best engineers now spend more of their bandwidth on tests than ever, as they are no longer a safety net but the thing that *lets the model cook*."

### 2. Search, don't write

Once you have metrics that can score prompt candidates, the prompt becomes something to search for rather than something to craft. The surface area of possible words, phrases, and structures in natural language is too vast for human search:

> "This is terrain LLMs were built to explore, and there are already systems (like DSPy and GEPA) that manage this work for you, holding prompts accountable to your designs."

### The payoff: model portability

When behavior is defined by measurements and prompts are generated rather than hand-tuned, model lock-in dissolves. Evaluating a new model takes hours instead of weeks. Whether a model is pulled for regulatory reasons (Fable) or deprecated due to age (Llama-3.1-8b), the fix becomes a chore rather than a fire drill.

## Key Quotes

> "The plain-English prompt that makes prototypes effortless turns out to be a poor way to specify how a system should behave, and the bill arrives slowly, disguised as ordinary progress, until the application can barely move."

This is the central metaphor — prompt debt is insidious because it doesn't announce itself. Every fix feels like progress; in aggregate they're strangling the system.

> "Our inability to easily swap models isn't the result of frontier labs coming up with a clever moat. No, it's the result of evolving a lossy natural language specification against a probabilistic model."

A genuinely useful reframe. The model lock-in we observe isn't strategic — it's an emergent property of building on sand.

> "Every mature engineering discipline eventually stops doing by hand the very thing it once prided itself on doing by hand. Assembly gave way to compilers, hand-tuned queries gave way to planners, and manual memory management gave way (mostly) to machines that do it better. Prompt-writing is no different."

The historical pattern that closes the argument. Compelling, though it's worth noting that each of these transitions took a decade or more to complete, and all three still have practitioners who swear by the manual approach for performance-critical work.

## Critical Analysis

**The diagnosis is stronger than the prescription.** Breunig's taxonomy of prompt debt stages and his explanation of *why* it happens (imprecision × probability, spurious correlations, fighting the weights) are precise and falsifiable. The prescription — metrics, evals, auto-generated prompts — is directionally correct but underspecified. "Specify behavior with measurements" is the right principle, but for many tasks the measurement *is* the hard problem. What's the eval for "write a good email"? What's the metric for "explain this concept clearly"? The prescription works cleanly for structured-output tasks and gets fuzzy fast for open-ended generation.

**The "fighting the weights" evidence is the piece's strongest contribution.** The documentation of repetition across production system prompts — ChatGPT (8×), Claude Code (7×), Fable (6×) — is concrete evidence that even the best-resourced teams at frontier labs are trapped by the same dynamic. This isn't a problem of skill or budget; it's structural.

**The DSPy/GEPA recommendation is underdeveloped.** Breunig mentions them in passing as evidence that auto-generated prompts exist, but doesn't engage with their limitations — they work well for classification and extraction tasks but struggle with the kind of open-ended system prompts that characterize most agentic applications. The gap between "DSPy can optimize a classification prompt" and "DSPy can replace Claude Code's system prompt" is vast. [[DSPy — Programming Not Prompting]] is the concrete framework: typed signatures compile to optimized prompts, with a three-layer architecture (signatures/modules/optimizers) that cleanly separates task definition from execution strategy from prompt tuning — the architectural property hand-written prompts structurally prevent.

**DSPy Flex closes part of this gap.** Flex extends GEPA's optimization surface from prompts alone to prompts *and code* — the optimizer can rewrite the module's Python source, author helper functions, implement routing logic, and decompose signatures alongside rewriting instructions. On a geospatial conflation task, Flex + GEPA produced a program that was more accurate (95.0% vs. 90.4%), 28% cheaper, and 40% faster than the baseline — by routing 75% of records through deterministic Python and reserving the model for genuinely ambiguous cases. The generated code follows a three-stage normalize-compare-decide pattern, with the LLM positioned as a last-resort fallback. Breunig's "search, don't write" prescription now extends to code, not just prompts. See [[DSPy Flex — Let the Model Write the Code]].

**The historical analogy is rhetorically powerful but mechanically imperfect.** Compilers, query planners, and garbage collectors succeeded because they operated on formal, well-specified inputs. Prompt search operates on natural language — the same imprecise medium it's trying to escape. The analogy works as aspiration but not as prediction.

**What's missing: the intermediate path.** Between "hand-craft every prompt" and "full metric-driven prompt search" there's a large, practical middle ground that Breunig skips over: writing prompts in structured formats (YAML, DSLs), decomposing monoliths into composable modules, versioning prompts, and A/B testing them. Many teams could reduce prompt debt significantly without adopting DSPy.

## Connections to the Wiki

The prompt debt diagnosis strengthens several existing threads:

- **[[Specifications as the Product]]** — Breunig adds the measurement dimension: the spec isn't just the durable artifact; it's the *measurable* artifact that constrains probabilistic output. Evals and metrics are the hard edges that prose specs lack.
- **[[Guardrails and Feedback Loops]]** — "Linters beat prompts" gets a deeper explanation here: prompts fail not because they're poorly written but because natural language is structurally unfit as a specification medium for probabilistic systems.
- **[[The Oracle Is the Asset]]** — Breunig extends Ruby's oracle concept from test suites to the entire specification surface, including evals, metrics, and typed specs. The oracle isn't just the test suite; it's everything measurable you assert about system behavior.
- **[[Smart Models Dumb Pipes]]** — The prompt sits in an awkward middle layer between judgment (model) and execution (infrastructure), corrupting both. Breunig's prescription moves the specification out of the prompt and into the measurement layer, restoring the clean separation McCormick argues for.
- **[[Loop Engineering]]** — Addy Osmani's meta-skill of "designing systems that prompt agents" is the practical implementation of Breunig's "stop writing prompts by hand." The loop engineer builds the measurement infrastructure and prompt-search machinery; individual prompts become output artifacts, not input craft.
- **[[Constraint Decay]]** — Empirical confirmation of Breunig's thesis: LLM coding agents lose ~30pp assertion pass rate when structural constraints are imposed, precisely because prompts can't reliably enforce invariants against a model's training weights.
- **[[Eval-Driven Development (Airbnb)]]** — Airbnb's production playbook is the concrete implementation of "measurement, not prose": EDD replaces hand-tuned system prompts with a calibration loop (golden dataset → judge agreement → rubric refinement) that turns natural-language rubrics into measurement instruments. When behavior is defined by evals and calibrated judges rather than prose instructions, model lock-in dissolves — evaluating a new model takes hours, not weeks, exactly as Breunig prescribes.

---

*Sources: [[raw/the-problem-is-prompt-debt]], [[summary/the-problem-is-prompt-debt]]*
*Last updated: 2026-08-07*
