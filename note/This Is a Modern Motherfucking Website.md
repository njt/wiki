# This Is a Modern Motherfucking Website

Robin Reel's 2026 sequel to the 2013 "motherfucking website" screed: a profanity-driven manifesto that the web platform already gives you everything developers build toolchains to get — accessibility, SEO, social cards, performance, dark mode — and that the meta-framework stack (1,100 dependencies, hydration, CI pipelines for a blog) is a solution to problems that were solved by typing tags into a file.

---

## Key Quotes

> "You nodded, laughed, shared it, and then went right back to work and made it **so much fucking worse**."

The sharpest diagnosis of the original screed's failure: satire without a decision change is just entertainment. Everyone agreed with the 2013 piece and the median website got heavier anyway. This sequel exists because the argument has to be made against the current stack, not the old one.

> "There are three major libraries that already worked out the kinks in all of this, they're called Gecko, Blink, and WebKit."

The best line in the piece. The platform is the framework that never needs `npm install`, and its "ecosystem" has been debugging since the 1990s. Reel's point about `<details>` — keyboard accessible, screen-reader friendly, find-in-page searchable, and functional in twenty years without reconstructing a build environment — is a durability argument that mirrors [[The Lindy Effect in Software]]: survival implies fitness.

> "Your client‐rendered SPA injects them with JavaScript after the crawler has already fucked off, which is why you then had to bolt on server‐side rendering to get back what **just fucking writing HTML** already gave you."

This is the article's actual technical thesis, hiding inside the joke: the industry's multi-year detour through client rendering and then SSR/RSC/hydration was a round trip back to the 1993 baseline. The framework didn't add capability; it removed capability and then sold it back.

> "Start with HTML. Add what you need. Stop when you're done. That's the whole fucking methodology."

Note what's absent: this is not minimalism for its own sake. Reel explicitly concedes that real applications (spreadsheets, video editors) deserve frameworks, and that 500 pages with a shared header is a real problem that a static site generator answers proportionately. The rule is proportionality, not purity — the same constraint-seeking instinct as [[State-Oriented Consistency]]: ask what each thing actually requires instead of defaulting to one answer.

## Key Themes

#concept #tool #pattern #comparison

- **The platform as the forgotten framework**: every listed "modern feature" — semantic landmarks, JSON-LD, Open Graph, speculation rules, `<picture>`, container queries, dark mode via media query — is a browser primitive already downloaded by the user before they ever visited your site. At best the framework "gives you a slower way to type them, and at worst it hides them behind a plugin ecosystem so you forget they were free."
- **Proportionality as methodology**: the concession paragraphs are what elevate this above a pure rant. Documents get HTML; applications get frameworks. The sin is category confusion, not tools.
- **Ephemeral complexity**: a blog with a loading spinner and a lockfile longer than the page is complexity nobody chose — it "fell out of node_modules." The closing test is legibility: view source and the artifact should be "readable top to bottom by a human being."

## Critical Analysis

The obvious weakness is that Reel strawmans the other side — the 23 KB page itself uses an image pipeline (`avifenc`, `cwebp`), which is a build step with a euphemism. And "developer experience doesn't matter" is bluster: DX is why the framework ecosystem exists, and telling people nobody visiting cares is how you lose arguments with teams. But the strongest form of the argument survives the strawman: the burden of proof should sit on the toolchain, and for the document-shaped 90% of the web it plainly fails that test. The piece is also accidentally relevant to the AI-coding era — agents generating HTML-first sites need no framework plumbing to produce correct results, and the aesthetic here (one file, no dependencies, deploy by copy) is the same instinct behind [[The GUS Stack — Go, Unix, SQLite]]: pick boring, legible, stable components so that both humans and agents can reason about everything you built. Where it connects to satire rather than engineering, [[Performative UI]] is the React-side twin: both catalogue the absurdity of the modern frontend stack, but Smith files it as an npm package while Reel files it as a single HTML file.

---
*Sources: [[raw/modernmotherfuckingwebsite-dreamstation-systems]], [[summary/modernmotherfuckingwebsite-dreamstation-systems]]*
*Last updated: 2026-09-29*
