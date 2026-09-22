# Software Factories, Light and Dark

Addy Osmani's O'Reilly Radar essay (republished from his blog) turns Dex Horthy's factory diagnosis into a general theory: software is a stack of loop → harness → factory, where a dark factory ships code no human has read and a lit factory moves judgment upstream. Its core rules — verification is the bottleneck, back pressure caps autonomy at what can be cheaply verified, and each loop individually earns its darkness — make it the clearest map of *where* human judgment physically lives in an agentic pipeline.

---

## Key Quotes

> "A software factory is many harnessed loops running at once, fed by a queue of work and drained through a review gate into production, with humans owning the whole thing from above. It isn't a bigger agent; it's an org chart made of loops."

The definitional move that makes the rest of the essay work: scale is organizational, not model-size. A factory is not a smarter loop — it's the same loop, queued and gated, which is why the failure modes turn out to be management problems (queue quality, gate placement) rather than capability problems.

> "By and large, every box in this diagram is almost zero cost... There's only one expensive box that proves stubbornly resistant to scaling, and that's the review gate. That shiny amber box is 'judgment.'"

The sharpest economic statement of the review bottleneck in this wiki. Everything else in the pipeline has already collapsed to negligible cost; the entire argument about whether software gets faster lives in one amber box. This is the framing [[The End of Code Review]] and [[Orchestrating AI Code Review at Scale]] are both attempts to attack from opposite directions — abolish the box, or automate it.

> "Comprehension debt is the widening gap between how much code exists and how much any human still understands. A dark factory doesn't pay it down; it takes it on as fast as it can, with the tests green the whole way."

Osmani adopting [[Agents and Acquiring Debt]]'s coinage and giving it a factory-floor mechanism. The killer detail is that the debt accrues *while every dashboard stays green* — the tests cannot detect a comprehension shortfall because comprehension isn't what they measure.

> "Back pressure is the rule that you can only hand a loop as much autonomy as you can cheaply and reliably verify, and not one inch more."

The term is imported from queueing systems — Fred Hebert's [[Queues Don't Fix Overload]] is the canonical statement — and it's exactly the right borrowing: generation capacity is now effectively infinite, so the finite resource (human attention) sets the flow rate. "What we're really suffering from is a surplus of bad PRs" is Horthy's line, and it converts the volume panic into a quality diagnosis.

> "The safety net is made up of perfectly ordinary architectural practices we've always known about and mostly ignored... now that we're using automated coding agents, that architecture is finally doing a second job as a cheap and hard-to-fake safety net against the mistakes the agent will make."

The quiet radical claim of the essay: architecture stops being taste and becomes verification infrastructure. Types, test seams, short call stacks, small blast radii, dependency injection — recast as oracles that are cheap to run and hard for a model to fake. This converges with [[The Economic Benefit of Refactoring]]'s finding that good structure cuts agent token costs by 83%: well-architected code now pays dividends in machine costs, not just human comprehension.

> "The genuine new move was trying to throw the diagram away, leaning on a loop where the model picks the path tool call by tool call... The discipline everyone is now rediscovering, owning your control flow, is really just walking the graph back around the loop."

The flowchart rehabilitation. Quoting Horthy's blunt line that most "agents" are "mostly deterministic code, with LLM steps sprinkled in at just the right points," Osmani argues the free-running loop was a two-year detour and the state machine was always the destination — echoing [[Dmitry Sotnikov's LLM Workflow]]'s behavior trees and David Khourshid's actor-model reminder.

> "Robots are fine operating in the dark, but humans need to see what they're doing. If everything on the factory floor is dark, and you can't see anything, and you can't even find the light switch, that's where the danger is."

The closing image, and the essay's actual position: not anti-darkness, but darkness as a budgeted, per-loop privilege rather than a default state.

## Key Themes

