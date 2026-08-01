# Engineering for Bounded Cognition

Your working memory holds about four things. Your attention is a torch beam, not a floodlight. Unrehearsed information decays in twenty seconds. This is the instrument we build software with — and the gap between its limits and our million-line systems is the foundational problem of engineering. Matt Williams argues that the real question isn't how to make people smarter or models bigger, but how to shape systems so a small mind can work on them without bringing everything down.

---

## Key Quotes

> "That's the instrument we build software with."

Williams drops this after walking through the gorilla experiment (half of observers missed a chest-beating gorilla) and the door study (people didn't notice their conversation partner was swapped for a different person). The deadpan delivery is the point: this isn't a bug in human cognition, it's the spec. Every engineering decision either accommodates this reality or pretends it isn't true.

> "A warning that roughly half of attentive people will look straight at and not see isn't really a warning. It's a decoration."

The sharpest line in the essay. It reframes incident postmortems that blame "human error" as system design failures. If your warning has a 50% miss rate under ideal conditions, the warning is the problem, not the operator. This is the [[Guardrails and Feedback Loops]] thesis applied to attention itself: deterministic enforcement beats pleading.

> "how do we shape the thing so that a small mind can work on it without bringing it all down"

This is the essay's central question, and Williams is careful to say it can't be answered by hiring smarter people or buying bigger context windows. The gap is permanent. Engineering is the set of practices we've evolved to live inside that gap.

> "Designing for the most constrained user isn't some charity that the rest of us put up with."

The OXO Good Grips parable — a vegetable peeler designed for arthritic hands that became a mass-market bestseller. The same principle applies to software: systems designed for the tired, distracted, or novice engineer end up being the systems *everyone* reaches for when their attention narrows. This inverts the usual "junior-proofing" framing. Designing for the weakest moment isn't dumbing down; it's designing for reality.

## The AI Connection

Williams extends the argument to LLMs by comparing context windows to working memory. The "Lost in the Middle" finding (Liu et al., 2023) shows that models, like humans, lose information buried in the middle of long inputs. More retrieved documents can pull accuracy *below* the no-documents baseline because attention is a fixed quantity. The model "loses the thread" exactly the way a tired person does.

This is the same insight driving [[Maybe Coding Agents Don't Need a Bigger Memory]] and [[Coding Agents Continuity Not Memory]]: bigger context windows don't solve the fundamental attention problem. They just give you a bigger warehouse for your torch beam to get lost in.

## Four Prosthetic Cognition Moves

Williams identifies four practices that move information from fragile biological memory into stable external structure:

1. **Naming** — a precise name means you no longer have to hold that fact in your head. This is why [[99 Bottles of OOP]] treats naming as the central act of design.
2. **Boundaries** — drawing a boundary creates a promise you can stop re-checking. The entire [[Agent Orchestration]] and microservices literature is an exercise in boundary-drawing as cognitive offload.
3. **Tests** — writing a test parks a decision somewhere it cannot fade. This is the [[Guardrails and Feedback Loops]] argument: linters beat prompts, tests beat memory.
4. **Undo** — anything that is undoable grants permission to be wrong. This is the argument for worktrees, feature flags, and rollback: they're not just safety mechanisms, they're *cognitive prosthetics* that let you operate near the edge of your competence.

These aren't framed as "best practices." They're framed as what you must do because your brain physically cannot hold the system otherwise.

## Key Themes

- #concept **Bounded cognition** — working memory as the binding constraint on software engineering
- #concept **Prosthetic cognition** — naming, boundaries, tests, and undo as external memory, not just methodology
- #concept **Attention as fixed quantity** — applies equally to humans (four chunks) and LLMs (lost in the middle)
- #pattern **Design for the constrained case** — OXO Good Grips principle applied to engineering systems: design for the tired, distracted operator and everyone benefits
- #person **Matt Williams** — author of "The Shape of the System" manifesto, of which this essay is the philosophical foundation

## Critical Analysis

**The strongest contribution is the human-AI symmetry argument.** Williams doesn't just say "LLMs have context window limits" — he maps them onto the same cognitive architecture that constrains humans. Both lose information in the middle of long sequences. Both have attention as a fixed, depletable resource. This is a genuinely useful frame: it means every technique we've developed for managing human cognitive limits (chunking, external memory, progressive disclosure) should have an analogue for managing LLM context limits. And vice versa — if we discover something that helps LLMs stay oriented in long contexts, it might tell us something about human cognition too.

**The essay understates the tension in its own argument.** Designing for the most constrained user is the OXO principle. But the *least* constrained users — the focused, expert engineers working at peak attention — may find those same guardrails infantilizing. There's a real tradeoff between "safe for four-slot mode" and "efficient for focused mode" that Williams doesn't address. The best systems probably need to degrade gracefully: guardrails that are present but not obstructive when you're operating at full attention, and load-bearing when you're not.

**The four "prosthetic cognition" moves are correct but not new.** Naming, boundaries, tests, and undo have been standard advice for decades. What *is* new is the framing: these aren't professional virtues, they're *necessities imposed by the hardware you're running on*. That reframe is valuable because it changes the conversation from "you should do this because good engineers do" to "you should do this because your brain literally cannot do otherwise." The former is aspirational and easy to skip. The latter is a design constraint.

**The essay is a prologue, not the argument.** Williams says upfront this is the philosophical foundation beneath a larger manifesto. As a standalone piece, it stakes out the territory beautifully — but it doesn't build on it. The concrete engineering implications are gestured at rather than developed. That's fine for a foundation, but the real test is whether the manifesto that follows actually derives non-obvious practices from the premises, or just re-brands existing advice in cognitive-science language.

**The most provocative implication is left unstated.** If working memory is ~4 chunks and our systems require holding vastly more than that in mind, then *all software methodology is prosthetic cognition*. Agile standups, code review, type systems, documentation, pair programming — the entire apparatus of software engineering is an external memory system that evolved to compensate for a four-slot biological limit. The question isn't which methodology is "correct." It's which arrangement of prosthetics best compensates for the specific gap between your team's cognitive limits and your system's complexity. That's a much more useful question than "Agile vs. Waterfall."

---

## See Also

- [[Agent Memory and Context]] — context management as the real engineering challenge; the synthesis hub for this conversation
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — same thesis applied to agent design: context ≠ continuity
- [[Coding Agents Continuity Not Memory]] — the operational thread, not the context window, is the primitive
- [[Who Does What — Team Topologies for the Agentic Platform]] — cognitive load as the binding constraint on team topology design
- [[Guardrails and Feedback Loops]] — linters beat prompts; deterministic enforcement over instructions
- [[Software Engineering Craft]] — the fundamentals hub: error handling, API design, SRE
- [[99 Bottles of OOP]] — Sandi Metz on OO design as line-by-line decision-making for small minds
- [[The Car Wash Question]] — LLMs fail at obvious inferences for the same reason: attention is fixed
- [[A Non-Anthropomorphized View of LLMs]] — the reality check: LLMs are functions through ℝⁿ, not proto-minds
- [[They're Made Out of Weights]] — "just weights" all the way down, and we've agreed not to care
- [[Man-Computer Symbiosis]] — Licklider's 1960 ur-text: the vision of computation as cognitive prosthesis
- [[I Have ADHD Skill]] — ayghri's output-shaping skill operationalizes these principles as an agent communication protocol: ten falsifiable rules grounded in the same cognitive constraints Williams maps, with a pre-send deletion checklist that treats output formatting as accessibility engineering
- [[The Mundanity of Excellence]] — excellence as qualitatively different choices, not more effort or bigger brains
- [[Thinking Hard Burns Almost No Calories]] — mental fatigue isn't energy depletion, it's adenosine hijacking perceived exertion; related but orthogonal to working memory limits
- [[The Art of Decision-Making]] — Rothman's essay names the same problem from the other side: bounded rationality means our biggest life choices can't be optimized, and the philosophers he surveys argue that's a feature — transformative choices reconfigure the values by which we'd evaluate them

---
*Source: [[summary/bounded-cognition]]*
*Last updated: 2026-07-03*
