# Lessons for Reusable Web Components

Daniel De Pietro's field notes from dropping a custom code-sandbox web component into multiple projects, distilled into five practical rules for any reusable UI component. The core insight: the platform is now good enough that you don't need a build step — namespace discipline, CSS custom properties as the public API, and modern CSS features handle most of what frameworks used to provide.

---

## Key Quotes

> "Every class and custom property in the component now starts with `csb`. Short, because it repeats on every element and every variable, and being short might shave a few bytes and a few minutes of typing. Distinctive, because the point of a component is to land in a page you don't control, and a generic `.header` or `.button` might collide with the host's styles."

Namespacing is the lowest-cost isolation primitive. The web has no module system for CSS classes — your prefix IS your sandbox. De Pietro gets the economics right: brevity isn't aesthetics, it compounds across every class, variable, and keystroke.

> "Rather than ship override classes, every value worth changing is a CSS custom property, with its default written in as the fallback."

This is the article's thesis statement. Override classes are a second API surface you have to document, maintain, and debug. CSS custom properties with fallbacks collapse theming into one mechanism: set a variable anywhere in the cascade, the fallback catches everything else. No `!important`, no specificity wars.

> "A reusable component lands in contexts you can't predict, so using the modern features that the platform offers is one of the best way to achieve adaptability and resilience."

Container queries over media queries. `light-dark()` over manual theme toggles. System fonts over custom stacks. Logical properties over physical ones. `em` over `px`. The checklist reads like a platform-feature bingo card because it is — each item eliminates a class of bugs that framework abstractions paper over.

> "A module only counts as reusable once someone other than you can actually use it and customize it."

The real definition of reusable: not that you *can* reuse it, but that someone else *does*. Documentation is what makes the component a product rather than a personal shortcut.

## Key Themes

- **#pattern** — CSS custom properties as the public API surface: one mechanism for defaults, theming, and customization. The variable list IS the contract.
- **#concept** — Namespacing as isolation: when CSS has no module system, a short distinctive prefix is the cheapest way to prevent collisions.
- **#tool** — NPM as distribution channel for no-build components: install with one command, import from CDN, update with version bumps instead of copy-paste archaeology.
- **#concept** — Platform-first resilience: preferring built-in CSS/JS features (container queries, `light-dark()`, system fonts, logical properties, `em`) over framework abstractions that assume a known context.
- **#pattern** — Publish over paste: the effort of copy-paste maintenance across projects exceeds the effort of packaging and publishing once.

## Critical Analysis

De Pietro's advice is so practical it's almost invisible — which is exactly the point. None of these lessons are novel in isolation; their value is in the synthesis and the underlying philosophy: **trust the platform, minimize the API surface, make the defaults work everywhere.**

The most provocative claim is the one he doesn't state explicitly: **you might not need a build step at all.** The component ships as a single JS file imported via `<script type="module">` from a CDN. No bundler, no tree-shaking, no configuration. Modern browsers can handle ES modules, CSS custom properties, container queries, and `light-dark()` natively. The build step that was mandatory in 2018 is optional in 2026 — if you're disciplined about your platform feature baseline.

This dovetails with a broader trend in the wiki: [[Smart Models Dumb Pipes]] argues for putting intelligence at the endpoints and keeping infrastructure simple; De Pietro applies the same logic to component design. The platform (browser) is the smart endpoint; your component should be a dumb pipe that inherits its environment.

The article's humility is its strength. It doesn't claim to be a framework or a methodology — it's a developer writing down what worked. The note upfront ("this doesn't touch advanced web-components techniques like shadow DOM or `<slot>`") is honest scoping. He's not building Web Components™; he's building components for the web. The distinction matters.

**What's missing:** No discussion of testing strategy for reusable components across contexts. No mention of versioning policy — he says "keep these [variable names] stable between versions" but doesn't address semver, breaking changes, or migration guides. The article describes the happy path of a solo maintainer; the advice scales less well to team-maintained component libraries.

**The implicit bet:** ES modules, CSS custom properties, container queries, and `light-dark()` have crossed the adoption threshold where you can treat them as universally available. In 2026, that bet is reasonable. In 2023, it would have been aspirational. The question is whether the next wave of platform features (CSS `@scope`, `@layer`, view transitions) follows the same adoption curve or fragments further.

---

*Sources: [[raw/lessons-reusable-web-components]]*
*Last updated: 2026-06-09*
