---
url: https://martinfowler.com/bliki/Yagni.html
title: Yagni
author: Martin Fowler
site: martinfowler.com
date_published: 2015-05-26
date_fetched: 2026-08-08
---

# Yagni

Martin Fowler's definitive bliki entry on YAGNI ("You Aren't Gonna Need It"), the Extreme Programming mantra that says don't build a capability until you actually need it. Using a vivid insurance-software example (pricing for storm risks today vs. piracy risks in six months), Fowler walks through why even *correct* predictions don't justify building ahead.

He identifies four costs of neglecting YAGNI:

1. **Cost of build** — wasted analysis, programming, and testing when the presumptive feature turns out to be wrong (⅔ odds, per Kohavi et al.'s Microsoft study finding only ⅓ of features improved their target metrics).
2. **Cost of delay** — building the presumptive feature now means you *didn't* build something else that could be generating revenue today. Revenue deferred is revenue lost, even when your guess is right.
3. **Cost of carry** — the extra complexity makes every subsequent feature harder to build and debug, compounding across every change until the presumptive feature ships (or is removed).
4. **Cost of repair** — when the feature turns out to be the *right feature built wrong* (because you learned things in the intervening months), you pay to fix it — or pay the ongoing tax of working around it ([[TechnicalDebt]]).

Fowler is careful to bound YAGNI: it applies only to capabilities that add complexity for a future need. It does *not* apply to refactoring, [[SelfTestingCode]], or [[ContinuousDelivery]] — these are *enabling practices* that make code malleable, and without them YAGNI "turns from a beneficial practice into a curse." YAGNI is both enabled by and enables evolutionary design.

The practical takeaway for developers: when tempted to build ahead, imagine the refactoring you'd do later to add the capability when needed. If it wouldn't be much harder, defer. If it would be, look for a minimal complexity addition now that reduces the later cost — lookup tables instead of inline literals, not a full extensibility framework.

Origin story: Kent Beck responding to Chet Hendrickson on the C3 project with "you aren't going to need it" to every proposed future capability.
