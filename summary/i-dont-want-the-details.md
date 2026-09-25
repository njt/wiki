---
url: https://michaelheap.com/i-dont-want-the-details/
title: "I don't want the details"
author: Michael Heap
date_fetched: 2026-09-25
date_published: 2026 (recent essay, exact date not given)
topics:
  - software-engineering-craft
  - ideas-and-culture
---

Heap recounts being on an incident call with an SVP who cut off his explanation with "Michael, I don't want the details" — not out of impatience, but as a declaration of trust. The executive assumed the people involved were competent and reasonable; what they wanted to know was what the organisation was changing.

The essay's core move is replacing the postmortem question "Why did this happen?" with "What are we changing so that the same class of failure is less likely next time?" Heap argues that a good explanation can make things worse: once everyone agrees the outcome was reasonable, urgency to change anything evaporates. Root-cause timelines become rituals that end with everyone nodding and moving on.

He applies this to familiar excuses — someone on holiday, requirements changed late, alert fatigue — and reframes each as a system-design question: how do we make ownership unambiguous when someone is unavailable? What happens when requirements change inside the launch window? How do we improve the signal-to-noise ratio of alerts? Fix the system, not the people.

A test for corrective actions: if your fix depends on people remembering a conversation from six months ago, "you don't have a corrective action. You have organizational folklore." Better test: "If the same situation happened tomorrow, what would cause a different outcome?" He also warns against over-correcting — not every failure deserves a process; sometimes consciously accepting the risk is right, but say so explicitly rather than "we'll be more careful next time."

The closing reframes the SVP's line: empathy must not become "the mechanism by which the organisation absolved itself of having to change."
