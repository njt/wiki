---
url: https://gist.github.com/5a3168d465667b77688dd74ba9b03271
title: "Building World-Class Engineering Teams in the Age of AI"
author: Rajeev Rajan, Thomas Dohmke
date_fetched: 2026-07-04
date_published: 2026-05-18
source_type: ytx gist (YouTube transcript + summary)
event: The Pragmatic Summit, 2026-02-11
moderator: Gergely Orosz (The Pragmatic Engineer)
speakers:
  - Rajeev Rajan (CTO, Atlassian)
  - Thomas Dohmke (CEO, Entire; former CEO, GitHub)
duration: 33m 36s
topics:
  - agent-coding-workflow
---

# Building World-Class Engineering Teams in the Age of AI — The Pragmatic Summit

A fireside chat with Rajeev Rajan (CTO, Atlassian) and Thomas Dohmke (former CEO of GitHub, now founding Entire.io), moderated by Gergely Orosz at The Pragmatic Summit, February 11, 2026. The conversation covers what "AI-native" engineering orgs look like in practice: how teams restructure, how roles blur across PM/design/engineering, and what changes when the bottleneck shifts from writing code to intent, review, and verification.

## Key Points

1. **AI-native mindset over tools.** Rajeev: teams must "really believe in doing AI native work" and working with agents. Some Atlassian teams still code traditionally, but AI-native ones write zero lines of code manually — everything is agent orchestration.

2. **Bottleneck shifts left and right of code.** As coding becomes "free," the difficult parts become planning/specking (left of code) and deployment/incident resolution (right of code). Atlassian retools the entire SDLC around this shift.

3. **Role collapse into overlapping Venn diagrams.** PMs become product engineers, designers become design engineers. Thomas: "The product manager is becoming a product engineer and the designer is becoming a design engineer."

4. **Productivity gains with creativity as the goal.** Rajeev reports PRs per engineer up 89%, issue cycle time down 42%, 51% of security vulnerabilities caught by agents. Yet the point is "what can you create now with AI that you could not create before" — the efficiency/headcount reduction framing "is missing the point."

5. **Context is magic for agent performance.** Atlassian's RoboDev beats Devin on SWE-bench by using the "teamwork graph" — mapping who works with whom on which PRs and Jira issues — to give agents rich context. "Agents are as smart as the context you give them."

6. **Distributed teams gain advantage from agents.** Thomas argues agents act as always-available pairing partners for brainstorming, code review, and research, leveling the competitive disadvantage of remote work vs. in-office whiteboarding.

7. **Engineering leaders can code again; span of control grows.** Rajeev sees leaders reconnecting with code via agents. Predicts flatter orgs with managers having 20–50 direct reports, fewer management layers. "Anytime somebody asks me about career path as an engineer, the first thing I tell people is don't be a manager."

8. **Token costs invert traditional budgeting.** Thomas warns that as developers get more productive, flexible token costs skyrocket, creating pressure to slow down some developers — a large-company problem to solve with finance. "Nobody wants to fire people to offset the cost of the tokens."

9. **Coding is fun again.** Both speakers emphasize that agents remove drudgery (build errors, unit tests, boilerplate) and restore the joy of building, especially for hobby projects and rapid prototyping.

## Pithy and Provocative Quotes

- **Thomas on the hype-reality gap:** "Hundreds of emails from investors" while looking for "the agent that solves all that for me" instead of managing polite rejections.

- **Rajeev on AI's real promise:** "The discussion about using AI to produce more efficiency and maybe smaller teams and fewer engineers is missing the point."

- **Thomas on markdown collaboration:** "A nightmare" to have 3,000 engineers collaborating on Markdown files. "Programming language is great" because "it reduces vocabulary on like 20 words that everybody can speak."

- **Rajeev on context as secret sauce:** "Agents are as smart as the context you give." Atlassian's "teamwork graph" — "that context really helps Robo dev do a much better job."

- **Thomas on the Homer Simpson car of features:** "What you get is the Homer Simpson car" — lots of features generated without process or creativity.

- **Rajeev on career advice:** "Anytime somebody asks me about career path as an engineer, the first thing I tell people is don't be a manager."

- **Thomas on Atlassian's CTO buying his own laptop:** CTO had to buy a laptop "on his own money to start coding" — proof incumbents can't move fast.

- **Thomas on joy:** "Coding is fun again." Agents "bring us back to the joy of coding."

## Tools, Practices, and Methodologies

- **RoboDev:** Atlassian's internal coding agent used across full SDLC — code generation, review, CI/CD, incident resolution. Built on Anthropic's model but beats Devin on SWE-bench via "teamwork graph" context.
- **Teamwork graph:** Internal knowledge graph mapping who works with whom on which PRs and Jira issues. Rich agent context.
- **Confluence for left-of-code speccing:** Engineers and PMs put intent, specs, and comments in Confluence; agents read comments and run RALPH loops to generate code. The artifact of record shifts from code to the spec.
- **Loom:** Video messaging used to express intent and thoughts, fed into AI agents to produce better artifacts — moving beyond text-only specs.
- **Cursor Composer:** Used by Thomas to generate cookie policy implementation in seconds.
- **Devin (Cognition AI):** Some CTOs use it overnight to build features, check results in the morning.
- **Replit / Lovable:** Low-code/no-code platforms PMs, marketers, and assistants use to build prototypes.
- **Codex Mac app (OpenAI):** Thomas built three native Mac menu-bar apps in SwiftUI without looking at the code.
- **AI-native SDLC:** Atlassian's holistic approach: ideation → coding → deployment → production, all augmented by agents.
- **Verification over inspection:** Shifting from line-by-line code review to verifying inputs/outputs, guardrails, and system behavior.
- **Forcing yourself not to look at code:** AI-native developers start greenfield projects by voicing intent through prompts, deliberately avoiding reading generated code.

## Unanswered Questions

1. Managing token costs at scale — Thomas flags the inversion where productive devs burn more tokens, requiring potential slowdowns. No solutions offered.
2. Junior engineers when seniors become "masters of agents" — how do juniors learn fundamentals, develop taste, or gain context to effectively prompt and verify?
3. Quality/security when humans stop reading code — verification of inputs/outputs is mentioned, but guardrails and testing strategies are vague.
4. Legacy codebase problem — Rajeev admits agents struggle with complex old codebases, says "we'll get there very soon," but no concrete approaches.
5. Preventing the "Homer Simpson car" when agents auto-merge/deploy — no process changes explored for product management adaptation.
6. Team morale and culture during role collapse — friction or resistance isn't addressed.
7. Language/communication barriers with English prompting — non-native speakers struggle to describe features. No mitigation offered.
8. Career ladder with fewer managers and flatter orgs — "managers and leaders, it's a little tougher" but no sketch of advancement paths.
9. Institutional knowledge when code is generated, not deeply understood — long-term maintainability when human intent is buried in prompts.
10. Ethical/legal dimensions — no mention of IP ownership, licensing, compliance, or bias in generated code.
