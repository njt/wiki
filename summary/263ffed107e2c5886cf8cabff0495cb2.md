---
url: https://gist.github.com/263ffed107e2c5886cf8cabff0495cb2
title: "Mind Your P's and Queues — Daniel Vacanti, Craft 2025"
author: Daniel Vacanti
date_fetched: 2026-09-13
date_published: 2025
topics:
  - software-engineering-craft
---

A ytx gist transcription of Daniel Vacanti's Craft 2025 talk "Mind Your P's and Queues" — the English expression decoded as "watch what you are doing," which becomes the talk's recurring thesis. Vacanti (author of *When Will It Be Done?* and *Actionable Agile Metrics for Predictability*) argues that agile prioritization optimizes the wrong moment: the Scrum Guide orders the backlog to "maximize value," but value is only realized when work is delivered, so what we actually care about is the order items **finish** — while prioritization only fixes the order they **start**. "Doesn't that seem backward?"

The mechanism is queues. Every board column and every level of the work breakdown structure — portfolio, initiative, feature, team — is a queue, and the thing "no Agile framework talks about at all" is how many items sit in them. His example: a team pulling one story each from features A through E looks disciplined at story level but has five features in progress; multiply by 10–50 teams and you get an organization-wide explosion of WIP. A Monte Carlo simulation over 18 features makes the consequence vivid: at WIP=2, priority #9 already loses to #10; at WIP=9, priority #1 has a lower completion chance than priorities 2 through 15; with everything pulled in progress, "stories not assigned to a feature" beats the #1 priority. Hence the flat verdict: "Prioritization is waste" — and the snowball: screw priority → no point planning → no predictability → no idea what to invest in across the portfolio. SAFe's implementation of weighted shortest job first is dismissed as "not a thing."

The prescription is to discover the organization's optimal capacity (the number of things it can genuinely work on at once — "maybe it's one thing, maybe it's two things. I guarantee you it's probably not 18 things") and limit WIP at every level, treating the board columns you already have as the queue map you're ignoring. In Q&A he argues team size is a red herring ("you can have a team of 60 people as long as you're not working on 60 things"), that the Agile Manifesto has no language around WIP or flow, that Scrum and SAFe becoming synonymous with agile is its "biggest perversion," and that Lean and Agile are not either/or. The gist's digest closes with an unusually sharp omissions list: no method for finding optimal capacity, no scaled answer for 50 teams, no account of what a product owner does once priority stops mattering, and the observation that the damning reorderings come from the speaker's own simulation rather than field data.
