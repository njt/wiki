---
url: https://coding-is-like-cooking.info/2026/06/i-stopped-coding-and-started-architecting-agents-and-why-you-should-too/
title: "I Stopped Coding and Started Architecting Agents (And Why You Should Too)"
author: Emily Bache
date_fetched: 2026-07-18
date_published: 2026-06-25
---

Emily Bache opens by noting that her technical coaching focuses on the "minute-by-minute habits" developers need for high-quality, durable software. Teams she encounters are now requesting guidance on AI tools. She still teaches the same code quality skills but now applies them to agentic AI, introducing the concept of **Harness Engineering**.

She describes programming as composed of many "microskills" that compound into better outcomes. The macro-behavior she advocates for is working in small steps with frequent steering opportunities. She references Birgitta Böckeler's work on Harness Engineering, explaining that a harness includes both **Guides** (feed-forward advice given before the AI acts) and **Sensors** (feedback mechanisms that check output after the fact). The harness's purpose is to "constrain the AI to come up with good quality code and tests."

Rather than downloading pre-made harnesses, she recommends teams learn to build their own so they can adapt as needs evolve.

Bache notes she has turned off AI-generated line completion, calling it "a huge distraction." Agentic AI differs fundamentally because it uses tools in a loop to iterate toward useful, compiling results.

She describes a flywheel effect: "a better harness leads to better code leads to better harness and code that improves over time." This is especially valuable with legacy codebases where poor existing patterns could otherwise be replicated by AI.

She recommends beginning with unit tests. Teams improve test design through iterative prompting and learning hours. Once a good design example exists, it gets codified as a **Guide** — a knowledge document the agent uses. Guides provide advice before code is written. **Sensors** provide feedback after code is written, often via deterministic scripts checking for code smells like excessive file length.

Each successful design task leads to updated Guides and Sensors, making it progressively easier to get good results from the agent while the codebase accumulates positive examples. She calls this the "harness flywheel" effect. In legacy code, design improving over time is a major win, allowing teams to defer risky rewrites.

She emphasizes that harness engineering also involves **removing** items. Harnesses tend to grow, but as codebases and models improve, less guidance may be needed. She references an Anthropic article noting that "A good AGENTS.md is a model upgrade. A bad one is worse than no docs at all."

She suggests making harness updates part of every task, doing spot checks by comparing results with and without a proposed change, and treating the harness as a first-class part of the codebase. Teams should avoid adopting someone else's harness wholesale because they "won't know what's in it and will be afraid to change it."

Bache expresses that agentic AI gives her hope that teams with legacy code can create a "harness engineering flywheel" where code design steadily improves. The same engineering principles she has always taught apply, but AI now helps teams "adopt this approach consistently on all changes."

The post closes with "Happy Coding!" and links to a related article titled *The Death of the Senior Dev? How Agentic Workflows Changed EVERYTHING in 2026*.
