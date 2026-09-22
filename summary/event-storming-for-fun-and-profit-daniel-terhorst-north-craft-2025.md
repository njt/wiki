---
url: https://gist.github.com/cb925de126d43663196746706037dacd
title: "Event Storming for Fun and Profit — Daniel Terhorst-North, Craft 2025"
author: Daniel Terhorst-North
date_fetched: 2026-09-13
date_published: 2025
topics:
  - software-engineering-craft
  - ideas-and-culture
---

A ytx gist transcription of Daniel Terhorst-North's Craft 2025 talk "Event Storming for Fun and Profit", consisting of a structured digest (key points, quotes, tools, unanswered questions) plus the full transcript and Q&A. The talk is designed by event-storming itself: it is narrated with the Disney story formula — "once upon a time Daniel decided to explain event storming… until finally the audience could try event storming for fun and profit" — because North's core claim is that event storming is narrative construction, not documentation.

The mechanics are deliberately trivial: orange stickies for domain events in the past tense, stimuli (commands, external events, time events like "the first of the month happened"), pink stickies for questions and puzzles placed diagonally, and view/read models. You start at "until finally" and fill left-to-right on a long wall with spare space on both sides, because you will discover you started in the wrong place. Attendance heuristic: "people with questions, people with answers, and someone with stationery." The goal is not the wall — it is that "everybody knows what everyone knows."

Three applications anchor the middle. Storm a business process to strip vestigial steps before automating, because automation does two things: makes a process deterministic, and "bakes it, calcifies it." Storm a legacy system to get past the data model — the suspiciously fast first dump is people reciting the schema, a phase North calls "clearing your throat" — hunting for Mike Feathers' "seams." Storm a new application and find that when pink stickies outnumber orange, "none of us knew": the work is "let's pay down some of this uncertainty," not design.

The signature war story: a trading firm's 30-day server procurement lead time, once the whole process was on a wall, dropped to 12 days by re-sequencing and parallelization (including CIO pre-approval), then to hours by pre-stocking three server types — "money makers," dev machines, "donkeys" — for a few thousand dollars of sunk cost. "We haven't done anything." The head of infrastructure: "I've never seen this process laid out end to end before."

The last third is the facilitation kit: be a time cop, set explicit ground rules, capture off-scope threads as pink stickies with actions; Goldratt's universal harmony (one reality, so conflict means divergent views — find where the parties last agreed); "arguing to learn" over "arguing to be right"; Satir's positive intent — "everyone is trying to help," ask what must be true for them, with the conceded exception that sociopaths exist "often in leadership roles"; the archetype playbook (disruptor, wallflower, helper, last-word person, surprise star); Pomodoro because storming is "very cognitively intense"; photographs plus OCR so walls are searchable; and "Trust Me Once" — frame any new practice as a cheap, reversible experiment, with Grace Hopper's forgiveness-over-permission ("JFDI") for closed-minded bosses. The gist's own digest honestly flags the gaps: no failure modes, no bridge from stickies to code, remote teams shrugged off, and no answer for who owns the pink stickies afterward.
