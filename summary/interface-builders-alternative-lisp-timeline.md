---
url: https://paulhammant.com/2013/03/28/interface-builders-alternative-lisp-timeline/
title: "Interface Builder's Alternative Lisp timeline"
author: Paul Hammant
date_fetched: 2026-11-11
date_published: 2013-03-28
topics:
  - ideas-and-culture
  - developer-tools
---

Paul Hammant reconstructs the history of Interface Builder's "other" lineage: Denison "Denny" Bollay's ExperTelligence team, whose 1986 Lisp-based Interface Builder demoed to Steve Jobs (the ancestor of NeXT/Apple's IB) and whose 1988 follow-up, Action! (built with Texas Instruments), was a fully live, runtime-modifiable UI tool that saved its state as pretty-printed Lisp — round-trippable in Vi/Emacs and source-control friendly decades before .xib.

The piece lays out a timeline of markup formats: binary Mac resource forks, NeXT's .nib, Apple's XML-based .xib, and Action!'s interpreted Lisp DSL, arguing Lisp was a better UI markup than any of them. Hammant then contrasts Action! with the modern DOM (live mutable object model, instant design/run switching, high-level-language bindings — but no URLs, no pages, no XPath) and asks "what if?" — including a 2023 ChatGPT-armed speculation on whether a Lisp-save-format NeXT would have given the web a Lisp-native HTML.

Two digressions carry the essay: Berners-Lee used NeXT's Interface Builder to build the first browser (WorldWideWeb/Nexus), and Bollay's CIA-funded "Dynamic Documents" hypertext system (with Xerox PARC's NoteCards heritage) reviewed the early HTML spec in the early 90s and judged it "dumb from almost every angle" — contextual, out-of-band, bidirectional links between arbitrary documents, no re-authoring required. "Obviously we misjudged," Bollay concedes. Hammant also notes that Action!'s 1988 sales vocabulary sounds strikingly like the modern Agile community.
