# HTTP Status Codes in Scots

A novelty page translating the HTTP status code table into Scots dialect — one line per code, from "AYE ON YE GO" (100) through "AH'M A FUCKIN TEAPOT" (418) to "THE OTHER BAMPOT'S BROKEN" (502). It is a joke, but a technically literate one: almost every phrase is an accurate gloss of what the status code means, rendered in the voice of a Glaswegian asking you what you think you're doing.

---

## What it is

A bare table: numeric code on the left, Scots phrase on the right. No prose, no explanation. The humour works because it assumes the reader already knows the RFC semantics and is grading the translation against them.

## Key quotes

> 404 | NO FUCKIN HERE PAL

The canonical case: the phrase carries exactly the information the code carries, plus an attitude the code has always implied to anyone debugging a broken link at 2am.

> 418 | AH'M A FUCKIN TEAPOT

RFC 2324's joke status code, kept as a joke — but translated rather than copied, which is the wittier choice.

> 508 | ROUND AN ROUND YA DAFT TWAT

A perfect gloss of "Loop Detected": the phrase describes the failure mode, not just the fact of failure.

> 502 | THE OTHER BAMPOT'S BROKEN

"Bad Gateway" is about the *other* server being the problem, and this is the only phrasing in the list that captures the shifting of blame which is the semantic heart of a 502.

## Themes

#concept — status-code semantics as folklore: the page demonstrates that developers already hold a vernacular understanding of HTTP codes, and formalising it into dialect reveals how much social meaning the numbers carry.
#pattern — good error messages describe the situation, not the abstraction. Nearly every translation here beats the official reason phrase on communicative content.

## Take

This is a comedy page that doubles as a surprisingly sharp critique of HTTP reason phrases. "Not Found" and "Bad Gateway" are bloodless bureaucratic abstractions; "NO FUCKIN HERE PAL" and "THE OTHER BAMPOT'S BROKEN" are how engineers actually talk about these failures. The Scots voice works because Scots vernacular is famously direct about blame and causation — which is precisely what a status code is for. Anyone who has ever had to explain the difference between a 401 and a 403 to a product manager will recognise that "WHO THE FUCK ARE YOU" versus "YE CANNAE DAE THAT" is a better teaching pair than "Unauthorized" versus "Forbidden". It belongs in the same drawer as [[This Is a Modern Motherfucking Website]] — profanity deployed as documentation strategy — and it sits alongside [[HTTP API Design Guide (Heroku)]] as a reminder that API design docs describe an idealised protocol that practitioners experience as a series of arguments.

Related: [[HTTP API Design Guide (Heroku)]] — the sober spec-driven counterpart; this page is what that guide's status-code table looks like in production. [[This Is a Modern Motherfucking Website]] — the same genre: profane web-humour pages that smuggle in real opinions about web craft.

---
*Sources: [[raw/scots-html]], [[summary/scots-html]]*
*Last updated: 2026-11-20*
