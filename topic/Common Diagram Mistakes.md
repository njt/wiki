# Common Diagram Mistakes

Billy Pilger's catalogue of seven anti-patterns in system architecture diagrams, from labeling failures to the temptation of AI-generated diagrams. A follow-up piece from the creator of Ilograph.

---

## Key Quotes

> "Every resource in a diagram should connect to other resources somehow. Including isolated elements undermines the diagram's purpose of showing relationships."

> "System diagramming remains primarily a human endeavor."

## Key Themes

#concept #architecture #communication

**Name your resources.** Icons show type; names show purpose. "Orders Table" beats a generic database cylinder. This is the diagram equivalent of good variable naming -- the same fight, different medium.

**Kill the master diagram.** One diagram that shows everything shows nothing. Break systems into perspectives: runtime, deployment, data flow. Each perspective tells one story. This maps directly to the "multiple perspectives" pattern in model-based diagramming tools.

**Conveyor belt syndrome.** Linear data-flow diagrams lie by omitting round-trips and orchestration. The fix: use sequence diagrams for behavioral interactions. The underlying insight is that static structural diagrams and dynamic behavioral diagrams are different tools for different questions -- using one where you need the other always misleads.

**Fan traps.** When a shared message broker sits between producers and consumers, the diagram collapses specific communication paths into generic arrows. The fix is to model topics or queues explicitly, restoring the visibility of who-talks-to-whom. This is an information-loss problem disguised as a simplification.

**AI can't diagram for you (yet).** Automatically generating architecture diagrams from source code produces vague, hallucinated output. The fundamental problem: diagramming requires strategic omission -- deciding what *not* to show -- and that's a design judgment AI lacks training data for.

## Critical Analysis

The strongest insight here is that most diagram mistakes are *communication* mistakes, not technical ones. The master diagram fails because it tries to serve every audience simultaneously. Conveyor belt syndrome fails because it answers the wrong question. Fan traps fail because they hide relationships the reader needs to see. These are the same failure modes you see in bad documentation, bad APIs, and bad specs -- trying to be everything to everyone, oversimplifying the wrong things, and losing critical information through premature abstraction.

The AI-generation warning (Mistake #7) resonates with what [[graphify]] discovers: auto-extracting structure from code is useful for *exploration* but unreliable for *communication*. graphify handles this honestly with confidence tagging; most AI diagram tools don't. The missing piece Pilger identifies -- strategic omission as a design skill -- connects to [[Specifications as the Product]]: the human value is deciding what matters, not generating artifacts.

The "multiple perspectives" solution to master diagrams echoes the spec decomposition patterns across this wiki. [[The Dark Factory is a DOT File]] argues each pipeline should tell one story. [[Spec-Driven Development]] argues specs should be modular. The principle is the same: one artifact, one purpose, one audience.

Surprisingly practical advice for a topic that usually devolves into tool recommendations. Pilger focuses on *thinking* mistakes rather than tooling, which is the right level of abstraction.

See also [[sql-crack]] for query visualization (diagrams from database operations), [[graphify]] for auto-generated knowledge graphs (the AI limitation Pilger warns about), [[Write Only Code]] for what happens when nobody reads the artifacts.

---
*Sources: [[summary/more-common-diagram-mistakes]]*
*Last updated: 2026-05-14*
