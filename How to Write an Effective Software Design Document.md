# How to Write an Effective Software Design Document

Michael Lynch's pragmatic field guide to design docs, drawn from experience at Google, Microsoft, and his own startups. An excerpt from *Refactoring English: Effective Writing for Software Developers*. The core insight: design docs aren't about completeness — they're about focusing investment where being wrong is expensive.

---

## The Guiding Principle

Lynch's central question: **"What's the penalty for being wrong?"** This is the filter that separates design-level decisions from implementation details. Language choice and storage backends are high-penalty — hard to change, expensive to get wrong. Pagination style ("load more" vs. display all) is trivially changeable. Specifying everything defeats the purpose. The design doc exists to force thinking on the decisions that hurt to reverse.

This converges with [[Specifications as the Product]]'s economic argument from the opposite direction. Where that thesis says "specs are the durable asset because code is disposable," Lynch says "design docs are insurance against the decisions that are expensive to unwind." Same destination, different framing.

## When to Write One

Six yes/no questions. Any single yes makes it likely worthwhile; two or more makes it "almost certainly" so:

1. Multi-person coordination?
2. More than three months of full-time work?
3. Will remain in production beyond six months?
4. Cross-team coordination?
5. Ambiguous or unclear requirements?
6. Would design time prevent a catastrophic outcome?

The honesty about "sometimes the right amount to invest is zero" is refreshing. This isn't process theater — it's a cost-benefit calculation keyed to project risk.

## The 23-Component Checklist

> "A good design doc can save you years of development time."

Lynch walks through 23 components, from title to alternatives considered. The list is comprehensive but the treatment is pragmatic:

**High-value components:**

- **Objective** — A single sentence in plain language on the first page. Any stakeholder should understand it. This is harder than it sounds and more valuable than it looks.
- **Background** — Must make sense to someone who wasn't in the verbal discussions. This is the test most design docs fail. If the doc only makes sense to people who already know the context, it's not a design doc — it's meeting notes.
- **Goals and Non-goals** — Goals described in terms of impact on users/team/company, not implementation. "Minimize outages related to deploying new app versions" not "Add Kubernetes." Non-goals explicitly fence off what's out of scope.
- **Scenarios** — Concrete walkthroughs of the completed system in practice. Lynch understands that abstraction needs grounding. The scenario section is where the design proves it actually works.
- **Diagrams** — "Tremendously valuable" because "reviewers don't have your mental picture of the system." Recommends editable tools (Excalidraw, draw.io, Mermaid, D2, Graphviz). Bans whiteboard photos. The source file or code should be linked for reproducibility. See [[Common Diagram Mistakes]] for what not to do.
- **Alternatives Considered** — Proactively answers "why didn't you do X?" Especially important for options that seemed appealing or required extensive research. A few brief lines per rejected alternative is sufficient; "overkill" to document everything.

**Components most teams skip that they shouldn't:**

- **SLOs** — Measurable latency/availability/scale targets. Lynch contrasts vague mandates like "performant on mobile" with specific targets like "50th percentile latency <=200ms for user-facing HTTP requests." If you can't say what good looks like in numbers, you don't know what you're building.
- **Open Issues / Resolved Issues** — An appendix tracking unresolved flaws, trade-offs, and information gaps. Each entry: problem, possible solutions, immediate next step. When decided, moves to Resolved Issues with the full original discussion retained. This is the [[Capturing Why Engineering Decisions]] ADR pattern built into the design doc itself — decisions as point-in-time records you supersede rather than living documents you maintain.

**The components that reveal Lynch's big-company experience:**

The Security, Privacy, Legal Considerations, and Logging sections wouldn't appear in a startup's design doc template. They're artifacts of Google/Microsoft compliance requirements. But the principle behind them is sound even for small teams: documenting why threats are considered *unlikely* is still useful, because "reviewers might identify threats you overlooked."

## Key Quotes

> "A good design doc can save you years of development time."

Commentary: The ROI claim is aggressive but defensible. A week spent on design that prevents a year of wrong-path implementation is a 52x return. The hard part is knowing which week.

> "What's the penalty for being wrong?"

Commentary: This is the line that earns the article a permanent place in the wiki. It's a decision filter, a prioritization heuristic, and a bullshit detector in seven words. Everything else in the 23-component checklist is scaffolding around this question.

> "Reviewers don't have your mental picture of the system."

Commentary: The argument for diagrams, but also the argument for the entire document. The author has context the reader doesn't. The design doc exists to transfer that context. If you skip it because "everyone already knows," you've confused your mental state with the team's.

