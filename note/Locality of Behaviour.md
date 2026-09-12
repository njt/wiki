# Locality of Behaviour

Carson Gross's formulation of a software design principle that operationalizes Richard Gabriel's insight about locality: make the behaviour of a code unit obvious on inspection, and treat the distance between behaviour-definition and behaviour-effect as the primary axis of maintainability. The essay is both a philosophical justification for htmx's attribute-on-element approach and a general-purpose lens for evaluating framework design.

---

## Key Quotes

> "The behaviour of a unit of code should be as obvious as possible by looking only at that unit of code."

The principle stated as a prescription. Gross is doing something specific here: taking Gabriel's descriptive observation ("locality is the primary feature for easy maintenance") and converting it to an imperative ("strive to make behaviour obvious on inspection"). The "as possible" qualifier and "in balance with other concerns" caveat are doing real work — this isn't an absolutist principle, it's a default posture that yields to context. The strength of the formulation is that it's falsifiable: given two implementations of the same behaviour, you can argue about which has better locality without agreeing on everything else.

> "This 'spooky action at a distance' is a source of maintenance issues and stands in the way of developers understanding of the code base."

The Einstein-borrowed phrase is the essay's most memorable contribution. "Spooky action at a distance" names the feeling every developer has experienced — changing a CSS class and breaking a JavaScript behaviour defined three files away, or modifying a shared utility and discovering six downstream effects. Gross doesn't argue that all spooky action is avoidable; he argues that it's a cost to be minimized, and that framework design choices determine the baseline cost everyone pays.

> "It is important to make the distinction between inlining the *implementation* of some behaviour and inlining the invocation (or declaration) of some behaviour."

The essay's sharpest conceptual move. Critics of LoB often conflate "putting behaviour near the element it affects" with "copy-pasting implementation details everywhere." Gross separates them: `<button hx-get="/clicked">` declares *that* a GET request happens without specifying *how* HTTP requests work. The framework handles the implementation; the attribute handles the declaration. This is the same distinction programming languages make between function declaration and function call — and Gross is essentially arguing that HTML attributes deserve the same courtesy.

> "The further behaviour gets from the code unit it effects, the more severe the violation of LoB."

The distance-as-severity heuristic is the essay's most practical contribution. A behaviour defined on a parent element three lines away is a mild violation; a behaviour defined in a separate file is a severe one. This gives developers a gradient to reason about rather than a binary — you can trade off LoB against DRY incrementally rather than having to choose one principle and abandon the other. It also implies that framework design should default to near and let developers push things further as needed, rather than defaulting to far and requiring effort to bring things close.

---

## Key Themes

#concept #software-design #htmx #maintainability #pattern

- **LoB as a framework design constraint, not just a coding style**: Gross argues that framework authors bear special responsibility for LoB. A framework that makes behaviour obvious-on-inspection easy is fundamentally different from one that makes it possible-but-awkward. htmx's design — attributes on elements — is LoB-native; React's design — behaviour in JSX that renders to DOM — is LoB-adjacent; jQuery's design — selectors in separate files — is LoB-hostile. The framework sets the ceiling on how local behaviour can be.

- **The DRY-LoB tension as the central tradeoff**: The essay's treatment of DRY is notably even-handed. Gross doesn't argue DRY is wrong — he argues it competes with LoB, and developers must choose per-situation. htmx's own attribute inheritance (placing `hx-target` on a parent so children don't repeat it) is explicitly a DRY-over-LoB choice. The honesty about his own framework's compromise is what makes the argument credible. Compare with [[The Economic Benefit of Refactoring]], where Fowler's controlled experiment shows that semantic decomposition — breaking a monolith into named modules — reduced agent token consumption 83%. That's LoB at the architectural level: each module's behaviour is obvious because it's scoped to a single concern.

- **Separation of Concerns as the established principle LoB pushes against**: Gross treats SoC as the incumbent — it's the default in most codebases, and LoB is the challenger. The observation that inline styles have become more prevalent is offered as evidence that SoC is already losing. But Gross is careful: this isn't a call to inline everything, it's a call to question whether the separation is worth the spooky-action cost. [[Progressively Enhanced Forms with HTMX]] lives this tension: Rafa's build-without-JS-first workflow is LoB-aligned (behaviour visible on the form element), but the `hx-target` coherence problem — partial swaps creating stale data in other page regions — is the LoB violation that SoC was supposed to prevent.

