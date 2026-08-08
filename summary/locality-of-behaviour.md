---
url: https://htmx.org/essays/locality-of-behaviour/
title: Locality of Behaviour (LoB)
author: Carson Gross
site: htmx.org
date_fetched: 2026-08-08
---

# Locality of Behaviour (LoB) — Summary

Carson Gross formulates Locality of Behaviour as a prescriptive software design principle: **the behaviour of a unit of code should be as obvious as possible by looking only at that unit of code**. It operationalizes Richard Gabriel's observation that "the primary feature for easy maintenance is locality."

The essay contrasts two AJAX implementations: an htmx button (`<button hx-get="/clicked">Click Me</button>`) where behaviour is visible on the element itself, versus a jQuery approach where behaviour is spread across separate files with a selector (`$("#d1").on("click", ...)`) — what Gross calls "spooky action at a distance."

A critical distinction runs through the argument: **surfacing behaviour is not inlining implementation**. Declaring *that* a button issues a GET request is different from inlining *how* that GET request is executed — just as calling a function is different from copy-pasting its body. Framework developers carry extra responsibility to make LoB "as easy and as conceptually clean as possible."

LoB inevitably conflicts with other principles, especially **DRY** (Don't Repeat Yourself) and **Separation of Concerns**. Gross treats these as subjective tradeoffs rather than absolutes: behaviour moved to a parent element a few lines away is a mild LoB violation; behaviour in a separate file is a severe one. The rise of inline styles in recent years is offered as evidence that Separation of Concerns is already losing ground.

The conclusion frames LoB as a **subjective principle** that must be balanced against other concerns and system constraints, but one that — pursued as far as practical — increases "software maintainability, quality and sustainability."
