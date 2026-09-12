# uiui — Dense Console UI Kit

uiui is a one-CSS-file design system for dense operations UIs — the network dashboards, device managers and admin panels with many rows and a handful of colored numbers — modeled on Ubiquiti's Unifi console. It ships light/dark, every class prefixed `ui-`, no build step, and it doubles as both a component catalog and an instruction set for coding agents.

---

## Key Quotes

> "A dense console UI kit inspired by the Unifi console. One CSS file, every widget on this page, light and dark."

The entire product pitch in a sentence. The "every widget on this page" move is the honest one — this is dogfooding, not marketing. The landing page is the test suite, and if a widget didn't render it would be immediately visible.

> "The words this site uses for its parts, so people and agents mean the same thing."

This is the sentence that marks uiui as a 2026 artifact rather than a 2016 one. A glossary is normal; a glossary that names its audience as "people **and agents**" is not. The whole page treats the coding agent as a first-class consumer of the design system, with a `skill.md` to read and a vocabulary to agree on before anyone writes a line.

> "Density is the point. 31px rows, 13px text, hairlines only. No zebra striping, no card per row."

The anti-SaaS aesthetic as a philosophy. Modern web UI has drifted toward whitespace and rounded cards; uiui argues that a console full of devices needs the opposite — information per pixel, with color reserved for the exceptions ("values are text-2 grey; only the exception gets a color").

> "Blue means 'you can act here'... Never use blue as decoration or as a data color except for download."

A disciplined affordance rule: blue is exclusively interactive (links, active tabs, checked boxes, the selected row edge, the sorted header). This is what separates a *system* from a *theme* — a theme recolors, a system constrains what each color is allowed to mean.

## Key Themes

**#tool — A design system as a single CSS file.** No npm package, no bundler, no framework. `<link rel="stylesheet" href="uiui.css">` and a `<body class="ui">`. The contrast with Tailwind's build step and shadcn/ui's component-by-component copy is the whole point: density UIs are niche enough that a hand-rolled vocabulary of ~20 prefixed classes beats a utility framework.

**#concept — Color as semantics, not decoration.** Four graphite surfaces carry hierarchy through lightness steps alone ("depth comes from the step between them, not from shadows or borders"); blue is reserved for action; green/red/yellow/orange are status, never row tints ("status is small — a 6px dot, never a tinted row").

**#pattern — The design system as agent-readable instruction set.** Beyond the stylesheet, uiui ships a `skill.md` and a glossary explicitly "so people and agents mean the same thing." The "For agents" section is a starting-shell recipe: wrap in `.ui`, compose from the classes, use the lucide sprite, invent device names and 192.168.0.x addresses, pick an example page as the shell. Design taste, compressed into instructions a model can follow.

**#tool — Unifi console as the reference aesthetic.** The choice of Ubiquiti's Unifi console as the model is quietly smart: it's the one "operations UI" a huge fraction of homelab and IT people already recognize, so the design language borrows trust instead of building it from scratch.

## Critical Analysis

**The right scope, picked honestly.** Most UI kits aim at marketing sites or SaaS dashboards; uiui aims at the dense-operations niche and refuses to generalise. That's the correct instinct for a solo-authored CSS file — a kit that tries to serve everyone becomes Tailwind, and Tailwind already exists. The design vocabulary (port grid, zone matrix, timeline rail, spectrum chart) is so specific to network management that it reads as "extracted from a real product" rather than "invented for a demo," which is exactly the credibility a design system needs.

**The agent angle is the interesting part and it's thin.** The glossary line — "so people and agents mean the same thing" — and the `skill.md` pointer are the most forward-looking ideas here, and they get the least space. The pattern of encoding taste as machine-readable rules is the same one [[Writing Style Guides for Better UIs]] extracts from IBM Carbon, but uiui has it pre-baked rather than argued: a named vocabulary (shell, rail, drawer, chip, meter, bucket) that a model can be told to use *consistently* instead of improvising. Whether the `skill.md` actually holds up under a real coding agent is unproven from this page alone — it's promised, not demonstrated.

**The six principles are the real substance.** "Blue means you can act here," "status is small," "destructive is a link," "three surfaces, nothing else" — these are not aesthetic preferences, they're *rules that fail loudly when broken*. That's what makes the system teachable: it's the difference between "use these colors" (a theme) and "a red-filled button is always wrong" (a system). The closest sibling in this wiki, [[smui]], proves the theme side of the spectrum — it recolors shadcn/ui toward a terminal aesthetic — but it stops at "looks like a spacecraft bridge," while uiui goes the next step to "and here are the constraints that keep it coherent."

**The Unifi homage is a double-edged choice.** Basing the kit so directly on Ubiquiti's console means it inherits a proven, widely-recognized information design for free — but it also means the kit's ceiling is "as good as Unifi's console," and the interesting design problems (what does a dense UI look like that *isn't* a network dashboard?) go unasked. That's fine for v1; it's the boundary to watch if the project grows.

## Related

- [[smui]] — the closest sibling: another shadcn/ui-adjacent aesthetic, but a *theme* (a Nord palette and sharp edges) where uiui is a *system* (semantic color rules that fail loudly). uiui shows the "single CSS file, defined shadcn variables" move generalized past shadcn to a framework-free stylesheet plus an agent skill.
- [[Writing Style Guides for Better UIs]] — Langworth feeds IBM Carbon's MDX guidelines to an agent and extracts them into a reusable skill; uiui ships that exact pattern pre-baked, with a glossary and `skill.md` written "so people and agents mean the same thing." Strengthens the claim that design systems are becoming machine-readable instruction sets, not just component libraries.
- [[Make Pages Interactive]] — HTML as the universal agent output format; uiui supplies the starting shell for the densest class of that output (admin/ops consoles) with "pick one of the example pages as the starting shell" as the onboarding instruction. Nuances it: for data-dense screens, an agent needs a *vocabulary*, not just a render loop.
- [[Lovelace]] — its design-system section commits to tokens and "no borders, one electric signal" — the same token-first discipline as uiui's "three surfaces, nothing else" — but as an app-private system rather than a consumable. uiui is that discipline extracted and made portable.

---
*Sources: [[raw/uiui-statico-io]], [[summary/uiui-statico-io]]*
*Last updated: 2026-09-13*
