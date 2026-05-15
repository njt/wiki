# How HTML Changes in ePub

Robin Whittleton's guide to the surprising ways ePub diverges from standard web development. ePub uses HTML technology, but it's XHTML (XML-based), not HTML5, and the CSS support in e-readers is "positively historic."

---

## Key Quotes

> "XHTML didn't work out for web browsers due to fragility and performance issues, yet persists as the ePub standard."

> "I'm wary of using :not() in ePub CSS for a widely distributed title."

## Key Themes

#web #epub #html #css #publishing

The core lesson: knowing HTML and CSS is necessary but not sufficient for ePub development. You have to unlearn modern conventions. Self-closing tags are mandatory. Namespace declarations are required. Modern CSS pseudo-classes like `:is()` and `:not()` are unreliable. The rendering engines in e-readers are years (sometimes decades) behind browser engines.

The XHTML requirement is the biggest gotcha. Web HTML is forgiving -- browsers parse broken markup and do their best. XHTML is XML, which means a single unclosed tag can make the entire document fail to render. This fragility is why the web abandoned XHTML circa 2009, but ePub kept it.

The `epub:type` semantic attribute is interesting -- it enables rich semantics (endnotes with modal display, backlinks, structural roles) that standard HTML doesn't support. But it's being deprecated in favor of the Digital Publishing WAI-ARIA spec, which isn't fully implemented yet. So ePub developers are in a transition period where the old way is deprecated and the new way doesn't work everywhere.

## Critical Analysis

This is essential reading for anyone who thinks "I know HTML, so I can make an ePub." The gap between web HTML and ePub XHTML is large enough to cause real frustration if you're not warned.

The recommendation to use Standard Ebooks toolset is pragmatic -- let the tooling handle the XML scaffolding so you can focus on content. But it also means ePub development has a "framework dependency" problem similar to web development: the underlying standards are complex enough that most people work through an abstraction layer rather than directly.

The broader lesson: standards diverge. HTML on the web and HTML in ePub started from the same place and evolved differently because the constraints (browsers vs. e-readers) were different. The same thing happens in software everywhere -- a "standard" format or protocol means different things in different contexts.

---
*Sources: [[raw/how-html-changes-in-epub]]*
*Last updated: 2026-05-14*
