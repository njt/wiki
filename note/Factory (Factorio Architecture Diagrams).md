# Factory (Factorio Architecture Diagrams)

Factory by echohack is a Claude Code Agent Skill that explains programs, algorithms and whole systems as **working, animated Factorio factories**. `/factory explain how PostgreSQL makes a COMMIT durable` produces a browser page where belts are streams, inserters are discrete handoffs, assemblers are work, and chests are stored state — drawn with genuine Factorio 2.0 sprite art imported from the user's own game install. Its bet is that a shared, physical, already-familiar design language beats mermaid and UML for architecture communication, and that the way to get an LLM to draw well is not better prompting but a constrained library of patterns with machine-checked rules.

#tool #project #agents #claude-code #visualization

---

## Architecture

The repo follows what it calls "Anatomy's useful separation" — instructions, reusable code, library, examples, checking tools:

- **`SKILL.md`** (239 lines) — the agent-facing instructions. Notably directive prose, not vibes: write an architecture inventory from the code with a file reference per item; map semantics before arranging; run `checkLayout` in the figure's tests; never download Factorio art from anywhere else; stop and ask the user to import if `assets/` has no PNGs.
- **`kit/`** — the runtime: `scene.mjs` (`FactoryScene`, a Canvas renderer that fits tile coordinates to a responsive canvas, draws shadows-then-bodies in ground-y order, and renders hanging wires with quadratic sag); `trace.mjs` (`TracePlayer`, a deterministic event-sourced playback engine where each event has `before`/`after`/`duration` — pause, step, seek, reset all operate on committed event boundaries); `sprites.mjs` (atlas decoding only the sprite IDs a scene lists); `transport.mjs` (inserter pickup/carry/release/return poses).
- **`catalog/`** — 68 world objects and 28 item tokens (`objects.mjs`, `vocabulary.mjs`), each carrying fixed vocabulary tags and one fixed meaning: an iron plate is always a record, copper cable a packet, an electronic circuit a request. Chest types map to software roles (requester chest = subscriber, active provider = flush/checkpoint).
- **`patterns/`** — six families (handoffs, belts, cells, fluids, layout, control) of arrangements Factorio players build repeatedly, each with one fixed software meaning: worker row = worker pool, splitter fan-out = load balancer, full belt = backpressure. `rules.mjs` implements the ground rules — whole-tile placement, one object per tile, exact inserter reach, underground pairing, power pole supply — as `checkLayout(layout)`, returning one human-readable violation per broken rule.
- **`examples/`** — complete figures. `obs-pipeline/model.mjs` (275 lines) is a pure, independently executable simulation of OBS: camera/game capture → async queue → graphics thread → preview cache → x264 encoder → stream/record outputs, plus mic/desktop → mixer → AAC, with a send buffer that drops P-frames under a `network` scenario (upload 0.6 vs 3). It precomputes an event trace from a discrete-event simulation with per-route capacity and handoff-cycle timing, then playback is pure re-presentation.
- **`scripts/import-assets.mjs`** — reads the game's `--dump-data` and `--dump-icon-sprites` output from Steam/standalone install locations, hashing and manifesting sprites. Game art is gitignored; only the manifest and catalog JSON are tracked. A `contained()` helper path-checks every asset against realpath escape.

## Key techniques

- **Deterministic model / presentational playback split.** The model computes every event trace up front (a discrete-event queue with `at(time, fn)` scheduling); `TracePlayer` merely walks committed event boundaries. Animation interpolates but "must not invent work, lose values, or change the answer." A stopped machine is visually stopped. This is the same truth-first discipline as [[Anatomy — a Skill for Claude Code]], enforced in code rather than prose.
- **Conservation checks.** Tests count values through the pipeline including inserter hand contents — an item must never be in both a chest and an inserter. This makes the diagram a simulation with an invariant, not an animation.
- **Machine-checked layout legality.** `checkLayout` encodes real Factorio physics (tile occupancy, inserter reach of one tile behind / drop in front, long-handed two-tile spacing, pole supply radius) so the agent physically cannot produce an impossible scene without a test failing. Geometry is linted, like [[Diagram Design]] lints SVG grammar.
- **Fixed semantic vocabulary.** 100+ terms (`write-ahead log`, `dead letter`, `load balancer`) tag both objects and patterns, so the agent can *search* the library by software concept (`vp run patterns retry`) and readers can transfer literacy between figures. Token meanings are pinned so a copper cable is never a "record" in one figure and a "packet" in another.
- **Asset boundary as design constraint.** The skill ships no art and refuses to fetch any; the user's own game copy is the source of truth for dimensions, offsets and frame counts. Copyright-clean, and it forces the figure to use real game rendering rather than approximations.

## Design decisions

- **Patterns over freeform composition.** Any arrangement is legal only if it satisfies ground rules; preferred arrangements come from the pattern library. The trade: far less layout freedom, far more cross-figure readability — the reader who learned one worker row reads every worker row.
- **Honesty rules over spectacle.** The skill forbids implying visual concurrency or physical distance measures timing unless the data supports it; parallel lanes "show independence, not measured timing"; omissions must be recorded with reasons ("no audio in a media pipeline is never an omission"). It is unusually explicit that the diagram can lie and pre-empts the specific lies.
- **Ruthless page minimalism.** One row of controls, no narration, no legends, no code listing, no status footer; the scene and readouts carry state. Static reference tables below the figure deliberately do not re-render during playback.
- **Aesthetic fixity.** A single "midnight blueprint" visual treatment, with textures credited to the author's other project (Campfire), is specified down to hex values — another way of removing per-generation taste variance.

## Comparison notes

- Closest cousin is [[Anatomy — a Skill for Claude Code]]: both make the mechanism *be* the explanation with a shared truth model. Factory differs by fixing the metaphor to one pre-existing game world with real assets and physics-checked layouts, versus Anatomy's invented machines as freeform SVG. Factory's patterns give cross-figure shared literacy; Anatomy's invention gives per-idea fit.
- Against [[Diagram Design]] (40 static editorial diagram types + CI gates), Factory is a single visual type with *runtime behavior*, and its "CI gate" equivalent is `checkLayout` plus conservation tests. Both solve "AI diagrams are slop" by constraint systems, not prompting.
- As skill design it is a strong example of the pattern in [[Claude Code Skills System]]: a SKILL.md directory with reusable code and references, deployed by cloning into `~/.claude/skills/` (also Codex and OpenCode paths), with the generated art gitignored.
- It also serves the planning workflow described across [[Agent Coding Workflow]] — the README explicitly pitches it for visualizing RFCs and plans, a verification/articulation aid rather than a code generator.

*Sources: [[raw/factory]], [[summary/factory]]*
*Last updated: 2026-10-18*
