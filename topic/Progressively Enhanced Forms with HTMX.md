# Progressively Enhanced Forms with HTMX

Rafa's field report on building a bookmark-editing form with HTMX and progressive enhancement: a catalog of HTML-native transient-state techniques, a build-without-JS-first workflow, and honest notes on where HTMX's abstraction leaks. A practitioner's companion to the pattern-level advice in [[How I Use HTMX with Go]] and the stack-level argument in [[The GUS Stack — Go, Unix, SQLite]].

---

## Key Quotes

> "Interfaces that work without JS are fundamentally different from single page applications: updating the UI in response to user interactions will often require a roundtrip to the server."

The article's thesis stated plainly. This isn't a performance argument or an ideological one — it's architectural. When every interaction is a roundtrip, transient state becomes the central design problem. Rafa's contribution is cataloging which HTML mechanisms carry which kinds of state across which roundtrips, and what degrades gracefully when JavaScript is absent.

> "In an SPA, transient state would live in JS memory, and you'd use some library with an unholy amount of dependencies to manage it. Since we're making do without JS, we'll have to use classic HTML features to manage this state, each with its own set of tradeoffs."

The "unholy amount of dependencies" aside is a value judgment, but the structural claim is correct: HTML gives you three state-carrying mechanisms (form values, query parameters, URL paths), each with different persistence characteristics. Form values disappear on reload; query parameters survive reloads and bookmarks; URL paths encode state positionally. The skill is picking the right mechanism for each piece of state — the same judgment call a React developer makes when choosing between `useState`, URL params, and context, but with fewer and simpler options.

> "I find it easiest to write a whole feature without htmx first. This makes sure I don't architect myself into a corner where I need to refactor later on when I find out something is not possible without JS."

This build-without-JS-first workflow is the article's most actionable advice. It's a constraint that prevents overreach: if you start with HTMX, you can accidentally build interactions that depend on JavaScript. If you start without it, every interaction must work with plain forms and links, and HTMX becomes a strict upgrade. This is the inverse of the typical SPA development pattern, where JavaScript is the default and server rendering is bolted on later. It's also the workflow that makes [[The GUS Stack — Go, Unix, SQLite]]'s "HTMX as much as possible" prescription actually implementable — you can't sprinkle HTMX on top if you don't have a working no-JS baseline.

> "I'm still figuring out the right way to scope htmx swaps. I've often found bugs because a too narrow part of the page was updated, leaving other parts of the page showing stale data."

The article's most honest passage. HTMX's `hx-target` attribute lets you swap only part of the page, which is efficient but creates a cache-coherence problem: other page regions that depended on the changed data are now stale. Out-of-band swaps exist but feel "error prone and overly hard to maintain." Rafa defaults to full-page swaps as the least-buggy option. This is the same judgment [[How I Use HTMX with Go]]'s Alex Edwards makes about `HX-Redirect` vs `HX-Location` — accept the full-page reload rather than navigate the complexity cliff. It's also the fundamental tension that [[Phoenix LiveView]] solves differently: LiveView pushes fine-grained HTML diffs over a persistent WebSocket, so every visible region is always consistent by construction. HTMX's request-response model can't make that guarantee without the developer manually tracking which regions depend on which state.

> "Resilient Web Design is an incredibly well-written book about designing robust websites. It's more about design in the holistic, rather than the visual sense, and has influenced lots of my technological choices."

Jeremy Keith's book as the philosophical foundation. Keith argues that the web's strength is its layered design: HTML (structure), CSS (presentation), JavaScript (behavior), each optional and independently degradable. Rafa's approach is Keith's philosophy operationalized: build the HTML/HTTP layer first, verify it works standalone, then add the JavaScript/HTMX layer as enhancement. The book recommendation is doing double duty here — it's both a citation and a statement of values.

---

## Key Themes

#htmx #progressive-enhancement #forms #web-development #server-side-rendering #pattern

- **Transient state as the central design problem**: When every interaction is a server roundtrip, the question "where does unpersisted user input live?" becomes architectural rather than incidental. Rafa maps three HTML-native answers — form values (simplest, lost on reload), query parameters/paths (survive reloads and bookmarking, good for search), and submit button `formaction` (changes the submit target per-button). Each has different persistence characteristics and different graceful-degradation behavior. This is the taxonomy that SPA developers never need because JS memory absorbs everything.

- **Build-without-JS-first as architectural discipline**: Writing the feature without HTMX first forces every interaction through plain HTML forms and links. This constraint prevents overreach — you can't accidentally design an interaction that only works with JavaScript because you never designed it with JavaScript at all. HTMX then becomes a strict upgrade (active search instead of submit-button search, spinner instead of blank loading state) rather than a dependency. The workflow inverts the SPA default and produces a more robust artifact as a side effect.

- **The `hx-target` coherence problem**: HTMX's partial-swap model creates a cache-coherence challenge: if region A's state depends on region B's data, and a swap only updates B, A shows stale data. Out-of-band swaps are the intended solution but feel fragile in practice. Rafa's default to full-page swaps is a judgment call that simplicity beats precision — the same tradeoff that appears throughout the HTMX ecosystem. [[How I Use HTMX with Go]] makes the same call on `HX-Location` vs `HX-Redirect`. [[Phoenix LiveView]] avoids the problem entirely by diffing from a single server-side state, but at the cost of requiring a persistent WebSocket connection.

