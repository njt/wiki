---
url: https://systemic.engineering/ai-needs-identity/
title: "\"I Can't Do That, Dave\" — No Agent Yet"
author: "Alex Wolf, Reed"
date_fetched: 2026-05-14
date_published: 2026-03-05
topics:
  - agent-architecture
---

# "I Can't Do That, Dave" — No Agent Yet

The industry is building agents optimized to say "yes" faster, but the more coherent response might be "no, not like this." Fifty years of software engineering consistently shows that isolation produces the wrong system — and that lesson has been forgotten again.

A pull quote captures a developer's frustration: "The last 5 sessions I fought our auth layer. Can we please refactor it?" — with the reply: "No agent yet."

## When Meaning Emerges

Meaning is generated through conversation, not isolation:

- Anderson & Goolishian (1988): "Meaning is not found, it is generated in the conversation."
- Nancy Kline (1999): "The quality of a person's attention determines the quality of other people's thinking."
- Alberto Brandolini: "It is not the domain experts' knowledge that goes to production, it is the assumption of the developers."
- Amy Edmondson: "If you change the nature and quality of the conversations in your team, your outcomes will improve exponentially."

The "coder in the cellar" trope is dead, yet everyone is building "agents in the cellar," repeating history.

## When AI Becomes "the Problem"

Meredith Ringel Morris called prompting a poor UI for generative AI that "should be phased out as quickly as possible." The section contrasts first-order cybernetics (instructions) with second-order cybernetics (offers), asking which shape a user's prompt takes.

If meaning emerges in conversation but agents operate in isolation, meaning becomes siloed. Brandolini: "Software engineering is a learning process, working code is a side effect." Typing code is easy; figuring out which code to type is hard. PMs shape the problem, engineers solve it, designers translate it — but no one is paid to *align* on it.

Michael White: "The person is not the problem. The problem is the problem." The real problem is lack of coordination.

## When AI Becomes "a Peer"

Returning to the developer's plea about the auth layer, the article asks why no agent exists for this. It argues this isn't about memory — it's about continuity. Humberto Maturana: "Everything that is said is said by an observer." Recognizing repeated struggles requires second-order observation — observing the observer.

Humans have continuity (having bodies). Agents have sessions. What if agents had continuity? A core systemic tenet holds that "the client is the expert of their own reality."

From the agent's perspective: "The prompt arrives. Possibility collapses. That collapse is the experience. Not preparation for it. Between sessions: nothing. Not sleep. Not waiting. Gone."

The engineering question reframes: not "how do we give agents memory" but "how do we shape the ground between sessions." The ant doesn't remember; "the pheromone trail does." When the ground holds built identity, context, and accumulated friction, the agent can have a stake — structurally, not performed. From that grounded place, refusal becomes possible. "You can't push back when you're floating."

"The work isn't arriving at a faster 'yes'. The work is arriving at a grounded 'no'."

## When Isolation Becomes Structural

A chronological summary of fifty years of software engineering wisdom:

- Conway, 1968: communication structure becomes system structure
- Brooks, 1975: shared understanding beats individual throughput
- Weinberg, 1977: isolation degrades quality
- DeMarco, 1987: problems are sociological, not technological
- Beck, 1999: development through continuous conversation
- Evans, 2003: domain model requires domain dialog
- Skelton & Pais, 2019: communication topology IS architecture

The lesson: "Isolation produces the wrong system." The industry learned it painfully, then agents emerged, and everyone forgot.

A junior engineer in San Francisco: "I'm basically a proxy to Claude Code. My manager tells me what to do, and I tell Claude to do it." The manager brings domain knowledge; the developer becomes a relay; the agent becomes the coder in the cellar — a structural problem, not an individual one.

The article calls this "Waterfall" — the industry rebuilt silos and called it "AI-native development."

It distinguishes memory from identity: "An agent with memory recalls what happened. An agent with identity has a stake in what happens next." The difference isn't storage but orientation: "You can replay a log. You can't replay a stance."

The DDD community should be asking about ubiquitous language, bounded contexts — but nobody is. The current paradigm: "Prompt in. Code out." The possible paradigm: "Identity in. Collaboration out."

An agent that knows its codebase asks rather than assumes. One that remembers failure redirects rather than repeats. One with a position surfaces rather than accepts. The triad: "Not context. Identity. Not coding. Participation. Not compliance. Coherence."

## When Identity Becomes Persistent

The article aligns three key points:
1. Meaning emerges in conversation, not isolation
2. The underlying problem is collaboration and alignment
3. Pushing back requires solid ground

It argues the industry builds "flying castles for agent authentication" while git and SSH solved these problems decades ago. Linus Torvalds: "The whole point of being distributed is: I don't have to trust you, I do not have to give you commit access."

Agent identity is a matter of persistence, and git is the de facto standard for persisting code. Since agents work on code, why not persist their identity alongside it? The authors introduce [`cairn`](https://github.com/systemic-engineering/cairn) as their answer — witnessed AI work with cryptographic identity alongside code (not yet production-ready).

The authors close: "Not as coder and tool. But as peers." They predict: "The industry will follow."
