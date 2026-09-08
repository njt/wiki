# TALA (Diagram Layout Engine)

Terrastruct's open-source autolayout algorithm for D2 — an orthogonal layout engine tuned for software architecture diagrams rather than the DAG-based engines (Dagre, ELK) that grow in one direction. Its distinguishing move is a division of labor aimed squarely at agents: models place nodes in 2D space, and TALA does the edge routing they still struggle with.

---

## Key Quotes

> "TALA is a novel autolayout algorithm designed with software architecture diagrams in mind. This means it's primarily an orthogonal layout engine, which more closely matches what you might find on whiteboards, rather than the DAG-based ones that grow in one direction."

The core bet. Most autolayout engines (Dagre, ELK, Graphviz's dot) are layered DAG drawers — they want a direction. Architecture diagrams on a whiteboard don't have one; they're orthogonal, right-angle, "rooms connected by hallways." TALA optimizes for the *shape* engineers already draw rather than forcing their mental model into a DAG.

> "It considers multiple objectives of 'aesthetic', including symmetry, median distance, flow, clustering of like nodes, and much more."

"Good layout" isn't a single scalar. TALA treats it as a multi-objective optimization — an honest acknowledgment that the other engines optimize one thing well and everything else emerges.

> "Node positions and sizes can be customized, e.g. locking in the coordinates. This lends itself especially well to agentic use cases, where models can draw in 2D space well, but TALA still takes care of routing, which models still struggle with."

The most wiki-relevant sentence. It inverts the usual AI-diagram pessimism: the problem was never that models can't draw — they place shapes fine. The problem is routing edges without spaghetti, which is the deterministic, optimization-shaped part an algorithm *should* own. TALA redraws the human/machine boundary as placement-to-model, routing-to-engine.

> "It has randomness in the algorithm. It finds the best layout by using a default of 3 seeds and choosing the one scored the best. Given the same seeds and same input, it'll produce the same diagram. But let's say you just add one more node. The diagram could look completely different."

A real, disclosed tradeoff with consequences for any diff-based or version-controlled workflow: adding one node can reflow the entire diagram, where Dagre/ELK keep the prior layout mostly stable. Determinism holds per-seed, not per-increment.

> "It doesn't do DAGs as well. I often find myself preferring Dagre or ELK when I want a long flowing graph."

Refreshing candor from the author of the tool — the right answer is engine-per-shape, not one engine to rule them all.

## Key Themes

- **#tool**: TALA — an MPL-2.0 orthogonal autolayout engine bundled into D2 v0.9.0, callable as `--layout=tala`
- **#concept**: Orthogonal vs. DAG layout as two different answers to "what is a diagram" — whiteboard shape vs. directed flow
- **#pattern**: Placement-to-model, routing-to-engine — the division of labor that makes AI-generated diagrams viable
- **#pattern**: Multi-objective aesthetics — layout quality as symmetry + median distance + flow + clustering, scored across seeds
- **#concept**: Determinism per-seed, not per-increment — the stability cost of randomized global optimization

## Critical Analysis

The open-sourcing itself is a low-key strategic move: bundling a genuinely novel layout engine into an already-open diagramming language (D2) makes the *whole stack* self-contained and inspectable, which matters more than most license announcements. What's more interesting is how explicit the post is about TALA's agentic target. The second batch of examples is explicitly AI-generated, and the framing — "models can draw in 2D space well, but TALA still takes care of routing" — is a precise claim about where model capability ends and deterministic engineering begins.

That claim reframes [[Common Diagram Mistakes]]' "AI can't diagram for you (yet)." Pilger's objection is that diagramming requires *strategic omission* — deciding what not to show — which models lack. TALA doesn't touch that judgment; it only solves the routing sub-problem. So the two positions are compatible, not contradictory: a model still can't decide what to omit, but once the nodes are chosen, placement is model-able and routing is algorithm-able. The useful synthesis is that "AI-generated diagrams" fail at the *curation* layer, not the *geometry* layer — and the geometry layer was always a solvable optimization problem.

The relationship to [[Flint Chart]] is a mirror image. Flint separates *semantics* from *rendering* so agents can be sloppy about chart authoring and still get polished output; TALA separates *placement* from *routing* so agents can be sloppy about edges. Both are instances of the same architectural instinct — find the sub-task agents are bad at, and compile it deterministically rather than asking the model to do it. Where Flint's layout engine is physics-inspired sizing, TALA's is graph-drawing aesthetics; the shared insight is that the *mechanical* half of visual output shouldn't be the model's job.

Next to [[Automatic Layout of Railroad Diagrams]], TALA sits on the far side of the same spectrum. Railroad layout is "1.5-dimensional" — structured enough to compile through three clean passes, avoiding general graph layout's NP-hard swamp. TALA lives *in* that swamp: general 2D orthogonal graph layout, which is why it falls back to randomized multi-seed optimization and scoring rather than a closed-form compilation. That's the real explanation for the disclosed tradeoffs (nonlinear scaling, per-increment instability): the domain is hard enough that "aesthetic" can only be approximated and sampled, not solved.

The one thing the post doesn't confront is the instability cost for version control. If adding a node reflows the whole diagram, then a one-line `.d2` change can produce a fully different `.svg` diff — a nightmare for anyone reviewing diagram changes as code. The author frames this as "sometimes desirable," which is true for final output but elides the diff problem. It's the same tension [[The Dark Factory is a DOT File]] raises about treating diagrams as reviewable artifacts: the layout must be stable enough to diff, or the artifact stops being an artifact and becomes a render.

---

*Sources: [[raw/tala-is-open-source]], [[summary/tala-is-open-source]]*
*Last updated: 2026-09-08*
