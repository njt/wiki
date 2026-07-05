# StrongDM Factory Techniques

Six practitioner patterns from the team building production software with zero hand-written code and zero traditional code review. This is the technique catalog for [[The Dark Factory is a DOT File]] — the "how" behind the architecture diagram. StrongDM's Justin McCarthy is the origin point [[Don't Fear the Dark Factory|Matt Wynne credits]] for the dark factory concept.

---

## The Techniques

### Digital Twin Universe (DTU)

> Clones the observable behaviors of critical third-party dependencies. Enables validation at volumes beyond production limits with deterministic, replayable conditions.

This is [[Harness Engineering]] operationalized: the DTU is a feedforward system. You define expected behavior of a dependency (API responses, failure modes, timing characteristics), clone it, and validate against the clone at scale. The insight is that real dependencies are the wrong substrate for testing — they're non-deterministic, rate-limited, and can't be pushed to failure conditions. The DTU gives you a deterministic, pushable replica.

### Gene Transfusion

> Transfers working patterns between codebases by directing agents to concrete examples. A good solution paired with a reference can be reproduced in fresh contexts.

A concrete answer to the abstraction problem in agent programming. Instead of trying to encode "good error handling" or "how we do auth" as rules, you point the agent at a file that does it right and say "like that." This works because LLMs are better at pattern-matching from examples than following abstract instructions. Echoes [[RepoMirror]]'s finding that 103-word prompts beat 1,500-word prompts — examples are higher-bandwidth than explanations.

### The Filesystem

> Models navigate repos efficiently and modify their own context via file read/write. Directories, indexes, and on-disk state become a practical memory substrate.

[[Planning With Files]] as architecture, not just a skill. The filesystem is the agent's long-term memory — not a vector database, not a context window, but directories and markdown files that the agent reads and writes to maintain state across sessions. This is deliberately low-tech. The constraint is real: context windows are RAM (volatile, limited); files are disk (persistent, unlimited). StrongDM's contribution is making this a first-class technique rather than an implementation detail.

### Shift Work

> Separates interactive tasks from fully specified ones. Once intent is complete — via specs, tests, or existing apps — agents run end-to-end without iterative back-and-forth.

The most operationally significant technique in the catalog. "Interactive" means the human is still figuring out what they want — exploring, specifying, designing. "Fully specified" means the intent is locked: there are tests, an existing implementation to port, or a spec complete enough that correctness is mechanically checkable. Once you cross that line, you hand it to [[Serf]]-style non-interactive agents and walk away. This is the [[Ralph]] loop pattern with an explicit gating function.

The implication: if your agents need constant back-and-forth, your spec isn't done. The bottleneck isn't the agent — it's the clarity of intent.

### Semport

> Semantically-aware automated ports, one time or ongoing. Moves code between languages or frameworks while preserving original intent.

[[RepoMirror]] at production scale. The "semantic" qualifier matters — this isn't syntactic translation (for-loop becomes for-loop), it's preserving what the code *means* and re-expressing it idiomatically in the target language. Ongoing ports mean you can maintain parity between a Python reference implementation and a Rust production system, with Semport keeping them in sync. This collapses the cost of "let's try it in a different language" to near-zero.

### Pyramid Summaries

> Reversible summarization at multiple zoom levels. Compresses context while retaining the ability to expand back to full detail.

Context compression with a back button. Most summarization is lossy and irreversible — you compress a 50K-token codebase to 2K tokens and can't get back. Pyramid Summaries preserve the ability to "zoom in" on any section. This is what makes [[Agent Memory and Context|context management]] tractable at scale: you navigate at the summary level, expand on demand. The pyramid structure means you can have 3-line, 3-paragraph, and 3-page summaries of the same material, each expanding into the next.

---

## The Validation Constraint

The meta-technique that governs all six:

> A system built with zero hand-written code and zero traditional review. Validated automatically without semantic inspection of source. Code is treated like an ML model snapshot — opaque weights whose correctness is inferred solely from external behavior.

This is the factory's constitution. It's a constraint, not a preference — a deliberate choice to treat code as non-reviewable artifact and invest ALL quality effort in validation harnesses. The analogy to ML model weights is precise: you don't inspect individual weights to judge a model; you run it against a test set. Same deal here.

This is either the most honest framing in agentic development or the most dangerous, depending on whether your validation harness is actually adequate. The entire enterprise rests on that "if."

---

## Key Themes

- #pattern — six named, reusable techniques from production agentic development
- #dark-factory #spec-driven #validation #context-management #porting #autonomous-agents
- #person — Justin McCarthy / StrongDM as the dark factory origin point
- #concept — code as opaque weights: correctness inferred from behavior, not inspection

---

## Critical Analysis

**This is the technique catalog everyone else is missing.** Most dark factory writing stays at the architecture level (pipeline engines, DOT files, five-level frameworks) or the testimony level ("I did this and it worked"). StrongDM's techniques page is the missing middle: named, specific patterns you can actually apply. "Gene Transfusion" is more useful than "use examples." "Shift Work" is a sharper knife than "batch your tasks."

**The catalog is short and specific, not comprehensive and vague.** Six techniques. That's it. Not 169 patterns, not a taxonomy of everything. Six things they actually reach for. This restraint is itself a technique — the discipline to name only what's proven.

**What's missing: failure modes.** Every technique has a shadow. DTU validation is only as good as your behavioral model of the dependency — miss an edge case and you're validating against a lie. Gene Transfusion propagates bad patterns as efficiently as good ones. Shift Work assumes specs can be complete, which is false for most novel work. The page presents techniques without their failure conditions, which makes it read as marketing even though the underlying ideas are sound.

**The validation constraint is doing enormous rhetorical work.** "Code as opaque weights" is a brilliantly concise framing, but it smuggles in an assumption: that validation harnesses can be cheaper to build than code review is to perform. For CRUD apps with clear behavioral contracts, yes. For systems where correctness is subtle (security properties, distributed consistency, UX coherence), the harness IS the hard part. The constraint doesn't eliminate complexity; it moves it.

**Connection to [[The Dark Factory is a DOT File]]:** The DOT file page describes the factory architecture; this techniques page describes what you *do inside* the factory nodes. They're companion documents. Read together, they're the most complete public description of how StrongDM builds software. Neither is sufficient alone.

**Why this matters:** May 2026 is the moment when "we don't read the code" stops being a provocation and starts being an engineering discipline. StrongDM's techniques page is the closest thing to a field manual for that discipline. It's not a methodology — it's six patterns and a constraint. That might be enough.

---

*Sources: [[summary/strongdm-factory-techniques]]*
*Last updated: 2026-05-15*
