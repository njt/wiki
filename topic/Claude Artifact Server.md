# Claude Artifact Server

A catalog of 22 Claude-generated interactive web artifacts, all produced on a single day (2026-03-29) and hosted at `artifacts.yolo.scapegoat.dev`. Every artifact wears the same design language: classic Macintosh System 1-era chrome, Chicago/Monaco typography, and desktop metaphor chrome. What matters isn't any individual tool — it's that an agent produced 22 polished, themed, single-file interactive applications in one session, spanning SQL IDEs, debuggers, lambda calculus tutorials, and ambient loading screens.

---

## The Artifacts

The 22 artifacts fall into rough clusters:

**Developer tools (heavyweight):** QueryMac (Mac-style SQL IDE), Datalog Notebook, Behavior Diff Provenance (version timeline + diff), Trace To Patch Debugger (stack frames → tests → patch proposals), Collaborator Atlas (object graph explorer), Playwright Browser (browser automation in retro chrome), Intent Console (constraint-driven prompt surface), Object-First Commands (typed command composition).

**Knowledge & writing:** Editor (single-file text editor), Deep Research Mac (research environment), The Lambda Calculus (interactive essay at 62 KB — the largest artifact), MathJax Formatter, Presentations.

**Interface experiments:** Retro Launcher, System 1 Tiling WM, Agent Workbench (multi-agent pane coordinator), Classic Mac Chat Browser, Business App (dashboard mockups).

**Aesthetic one-offs:** Severance Loading Screen (11 KB — the smallest), Chart Widget.

All created 2026-03-29. Format split: `.html` for standalone artifacts, `.jsx` for React-based ones. Sizes range from 11 KB to 62 KB — intentionally compact single-file applications.

---

## Key Quotes

> "22 artifacts from /app/imports"

Twenty-two artifacts in one day. The number is the story — not the polishing of any single one.

> "First pass only: type to filter locally in the browser. A richer search UX can layer on top of the same search bundle later."

The filter note is the most honest artifact metadata on the page. It's an admission that this catalog is a first draft — functional but unfinished — and that the architecture supports iterative refinement. This is the artifact-server equivalent of "TODO: make this better later."

---

## Key Themes

#tool #claude-code #retro #agentic-development #design #interactive-artifacts

The dominant theme is **generation at scale.** Claude produced 22 distinct interactive applications, each themed to a consistent design language, in what appears to be a single session. This is a showcase of generation speed, not refinement depth.

The **retro aesthetic as brand.** Classic Mac OS design isn't just a style choice — it's a coherence mechanism. When an agent generates 22 different tools across wildly different domains (SQL, lambda calculus, browser automation, business dashboards), a strong visual theme is the glue that makes the collection feel intentional rather than random.

**Single-file architecture.** Every artifact is self-contained. No build step, no dependency tree, no deployment pipeline. This is [[vibes-cli]]'s "the constraint is the feature" applied at catalog scale: when artifacts are single files, the server is just a directory listing.

---

## Critical Analysis

**The interesting part isn't any one artifact — it's the batch.** Individually, these are demos. QueryMac doesn't need to be a real SQL client; it needs to prove Claude can generate a convincing one. The Lambda Calculus essay doesn't need to be novel; it needs to demonstrate structured, interactive educational content. Each artifact is a capability proof, and the collection is a portfolio.

**The yolo.scapegoat.dev domain name is doing work.** "YOLO mode" in Claude Code means the agent operates without permission prompts. "Scapegoat" implies this server catches the output — the artifacts that would otherwise vanish when the session ends. The domain is infrastructure for [[Write Only Code]]: generated faster than reviewed, hosted for later inspection.

**The format split (.html vs .jsx) is telling.** The simpler tools (Editor, Chart Widget, Severance screen) ship as standalone HTML. The more complex ones (debuggers, the Playwright browser wrapper, Intent Console) use React. This suggests a complexity threshold where the generation shifts from vanilla JS to React components — an implicit judgment about when a framework earns its weight.

**The retro aesthetic carries unexamined assumptions.** Classic Mac design constrains interaction to menu bars, overlapping windows, and dialog boxes. It's charming, but it forecloses any post-1984 interaction model. Tabs, search-as-primary-interface, infinite scroll, touch — none of these exist in the Mac System 1 vocabulary. The aesthetic is a straitjacket as much as a brand, and none of these artifacts question whether a 1984 interface is the right container for a 2026 tool.

**The catalog is the product, not the artifacts.** Nobody is deploying QueryMac into production. The artifact server demonstrates what an agent can *produce*, not what it can *maintain*. The artifacts are frozen on their creation date. This connects to [[Specifications as the Product]]: these artifacts are disposable generation output, not durable software. The server is the durable artifact.

**Relation to [[vibes-cli]]:** Both generate single-file HTML applications. But vibes-cli targets non-coders building collaborative tools, while the Artifact Server is a developer showcase. Different audiences, same enabling constraint.

**Relation to [[json-render]]:** Vercel's generative UI framework constrains LLM output to a typed component catalog, yielding progressive rendering. The Artifact Server takes the opposite approach: unconstrained generation within a visual theme, yielding standalone files. Two different answers to "how do you make AI-generated UI feel coherent?"

**Relation to [[Intent Is the Interface]]:** The Intent Console artifact explicitly explores the "design capabilities, derive interfaces" thesis. Among the 22, it's the most conceptually aligned with the wiki's agent-native philosophy.

**Relation to [[Experience Design for Agents]]:** These artifacts are experience design for *human users of agent output*, not for agents themselves. But the same principle applies: UX determines whether the output gets used.

---

*Sources: [[summary/claude-artifact-server]]*
*Last updated: 2026-05-22*
