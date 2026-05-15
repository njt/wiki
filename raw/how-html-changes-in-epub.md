---
title: "How HTML Changes in ePub"
url: https://www.htmhell.dev/adventcalendar/2025/11/
date_fetched: 2026-05-14
section: "Random"
---

# How HTML Changes in ePub
By Robin Whittleton (December 11, 2025)

## Core Argument
ePub uses HTML technology but with significant departures from web development. Developers must "unlearn" conventional knowledge to work effectively with ePub's constraints.

## Key Technical Differences

### XHTML Foundation
ePubs require XHTML based on the XML specification, not modern HTML5:
- Syntactically valid XML markup required
- Self-closing tags mandatory
- Correct namespace declarations
- XML attributes (xml:lang)

"XHTML didn't work out" for web browsers due to fragility and performance issues, yet persists as the ePub standard.

### CSS Limitations
E-readers use "positively historic" rendering engines. Developers should avoid modern pseudo-classes like :is() and :not(). CSS must account for XML namespaces using the pipe separator.

"I'm wary of using :not() in ePub CSS for a widely distributed title."

### MathML and SVG
Both require namespace declarations in XHTML with prefixed elements.

### ePub-Specific Semantics
The epub:type attribute enables semantic richness unavailable in standard HTML (endnotes, backlinks). Gradually being deprecated in favour of Digital Publishing WAI-ARIA spec.

### Extended Vocabularies
Z39.98-2012 Structural Semantics Vocabulary extends ePub through namespaced attribute values.

## ePub File Structure
- META-INF/container.xml pointing to package files
- Package file with metadata, manifest, and spine
- XHTML files for content
- Zipped archive with .epub extension

Recommends Standard Ebooks toolset for beginners.
