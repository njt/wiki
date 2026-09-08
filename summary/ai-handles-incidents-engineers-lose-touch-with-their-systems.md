---
url: https://www.sylvainkalache.com/blog/ai-handles-incidents-engineers-lose-touch-with-their-systems
title: AI handles incidents, engineers lose touch with their systems
author: Sylvain Kalache
date_fetched: 2026-09-08
date_published: undated
---

Sylvain Kalache — AI Labs lead and DevRel at Rootly, formerly an SRE at LinkedIn and co-founder of Holberton School — argues that AI-assisted incident response ("AI SREs") is quietly robbing engineers of the one thing that makes them good at incidents: practice. The better these tools get at auto-resolving routine incidents, the less experience human responders accumulate, so that when a genuinely novel, high-severity incident arrives — the kind automation cannot solve — engineers take over with less intuition than the previous generation had.

Kalache grounds the worry in Lisanne Bainbridge's 1983 paper *The Ironies of Automation*: automation removes operators' opportunities to practice routine work while still leaving them responsible for abnormal situations, so they actually need *more* skill and training than before. He predicts average MTTR for routine incidents will fall while resolution time for complex incidents "shoots up."

The prescription is borrowed from aviation, where pilots must rehearse rare emergencies in simulators (FAA recurrent training every six months) precisely because a real engine failure may never occur in a career. Rootly partnered with Uptime Labs to build realistic incident simulations — engineers take the incident commander's seat during a simulated outage, coordinating with LLM-powered stakeholders in Slack. Kalache also suggests using AI as a trainer (ask an agent to explain its steps and evidence), but warns that explanation is not a substitute for practice: "you only learn tennis by getting on the court." His central coinage is **comprehension debt** — the growing gap between how systems work and how well responders understand them — and his proposed fix is to fold incident simulation into on-call readiness.