- **`formaction` as the undiscovered primitive**: Rafa admits they only learned about submit buttons' `formaction` attribute while writing the post. It changes the form's action URL only when that specific button is pressed — solving the "which button was clicked?" problem without JavaScript. This is the kind of HTML feature that's been in the spec for years but is invisible to developers who reach for JavaScript first. The article's value is partly in surfacing these forgotten primitives.

- **Enter-key behavior as form-design constraint**: Rafa split their bookmark-editing form into two separate forms because pressing Enter in the list-search input would otherwise trigger rename — the browser submits the first submit button of the associated form. This is a constraint that SPA developers never encounter (they intercept the Enter key in JavaScript) but that shapes the HTML structure of progressively enhanced forms. It's a concrete example of how "works without JS" forces different architectural decisions.

- **The reading list as canon**: Rafa's recommended resources form a coherent canon for the progressive-enhancement approach: Petros (minimal HTMX), Keith (web design philosophy), and _Plain Vanilla Web_ (modern browser capabilities). This is a different canon from the React/Next.js/TypeScript ecosystem, and deliberately so. It's the reading list of someone who chose a different technical tradition.

---

## Critical Analysis

**The article is a field report, not a framework.** Rafa isn't proposing a new pattern language or abstraction. They're describing what worked on one real feature and cataloging the techniques and tradeoffs discovered along the way. This makes the article more useful than a theoretical treatment — it's grounded in a specific form with specific requirements — but also means the techniques don't all generalize. The two-form split (rename form + list-assignment form) is driven by Enter-key behavior, which is specific to having multiple submit actions on one page. Not every form needs this.

**The transient-state taxonomy is incomplete.** Rafa covers form values, query parameters, and URL paths. Missing from the taxonomy: cookies (survive everything, limited size, sent on every request), `localStorage` (requires JS, survives everything), hidden form fields (same properties as form values but not user-visible), and the `autocomplete` attribute (browser-managed form memory, works without JS). This isn't a criticism — the article covers what the specific feature needed — but it's worth noting that the full transient-state design space is larger than what's presented.

**The "unholy amount of dependencies" line is a tell.** It signals that Rafa is writing from a position of SPA fatigue, not neutral evaluation. The claim is directionally true — the average React project does pull in an absurd dependency tree for state management — but it's a values statement, not an engineering argument. Someone who prefers Redux or Zustand would point out that their dependency tree is large but their state management is predictable, debuggable, and well-understood by a large hiring pool. The tradeoff is real in both directions.

**The article's relationship to [[How I Use HTMX with Go]] is complementary, not redundant.** Edwards provides the Go-specific implementation patterns (template architecture, `htmlRenderer`, dual-mode handlers). Rafa provides the framework-agnostic interaction-design patterns (transient-state techniques, progressive-enhancement workflow, `hx-target` scope decisions). Read together, they cover the full stack of HTMX development: Edwards tells you how to structure your Go backend for HTMX; Rafa tells you how to structure your HTML forms for progressive enhancement. Neither duplicates the other.

**The `hx-target` coherence problem is the HTMX ecosystem's biggest unresolved tension.** Rafa defaults to full-page swaps because partial swaps create stale-data bugs. Edwards uses partial swaps carefully, with explicit `Vary: HX-Request` headers to keep caches correct. Neither approach fully solves the problem. [[Phoenix LiveView]] solves it architecturally (single source of truth, pushed diffs) at the cost of WebSocket complexity. The fact that HTMX's core maintainers haven't proposed a general solution suggests the problem may be inherent to the request-response model — partial updates will always risk inconsistency unless the developer manually tracks data dependencies across page regions. This is the complexity that SPAs solve with client-side state management, and it's the complexity that HTMX projects encounter once they grow beyond simple form interactions.

**The progressive-enhancement workflow is the article's most exportable idea.** "Build it without JS first, then add HTMX" is concrete, testable, and doesn't depend on any particular backend stack. It works with Go (Edwards), with Python/Django, with Rails, with anything that serves HTML. It's also the workflow that makes HTMX projects testable — you can verify the no-JS path with standard HTTP integration tests, then add HTMX-specific tests for the enhanced interactions. This is the testing strategy that Edwards' article omits and that Rafa's workflow implies.

**The real insight is that progressive enhancement is a design constraint, not a feature checklist.** Rafa isn't aiming for "works without JS" as a bullet point. They're using it as a design tool: if you can't make an interaction work with plain forms and links, you either find a different interaction design or you accept that this particular interaction is JS-dependent. The constraint forces better architecture not because JS-free is inherently virtuous, but because every JS-dependent interaction is a future maintenance burden, an accessibility barrier, and a point of failure. HTMX changes the economics by making the JS-dependent interactions thinner and the JS-free baseline richer.

**The build-without-JS-first workflow is Fielding's Code-On-Demand constraint as practice.** [[Components of a Hypermedia System]] documents that scripting is a legitimate, optional part of REST — provided it augments rather than replaces the hypermedia model. Rafa's discipline (build it with plain forms first, then add HTMX) is exactly that boundary: HTML/HTTP as the self-standing core, JavaScript as the enhancement layer. The workflow doesn't just produce robust applications; it produces *RESTful* ones in Fielding's original architectural sense.

---

*Sources: [[raw/progressive-enhanced-forms-htmx]], [[summary/progressive-enhanced-forms-htmx]]*
*Last updated: 2026-08-07*
