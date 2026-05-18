---
url: https://gist.github.com/njt/fc8fac105dcc477c2f654da67c349029
source_url: https://www.youtube.com/watch?v=owmJyKVu5f8
title: "Simon Willison: Engineering practices that make coding agents work"
event: The Pragmatic Summit
author: Simon Willison
date_published: 2026-02-11
date_fetched: 2026-05-18
tags: [coding-agents, tdd, sandboxing, prompt-injection, open-source, vibe-coding]
---

## Summary

Simon Willison's talk at Gergely Orosz's inaugural Pragmatic Summit (February 11, 2026, San Francisco). Co-presented with Marcos Arribas (VP Engineering, Statsig). Willison walks through his personal engineering practices for working with coding agents — what he's learned from shipping hundreds of projects, and where the boundaries of responsible agent use currently sit.

### Key Points

1. **The adoption ladder has a new rung: don't read the code.** Programmers progress from asking chatbots questions to letting agents write snippets, to agents writing more code than humans, to writing no code at all. The newest stage — arriving only weeks before this talk — involves *not reading the code* the agent produces. Willison calls this "clear insanity" but says it's possible if you shift energy into making the agent *prove* the code works.

2. **Trust arrived in November 2024, and again last week.** The key models are Claude Opus 4.5 / GPT-5.01 (November) and later Opus 4.6 / Codex 5.3. Before November, agent-written code was "janky" and needed fixing. Now, for well-understood problem classes, code is predictably correct. Willison reports "one-shotting basically everything" and no longer needing to ask whether it will work.

3. **Test-driven development is the unlock, and tests are now free.** Willison hated TDD for himself but finds it perfect for agents. Telling an agent "use red-green TDD" (five tokens) dramatically raises the chance of working code. Because agents do the work, the old excuse that tests are extra effort evaporates. "Tests are no longer even remotely optional."

4. **Manual testing by the agent catches what automated tests miss.** Willison instructs agents to start the server and exercise the API with `curl`, which often surfaces bugs the test suite didn't cover. He built a tool called Showboat that produces a markdown log of the manual test session.

5. **Conformance-driven development: let a test suite be the spec.** When a language-agnostic conformance suite exists (e.g., WebAssembly's spec tests), you can give it to an agent and say "write code until this passes." Willison also reverse-engineers a standard by having an agent build a test suite that passes against six existing implementations, then using that suite to drive a new implementation.

6. **Code quality is a choice you make — or don't.** For throwaway "vibe-coded" tools, spaghetti is fine. For maintained projects, you can feed refactoring instructions back to the agent and end up with code *better* than you'd write by hand, because the agent does the tedious cleanup you'd skip.

7. **Templates and existing patterns act as a straitjacket for the agent.** Starting from a cookie-cutter template that sets up tests, CI, and a README means the agent will follow those patterns. Keeping your codebase high-quality ensures the agent adds to it in a high-quality way — exactly like a human team that copies the first Redis usage.

8. **Prompt injection is the lethal trifecta, and sandboxing is the only real defense.** The three legs: a model with access to private data, exposed to malicious instructions, and possessing an exfiltration vector. The only guaranteed fix is to cut off one leg — usually the exfiltration channel. Sandboxing (containers, remote VMs) limits damage. Willison runs Claude with dangerously skipped permissions on his Mac despite being "the world's foremost expert on why you shouldn't do that," but defaults to Anthropic's hosted containers on his phone.

9. **Managing agents is mentally exhausting, and that might save our careers.** Keeping three or four agents busy in parallel requires operating at full throttle. After a couple of hours, Willison is "done for the day." The limit isn't the AI; it's the human's cognitive stamina. That, he suggests, is why one engineer won't replace a thousand.

10. **The models' capabilities are a moving target we haven't begun to map.** Willison refuses to predict more than a week ahead. He believes it will take six months just to explore what Opus 4.6 can do. His advice: every time a model fails at something, tuck it away and retry in six months — you might be the first to discover a new capability.

11. **Open source is being reshaped in uncomfortable ways.** Why use a date-picker library when you can vibe-code the exact widget you want? The market for paid component libraries is collapsing. Meanwhile, maintainers are drowning in junk AI-generated pull requests, to the point that some want GitHub to disable pull requests entirely.

### Notable Quotes

- "The new thing, as of what, three weeks ago, is you don't read the code. … That is a wildly irresponsible thing to do."
- "Tests are no longer even remotely optional. Tests are—they're free now."
- "I end up with code that is way better than the code I would have written by hand because I'm a little bit lazy."
- "The lethal trifecta is when you've got a model which has access to three things … the only guaranteed solution is to cut off one of the legs."
- "We're seeing contributor projects are flooded with junk contributions at the moment, to the point that people are trying to convince GitHub to disable pull requests."
- "I try not to predict more than a week ahead."
- "I think that might be what saves us. I think the fact that no, you can't have one engineer and have him do a thousand projects because after three hours of that he's going to literally pass out in a corner."
- "Don't learn it, just start writing code in it." (on learning new programming languages)

### Tools and Practices

- **Red-green TDD with agents** — Every session starts with "use red-green TDD."
- **Showboat** — A tool (48 hours old at the time) that produces a markdown document logging the agent's manual testing steps via `exec` command capture.
- **Cookie-cutter templates** — Python's `cookiecutter`; half a dozen templates that scaffold projects with tests, README, and CI.
- **Conformance-driven development** — Give an agent a language-agnostic test suite; tell it to write code until it passes.
- **Agent-performed manual testing** — After automated tests pass, tell the agent to start the server and exercise the API with `curl`.
- **Parallel agent sessions** — Keep multiple projects active to switch when one agent is churning.
- **Sandboxing via remote containers** — Claude Code for the Web's Anthropic-managed Linux VM.
- **Mocking over production data** — Invest in good mocking rather than copying sensitive user data into agent environments.
- **Learning new languages by prompting** — Three Go projects in two weeks without being a fluent Go programmer.
- **Vibe-coding throwaway tools** — For one-off tools, accept spaghetti; for maintained projects, demand quality.
- **Refactoring via agent feedback** — Feed refactoring instructions back; the agent does tedious work you'd skip.

### Gaps and Unanswered Questions

- How do you actually "prove" correctness without reading code?
- What about security vulnerabilities in agent-generated code (SQL injection, auth flaws, crypto mistakes)?
- The exhaustion problem is raised but not solved — no strategies for sustaining pace.
- What happens to junior engineers? How do novices build mental models?
- The economics of open source are left dangling.
- Cost and environmental impact are absent ($200/month plans, energy footprint).
- What about non-code artifacts? Architecture, system design, documentation.
- The "don't read the code" threshold is fuzzy — no guidance on building that intuition.
- Model lock-in and monoculture risk are not discussed.

---

*Full transcript available at the source gist. This summary condenses a 28-minute talk.*
