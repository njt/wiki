# Patterns.dev

The definitive modern reference for web application design, rendering, and performance patterns. Co-created by Lydia Hallie and Addy Osmani, patterns.dev extends the classic GoF pattern catalogue into the framework era — covering vanilla JS, React/Next.js, and Vue.js — and adds two dimensions the original patterns never addressed: rendering strategy and performance optimization at scale.

---

## Key Quotes

> "Design patterns are descriptive, not prescriptive" — they guide developers facing common problems but aren't meant to be forced into every scenario.

This is the site's thesis statement. It's a catalog for awareness, not a checklist. The distinction between *recognizing* a pattern and *applying* one is what separates pattern literacy from cargo-cult architecture.

> "We publish patterns, tips and tricks for improving how you architect apps for free."

The free-and-open framing matters because most pattern references are gated behind Manning/O'Reilly paywalls. CC BY-NC 4.0 license means these patterns can be remixed and built upon — unusual generosity for content this polished.

## Key Themes

#pattern #design-pattern #rendering #performance #web-development #React #Vue #JavaScript #reference

**Architecture patterns meet runtime reality.** Patterns.dev's innovation is bridging design patterns (how you structure code) with rendering patterns (how that code reaches users) and performance patterns (how fast it feels). The GoF never had to think about bundle splitting or hydration.

**Framework-agnostic, framework-aware.** The site covers vanilla JS patterns first, then shows React and Vue adaptations. This ordering is deliberate: it teaches that patterns transcend frameworks, even as implementation details differ.

**Interactive by default.** Every pattern links to a live CodeSandbox. The animated explanations (particularly for progressive hydration and streaming SSR) make concepts that are normally hand-wavy concrete. This is a reference built for the web, not a printed book ported to the web.

## Critical Analysis

Patterns.dev is the best free resource for modern web patterns, period. Its triangular structure (design + rendering + performance) captures what's actually hard about building web apps in a way that pure pattern catalogues miss.

The tension is that LLMs now encode all of these patterns implicitly. Claude and Copilot generate singleton wrappers, HOC compositions, and bundle-splitting strategies without being told to. This makes pattern literacy simultaneously more valuable (you need to *recognize* what the AI generated and whether it's appropriate) and less obvious (memorizing implementation details is wasted effort). The site's "catalog for awareness, not a checklist" framing is precisely right for the AI era — you don't need to remember how to implement the Proxy pattern, but you do need to know when one is being used on your behalf.

The Vue section is notably thinner than React, reflecting market reality rather than editorial bias. The JavaScript patterns section is the most timeless — singletons, observers, and factories don't care about framework churn.

What's missing: patterns for error handling, patterns for state machines, patterns for offline-first architecture. Also absent is any treatment of patterns as anti-patterns in specific contexts — the site describes *what* each pattern does but rarely says *when not to use it*. This is the weakness of the catalog approach: it tells you the shape of the hammer, not when you're holding a screw.

Pattern thinking isn't confined to web and application code. Neil Brown's [[Linux Kernel Design Patterns]] series (2009) extracted ten patterns from kernel source — reference counting strategies, data structure embedding, and the "midlayer mistake" anti-pattern — using the same problem→solution→consequences structure. It's a reminder that the most interesting pattern catalogs may live at levels of the stack where correctness constraints are harder and the cost of getting the abstraction wrong is higher.

Co-creator Addy Osmani's [[Addy Osmani's Workflow]] for AI-assisted development sits in productive tension with patterns.dev: his workflow argues classical engineering practices become *more* critical with AI; patterns.dev supplies the pattern vocabulary those practices need. [[Awesome Agentic Patterns]] is patterns.dev's spiritual sibling for the agent domain — same catalog structure, different technology layer. [[The Claude C Compiler]] (Lattner's finding that AI implements known patterns well but invents nothing new) validates the entire premise: patterns are the durable unit of software knowledge, and AI is pattern recombination, not invention.

[[Software Engineering Craft]] provides the broader context. [[Elements of Code]] and [[Simplicity in the Age of AI-Assisted]] offer complementary perspectives on what makes code comprehensible and maintainable beyond pattern selection.

---
*Sources: [[summary/patterns-dev]]*
*Last updated: 2026-05-14*