- **LoB and the agentic coding era**: Gross wrote this essay before coding agents became mainstream, but LoB has become more relevant, not less. Agents read code linearly; they don't build the same mental models humans do. An htmx button with `hx-get` is legible to an agent in a single glance; a jQuery button whose behaviour is defined in `app.js` line 847 requires the agent to trace selectors across files. [[AI Slop Starts with the Codebase Itself]] argues that codebases speaking a dialect models already know are productivity multipliers — LoB adds that *how* you encode behaviour determines whether an agent can reason about it at all. [[How I Use HTMX with Go]]'s Edwards makes the same point implicitly: his `htmlRenderer` abstraction is a small, composable type whose behaviour is obvious from its API surface.

- **The Gabriel lineage**: Gross credits Richard Gabriel's observation about locality but doesn't explore the intellectual history. Gabriel was writing about Lisp and object-oriented design in the 1990s; his point was that understanding a system requires tracing control flow, and locality minimizes the tracing distance. Gross generalizes this from Lisp to HTML+JS and adds the prescriptive layer. The essay sits in a tradition that includes Parnas on information hiding (what to encapsulate) and Gabriel on locality (where to put things). Gross's contribution is applying both questions to the frontend stack and answering: encapsulate the implementation, localize the declaration.

---

## Critical Analysis

The essay's strength is its clarity and honesty. Gross doesn't pretend LoB is the only principle that matters — he explicitly names the tradeoffs with DRY and SoC, and he acknowledges that htmx itself violates LoB in service of DRY (attribute inheritance). This makes the argument harder to dismiss than a zealot's manifesto.

The essay's limitation is its scope. Gross treats LoB primarily as a property of individual code units (a button, a function) but doesn't explore what LoB means at higher levels of abstraction. What does LoB look like for a module? A service? A deployment pipeline? The distance-as-severity heuristic suggests an answer — behaviour far from effect is worse — but doesn't help with the harder question of what *counts* as a code unit when the units are architectural rather than syntactic. [[Vertical Slice Architecture]] is another answer at the feature level: organize code by use case and colocate the slice's data access with its handler, with the A-Frame split (business logic / infrastructure / coordination) doing the work of keeping that colocation from re-coupling business logic to infrastructure. [[The Economic Benefit of Refactoring]] provides one answer: a module whose name accurately describes its contents has good architectural LoB because an agent (or human) can infer behaviour from the name alone.

The essay is also silent on the tooling question. Gross's examples assume a developer reading raw source code, but most developers use IDEs with go-to-definition, find-references, and call-hierarchy views. These tools reduce the cost of spooky action — you can click through from `$("#d1")` to the behaviour definition in a second. The counterargument is that tooling mitigates but doesn't eliminate the cost: you still have to know to look, and every additional context-switch is a cognitive interruption. The rise of coding agents strengthens this counterargument — agents don't benefit from IDE tooling the way humans do, making source-level locality more important, not less.

The inline-styles-as-evidence argument is the essay's weakest point. Gross mentions that inline styles have "become more prevalent as of late" as evidence that SoC is losing support, but this conflates several distinct trends: Tailwind's utility-class approach (which is inline-adjacent but not inline), CSS-in-JS (which colocalizes styles with components but still separates concerns at the component boundary), and actual inline styles in HTML attributes (still rare in production). These are different answers to the same question — how close should style definition be to the styled element? — and collapsing them into "inline styles" obscures more than it reveals.

Despite these limitations, the essay has real practical value. The distance-as-severity heuristic alone is worth the read: it gives teams a shared vocabulary for arguing about where to put behaviour without defaulting to whichever principle (DRY, SoC, LoB) the most senior person prefers. In a world where coding agents are generating more code than humans write, making behaviour obvious on inspection isn't just a maintenance nicety — it's the difference between code an agent can modify safely and code that becomes a haunted house no one understands.

---
*Sources: [[raw/locality-of-behaviour]], [[summary/locality-of-behaviour]]*
*Last updated: 2026-08-08*
