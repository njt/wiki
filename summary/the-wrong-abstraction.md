---
url: https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction
title: The Wrong Abstraction
author: Sandi Metz
date_published: 2016-01-20
date_fetched: 2026-08-08
topics:
  - software-engineering-craft
---

Sandi Metz's influential essay on the cost of wrong abstractions in software. Originally written for her Chainline Newsletter and lightly edited for her blog, the piece expands on a claim from her RailsConf 2014 talk: "duplication is far cheaper than the wrong abstraction."

Metz traces a recurring pattern: Programmer A sees duplication, extracts an abstraction, and leaves. Time passes. New requirements arrive that don't quite fit. Programmer B, feeling honor-bound to preserve the existing abstraction, adds parameters and conditionals. More requirements arrive. More parameters. More conditionals. The code becomes incomprehensible — but its very complexity creates a sunk-cost pressure to preserve it.

The core advice: when you find yourself in this situation, the fastest way forward is back. Re-introduce duplication by inlining the abstracted code into every caller, trim each caller to only the code it actually needs, and only then re-extract abstractions from what remains. This isn't retreat — it's advance in a better direction.

The essay closes by connecting this idea to the sunk cost fallacy: the more complicated and incomprehensible the code, the deeper the investment feels, and the more we feel pressure to retain it. The antidote is giving yourself permission to rethink abstractions in light of current requirements.