> "Sometimes, the right amount to invest in a design doc is zero."

Commentary: The honesty that saves the article from being process propaganda. Lynch isn't selling a methodology — he's providing a toolkit and trusting you to know when to use it.

## Key Themes

- `#pattern` **Design docs as decision insurance** — Invest where the penalty for being wrong is high. Skip where it's low. This is risk-calibrated documentation, not documentation-as-virtue.
- `#concept` **The penalty-for-being-wrong filter** — A portable heuristic for deciding what deserves design attention. Works for features, architecture, tool choices, and documentation itself.
- `#pattern` **Open Issues as living appendix** — Track unresolved problems, trade-offs, and information gaps. When resolved, move to Resolved Issues with the full original discussion. Converges with the ADR-as-RFC pattern from [[Capturing Why Engineering Decisions]].
- `#concept` **Diagrams as context transfer** — Reviewers don't have your mental model. Diagrams bridge the gap. Editable source files, not whiteboard photos.
- `#tool` **SLOs as design requirements** — If you can't express what "good" looks like in numbers, you haven't finished designing. Vague mandates ("fast," "reliable") are wishful thinking, not requirements.
- `#concept` **Background for the absent reader** — The test: can someone who wasn't in the meetings understand why this project exists?

## Critical Analysis

**The article is better than its genre.** "How to write a design doc" is a well-worn topic that usually produces either academic formalism (IEEE 830) or process propaganda (Scrum ceremonies extended to documentation). Lynch avoids both. His Google pedigree gives him credibility, but his startup experience keeps him honest — the admission that "sometimes zero is the right amount" is something a career Googler wouldn't say and a process consultant couldn't say.

**The penalty-for-being-wrong filter is the keeper.** It's a genuinely useful heuristic that applies beyond design docs. What tests to write? What to monitor? What to document? What to review? The answer each time: whatever's expensive to get wrong. This is Lynch's real contribution and it's a better decision heuristic than most frameworks that took entire books to develop.

**The 23 components are too many for most teams.** Lynch doesn't claim you need all 23 — he's providing a menu, not a template. But the Security/Privacy/Legal/Logging tail betrays enterprise assumptions. A 3-person startup doesn't need a Legal Considerations section in their design doc. The risk is that junior engineers will cargo-cult the full checklist and produce 50-page documents that nobody reads, missing Lynch's own point about investment calibration.

**The SLO section is underrated and under-explained.** Lynch treats SLOs as a single component in a 23-item list, but they're arguably the most important one. A design doc without measurable success criteria is a wish list. The jump from "the system should be fast" to "p50 latency <=200ms" is the difference between engineering and aspiration. More space on this would have been justified.

**What's conspicuously absent: design doc review.** The article is about *writing* design docs, not about the review process that makes them valuable. Lynch links to a separate article on getting useful feedback, but the write/review cycle is where design docs earn their keep or fail. [[Vibe Coding as a Team Sport]]'s two-gate review pattern and [[The End of Code Review]]'s argument for architectural coherence through design docs (not per-commit review) are the complementary pieces.

**The example design doc is a power move.** Lynch wrote a complete design doc for "Little Moments" before writing any code and is adhering to it during implementation. This is the strongest possible endorsement of his own advice. Most authors in this genre don't eat their own dog food.

**Comparison to SDDW:** [[SDDW (Spec-Driven Development Workflow)]] is the operational implementation of the spec-first philosophy for AI-assisted development. Lynch's article is the human-scale version — no agents, no pipeline, just one engineer writing a document for other engineers. But the underlying principles are identical: make decisions before code, capture rationale, separate design from implementation. SDDW automates the process; Lynch explains the craft.

**Comparison to Slowing Down:** [[Slowing Down in the Age of Coding Agents]] argues the bottleneck has shifted from code production to thinking. Lynch's design doc is the artifact that results from that thinking. Odendahl annotates agent-generated designs on e-ink; Lynch writes them from scratch. Same insight, different toolchain.

**What the article gets right that almost everyone else gets wrong:** The Alternatives Considered section. Most design docs present the chosen approach as inevitable. Lynch understands that documenting *rejected* alternatives — especially the ones that seemed appealing — is more valuable than documenting the chosen one. It preempts review arguments, captures research that would otherwise be lost, and prevents future teams from re-litigating decisions without the original context. This alone justifies the design doc's existence.

---

*Sources: [[raw/how-to-write-an-effective-design-doc]]*
*Last updated: 2026-07-03*
