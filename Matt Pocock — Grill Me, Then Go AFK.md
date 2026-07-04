# Matt Pocock — Grill Me, Then Go AFK

Matt Pocock (Total TypeScript) lays out his full AI-assisted development pipeline: a structured workflow from idea to shipped code that keeps the human in the loop for planning and QA but goes fully AFK for implementation. The talk's two most original contributions are the **smart zone / dumb zone** model of LLM context windows and the **grill me** alignment technique. The rest is a polished synthesis of ideas circulating in the agentic-coding community — TDD, kanban boards, tracer bullets, deep modules — assembled into a coherent pipeline by someone who's actually shipping with it.

---

## The Smart Zone

Pocock's most useful mental model: LLMs have a **smart zone** (~100k tokens) where attention relationships are reliable, and a **dumb zone** beyond it where quality degrades quadratically. Every token you add is like adding a team to a football league — the number of relationships scales as n², and the model's ability to maintain them degrades.

This is the engineering constraint that shapes everything else: task sizing, sub-agent delegation, context clearing. It's presented as a rule of thumb without rigorous evidence, but as a design heuristic it's powerful. If you accept it, you design differently — smaller tasks, more context resets, less reliance on long-running sessions.

The companion insight is the **Memento model**:

> "LLMs are kind of like the guy from Memento, right? They just continually forget, they could just keep resetting back to the base state."

Pocock prefers clearing over compacting. A cleared context gives a predictable, repeatable starting point. Compacting (summarizing history) produces a degraded, non-deterministic state. This is the cleanest argument I've seen for why the [[Ralph]] loop pattern (fresh context per iteration) beats long-running agent sessions. The Memento protagonist doesn't get worse at his job each time he resets — he starts from the same clean slate. Your agent should too.

---

## The Grill Me Skill

This is the most original technique in the talk, and it's brilliantly simple:

> "I didn't need an asset, I didn't need a plan. I needed to be on the same wavelength as the AI, as my agent."

The **grill me** skill is a tiny prompt that makes the AI interview you relentlessly, one question at a time, about every aspect of a plan. The goal isn't a document — it's shared understanding. You don't read a spec; you *arrive at* a design concept together through Socratic questioning.

This inverts the usual relationship. Instead of you prompting the AI, the AI prompts you. It forces you to make decisions you'd otherwise defer. It surfaces assumptions you didn't know you had. By the end, you and the agent share a mental model of what you're building.

The PRD that follows is a **destination document** — the agent writes it, the agent reads it. Pocock explicitly doesn't review it:

> "I don't look at these [PRDs]. The reason I don't look at these is because what am I testing at this point? … I know that LLMs are great at summarization… I have reached the same wavelength as the LLM."

This is either disciplined trust or dangerous delegation, depending on how good the grilling session was. If the shared understanding is real, the PRD is a formality. If it's not, you're building the wrong thing and won't know until QA.

---

## The Pipeline

The full workflow, end to end:

