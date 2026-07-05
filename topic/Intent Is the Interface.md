# Intent Is the Interface

Ramon Marc argues that UI design has spent decades refining shadows while neglecting the object casting them. A capability — "request a ride" — exists independent of its interface. The app, the voice command, the watch tap are just projections. With AI agents inverting the initiation model (they act, then surface for approval), the screen-first paradigm breaks: interfaces must be derived from intent and context, not designed by hand for each surface.

---

## Key Quotes

> "Decades refining shadows. The object casting them got neglected."

The essay's sharpest line. UI design as a discipline has optimized the surface layer — pixel precision, animation curves, responsive breakpoints — while the underlying capability model remained implicit and under-theorized. This is the kind of observation that's obvious once stated but reorganizes how you see the entire field.

> "The screen was a constraint we mistook for the product."

A more actionable version of the same critique. Every design tool assumes a viewport. Every design system assumes components render in a rectangle. Marc's point is that these weren't design decisions — they were inherited constraints from the technology stack that nobody questioned.

> "The interface isn't designed. It's derived."

The core thesis in five words. If capabilities decompose into intents and context supplies the projection parameters, then the interface is a function of (intent, context) → expression. This is the design equivalent of what [[Smart Models Dumb Pipes]] argues for system architecture: separate judgment from execution, then let the infrastructure handle the mechanical work.

> "Agents invert this. They act, then surface for approval. You respond."

The second broken assumption. Fifty years of HCI assumed the user initiates and the system responds. Agent-driven interaction reverses the polarity: the system acts, the human approves or redirects. This makes the interface a negotiation surface, not a control panel. Static layouts can't serve this pattern.

> "Systems built around screens will bolt AI on awkwardly. Systems built around intents will treat AI as another projection surface."

The practical fork in the road. Incumbents with screen-centric architectures will struggle to retrofit agent-native interactions. Greenfield systems that model capabilities and intents first can render to any surface — screen, voice, API, agent — as a projection.

> "Conversation has no viewport."

The shortest version of the argument. Voice and chat interfaces don't have width, height, or scroll position. If your design model assumes a viewport, you've already lost the ability to design for the fastest-growing interaction modalities.

---

## Key Themes

#concept #design #intent #agent-interaction #multi-surface

### Capabilities, Not Screens

The fundamental unit of design should be the capability — what the user can accomplish — not the screen that enables it. A capability ("request a ride") can manifest as a phone screen, a voice command, a watch tap, or an API call. Each is a projection of the same underlying thing.

This connects directly to [[Experience Design for Agents]]'s four-layer model, where "human owns intent/judgment" is the top layer. Marc pushes the boundary further: the interface layer itself should be derived, not designed. Where Kemple describes responsibilities, Marc describes a rendering pipeline.

### Eight Intents

Marc proposes that all interface actions decompose into eight intent patterns. He illustrates these but doesn't enumerate them in the text — they appear as a framework graphic. The claim is that this small set covers "most of what we build," making it a candidate universal grammar for interaction design.

This is adjacent to [[Elements of Agentic Systems Design]]'s ten-element taxonomy, but operating at the interaction level rather than the system level. Both share the ambition of finding a small set of primitives that compose into everything.

### Context as the Derivation Engine

Context — with at least six dimensions — is what transforms an abstract intent into a concrete interface. "Same intent. Different situation. Different expression." Screen width (the obsessive focus of responsive design) is just one of these six dimensions.

The implication is that responsive design was a special case of a general problem nobody knew they had. [[json-render]]'s generative UI from Zod-constrained JSON is an implementation of this idea: the same data structure renders differently based on context.

### The Inversion of Initiation

When agents act first and surface for approval, the interface becomes "a negotiation you manage" rather than "a tool you operate." This is a fundamentally different interaction model that most design tools and patterns don't support.

[[From AI Studio to AI Forge]] describes this as "human changes altitude" — the human moves from operating controls to supervising decisions. Marc provides the interaction-design vocabulary for what McCormick described architecturally.

---

## Critical Analysis

The essay is a genuine contribution to thinking about post-screen interaction design. The "decades refining shadows" line alone is worth the read. It names something that's been under-specified: we've been optimizing the wrong unit.

But it's a manifesto, not a framework. The eight intents are asserted, not validated. The six dimensions of context are illustrated, not enumerated. The central claim — "the interface is derived" — is stated as conclusion rather than demonstrated as method. This is a vision document dressed as a framework, and it works better as the former than the latter.

The hard problems are acknowledged but handwaved with admirable honesty: "These are real problems. Unsolved." The entire paradigm depends on solving state management across projections, context detection without uncanny-valley UI shifts, and rule complexity that doesn't explode combinatorially. Without those, "derive the interface from intent" is responsive design with more steps and worse tooling.

The petri dish metaphor (capabilities exist before you observe them) is evocative but dodges a genuine question: are capabilities truly independent of their interfaces, or are they co-constructed? A capability you can't access isn't a capability — it's a wish. The test suite is whether any nontrivial system has actually been built this way. Marc doesn't cite one.

The Claude Opus 4.5 writing credit is refreshingly transparent and gives the piece an "early adopter figuring it out" authenticity. The frameworks (the eight-intent diagram, the six-dimension illustration, the pipeline graphic) were made manually — the AI shaped prose, not concepts. This is a good model for AI-human collaboration: the human brings the mental models, the AI brings the speed.

Where the essay is strongest is in diagnosing the cost of the current approach: "designing 45 interfaces per feature by hand is already impossible. We just pretend otherwise." Every team shipping to web, iOS, Android, watch, voice, and API already knows this. Marc gives them language for why it feels broken and a direction — if not a solution — for what comes next.

The final vision — intent zones on a canvas, hover to see projections across surfaces — is genuinely compelling as a design-tool aspiration. It's the design equivalent of [[The Dark Factory is a DOT File]]: one intent model, rendered to infinite surfaces. But the gap between vision and tool is the whole game, and Marc hasn't closed it.

---

## Cross-Links

- [[Experience Design for Agents]] — "human owns intent/judgment" is the same layer; Marc argues the interface below it should be derived
- [[The Plan Is the Program]] — same collapse of intent and execution, applied to interaction design rather than code generation
- [[Smart Models Dumb Pipes]] — separation of judgment from execution; intent-driven design applies the same principle to UI
- [[Specifications as the Product]] — specs encode intent; code is a disposable rendering. Marc extends this to interfaces
- [[Building Production-Ready Voice Agents]] — voice as a projection surface without a viewport
- [[json-render]] — generative UI from constrained JSON; an implementation of "derive the interface from intent"
- [[From AI Studio to AI Forge]] — "human changes altitude" maps to Marc's inverted initiation model
- [[10 Principles for Agent-Native CLIs]] — designing for agents first; Marc's intent-first design is the UI equivalent
- [[Two Kinds of User Are Emerging]] — the interface as negotiation surface favors power users who can steer agents
- [[Elements of Agentic Systems Design]] — ten-element taxonomy; Marc's eight intents are a candidate interaction-level complement
- [[The Dark Factory is a DOT File]] — one pipeline spec, many runners; one intent model, many projection surfaces

---
*Sources: [[summary/intent-is-the-interface]]*
*Last updated: 2026-05-15*