- #concept **Comprehension debt as the dark factory's ledger** — the four-month HumanLayer run (no human read any of the code) as the field evidence; the reckoning is "quiet and late," not a dramatic collapse.
- #pattern **Back pressure** — autonomy expands exactly as far as cheap, reliable, unfakeable verification carries it; the review gate is the only non-scaling box.
- #pattern **The per-loop light switch** — darkness is earned per loop (cheap, high-frequency, drift-free oracles: type gates, property tests, rubric-coupled review agents), never granted globally; agents hold 3–10 steps and fray past twenty.
- #pattern **Architecture's second job** — types, seams, and boundaries as the model-independent safety net; judgment moved upstream to plan review (a 200-line plan beats a 2,000-line diff).
- #person Addy Osmani (third Radar essay in this wiki) and Dex Horthy (HumanLayer, the underlying talk).

## Analysis

Provenance first: this is the most derivative of Osmani's three Radar essays in this wiki, and it says so — the piece is an annotated remix of Horthy's talk, carrying his four-month anecdote, his 3–10-steps heuristic, and his bad-PRs diagnosis largely intact. What Osmani adds is connective tissue and genealogy: the Bemer 1968 lineage (the factory dream failing for half a century because "stamping out ideas" was hard), the manufacturing etymology (FANUC since 2001, Xiaomi 2024 — "dark" is a physical claim, not a vibe), and the fusion of Horthy's training-objective argument with the debt literature. Read as a bridge document, it's excellent; read as original reporting, it's thin.

The strongest contribution is the switch-grid model. Where [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] collapses automation onto one axis per team, Osmani makes autonomy a property of each individual loop: the nightly lint cron runs dark because its oracle is cheap and unfakeable; the billing engine stays lit because its failure is expensive and only a person catches it. This dissolves the sterile dark-vs-skeptical debate — [[Don't Fear the Dark Factory]] and [[Harness Engineering is not Enough]] turn out to be arguing about different loops.

The honest weaknesses: the title oversells the body (the essay recommends almost nothing run fully dark — "lights mostly on, carefully rationed darkness" is the actual position); the load-bearing numbers are Horthy's, relayed without independent verification, so treat this as a strong secondary source pointing at a primary talk; and the four-month anecdote — the empirical spine — arrives with no postmortem detail at all. The graphs section is the weakest quarter: the flowchart-rediscovery argument is correct but well-trodden, and its best line is a quote of someone else.

Verdict: keep it for the vocabulary — loop/harness/factory, the amber review gate, back pressure, per-loop switches — and as the middle volume of Osmani's accidental trilogy: [[The New Software Lifecycle]] argued harness over model, this maps where the factory runs and which lights stay on, and [[Own the Outer Loop]] names who answers for what ships.

## Relationships

- [[Harness Engineering is not Enough]] — this essay is Osmani's public tour of Horthy's talk, and it strengthens that page by generalizing the diagnosis into factory vocabulary (queue → harness → review gate → production) while softening the talk's absolutist title into a tractable rule: not "harness engineering is not enough" but "harness engineering is enough exactly up to what verification can carry."
- [[Own the Outer Loop]] — same author, same venue, next paragraph of the same argument: this essay supplies the floor plan (judgment moved upstream to plan review plus one amber review gate) that the accountability essay formalizes into Quality/Verdict/Answerability; together they are the "where" and the "who" of keeping the lights on.
- [[Agents and Acquiring Debt]] — adopts bl00cyb's comprehension-debt coinage wholesale and gives it its best field evidence yet (the four-month no-human-reads run, tests green the whole way), while complicating the fix: where the debt essay reaches for ADRs, Osmani reaches for upstream planning and architectural seams as the paydown mechanism.
- [[Don't Fear the Dark Factory]] — Wynne's conversion narrative is the practitioner counterweight, and the triangulation is productive: his yaks loop survives darkness precisely because its oracle (a validation harness requiring *all* recommendations addressed) meets Osmani's cheap/high-frequency/unfakeable bar — darkness is earned per loop by oracle quality, not granted per team by ideology.

---
*Sources: [[raw/software-factories-light-and-dark]], [[summary/software-factories-light-and-dark]]*
*Last updated: 2026-09-22*