1. **Grill me** — Socratic alignment, human in the loop
2. **Write PRD** — Agent produces a destination document (human doesn't read it)
3. **PRD → Kanban** — Decompose into vertical slices (tracer bullets) with blocking relationships
4. **AFK implementation** — Agents grab unblocked issues, implement with TDD, commit on green
5. **Manual QA + code review** — Human taste and judgment applied here, not before

The pipeline's philosophy: **planning is human-in-the-loop, implementation is AFK, QA is human again**. This is a clean separation of concerns. The human does what humans are good at (design, taste, judgment). The agent does what agents are good at (grinding through implementation with mechanical verification).

The Kanban board is the coordination surface — same pattern as [[Managing Agents via Kanban Boards]] and [[ralph-ban]]. Blocking relationships create a DAG of tasks, enabling parallel agent work on independent branches.

---

## TDD as the Agent Unlock

> "TDD is so, so good for places where you can pull it off. And in fact, it's so good that I sort of warp my whole technique around getting TDD to work better."

Pocock's argument for TDD with agents is specific and practical: if the AI writes tests after implementation, it cheats. It writes tests that pass the implementation rather than tests that verify the specification. Red-Green-Refactor prevents this — the test must fail first, so the AI can't retro-fit tests to whatever code it produced.

This aligns with [[Simon Willison — Engineering Practices That Make Coding Agents Work]] and the broader [[Guardrails and Feedback Loops]] theme: the quality of your feedback loops IS the ceiling for AI code quality. Tests, type checks, linters — these are the ratchet that prevents regression. Improving them is the highest-leverage investment you can make.

> "The quality of your feedback loops influences how good your AI can code. Essentially, that is the ceiling."

---

## Deep Modules, Shallow Modules

Pocock channels John Ousterhout's *A Philosophy of Software Design*:

> "Bad code bases make bad agents. If you have a garbage code base, you're going to get garbage out of the agent."

**Deep modules** (small interface, lots of internal functionality) are AI-friendly because the agent only needs to understand the interface to use the module. **Shallow modules** (many small files with tangled dependencies) force the agent to load enormous context to understand anything.

This is a concrete, actionable insight: architecture decisions that predate AI agents now have a second-order effect on agent productivity. The modules you designed for human maintainability may or may not be good for agents — but deep modules benefit both.

His **Improve Code Base Architecture skill** scans repos for shallow modules and suggests deepenings. This is [[Compound Engineering]] applied to architecture: each cycle makes the codebase more agent-friendly, which makes the next cycle faster.

---

## Where Taste Lives

> "If you try to automate the sort of creation of the idea, automate the QA, automate the research, automate the prototype, you end up with apps that I feel just lack taste and are bad."
> 
> "We are not producing slop here. We're trying to produce high quality stuff."

QA is where human judgment enters. Not in code review of every line — that doesn't scale — but in *using the thing* and deciding whether it's good. This is the clearest articulation I've seen of where the human belongs in an otherwise automated pipeline. Taste can't be automated. Judgment can't be prompted. You have to look at the thing and decide.

This aligns with [[Radical Accountability]] (taste is all that's left when building is cheap) and stands against the pure [[Vibe Coding and the Maker Movement]] extreme where nobody looks at anything.

---

## Critical Analysis

**What's strong.** The smart zone / dumb zone model is a genuinely useful design heuristic, even if the 100k boundary is hand-wavy. The grill me skill is original and practical — I expect variants of it to become standard in agentic workflows. The pipeline's separation of concerns (human plans, agent builds, human judges) is clean and scalable. The TDD argument is specific and persuasive.

**What's hand-wavy.** The 100k token boundary is presented as universal despite no evidence and despite context windows growing rapidly. If the smart zone is always "about 100k" regardless of total window size, that's a fundamental architectural limitation worth naming explicitly. If it scales with window size, the claim needs updating. Either way, present it as a heuristic, not a law.

**What's missing.** The PRD-as-unreviewed-artifact pattern assumes the grilling session produced perfect alignment. It won't always. The first time the agent builds the wrong thing and you don't catch it until QA, you've burned a full cycle. A lightweight verification step between grill and PRD — a one-paragraph summary the human *does* read — would catch misalignment cheaply.

The talk is also entirely solo-developer. How does this work with a team of 5? Who runs the grill me sessions? Who owns the PRDs? What happens when two developers have different mental models of the same feature? These are the hard problems, and they're not addressed.

**What's wrong.** The claim that "LLMs are great at summarization so PRDs don't need human review" is the weakest link. Summarization quality varies enormously by domain, complexity, and model. Hallucinations in summaries are well-documented. The grill me session is the defense against this, but it's a single point of failure.

**The old books insight is underrated.** Pocock mentions that *Pragmatic Programmer*, *Refactoring*, and *A Philosophy of Software Design* are gold for prompting because they verbalize best practices in English. This is a deeper point than it sounds: the books that taught humans to engineer software are now the books that teach AI to engineer software. The knowledge transfers because it was always about principles, not syntax. [[99 Bottles of OOP]] and [[Software Engineering Craft]] are the same class of resource.

---

## Key Themes

#agentic-coding #context-engineering #tdd #workflow #kanban #deep-modules #feedback-loops #taste #alignment #grill-me

## Cross-References

- [[Agent Coding Workflow]] — This pipeline sits at the compound-engineering end of the maturity spectrum
- [[Ralph]] — The AFK loop is a direct implementation of the Ralph/Wiggum pattern
- [[Managing Agents via Kanban Boards]] — Same coordination surface, different implementation (Notion vs. Sandcastle)
- [[ralph-ban]] — Kanban board as agent task manager
- [[Compound Engineering]] — The improve-architecture skill is compound engineering applied to codebase structure
- [[Guardrails and Feedback Loops]] — TDD, type checks, and linters as the quality ceiling
- [[Specifications as the Product]] — The grill me → PRD pipeline as specification-first development
- [[Loop Engineering]] — The meta-skill Pocock is practicing: designing systems that prompt agents
- [[Agent Memory and Context]] — The smart zone model as context engineering
- [[Agent Orchestration]] — Kanban DAG with blocking relationships as multi-agent coordination
- [[Breaking the Spell of Vibe Coding]] — Pocock's "specs-to-code sucks" is the same diagnosis
- [[Simon Willison — Engineering Practices That Make Coding Agents Work]] — TDD as the agent unlock
- [[Don't Fear the Dark Factory]] — Same insight: the loop is simple, the harness is the work
- [[Designing Agentic Loops]] — Willison on the same meta-skill
- [[Software Engineering Craft]] — The old books as prompt engineering gold
- [[99 Bottles of OOP]] — Another classic that verbalizes principles agents can execute
- [[Memento]] — The film metaphor applied to agents (different context: knowledge management, not context windows)
- [[Vibe Coding and the Maker Movement]] — The extreme Pocock is pushing against
- [[Radical Accountability]] — Taste as the differentiator when building is cheap

---
*Source: [[raw/matt-pocock-smart-zone-afk-pipeline]] — ytx gist [a80e60a6ff03bd5a853c91b130d22bd0](https://gist.github.com/a80e60a6ff03bd5a853c91b130d22bd0)*
*Date ingested: 2026-07-04*
