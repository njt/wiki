# Building World-Class Engineering Teams in the Age of AI

Rajeev Rajan (CTO, Atlassian) and Thomas Dohmke (former CEO GitHub, now Entire.io) in a fireside chat with Gergely Orosz at [[The Pragmatic Summit]]. The most candid public conversation between two leaders of large engineering orgs about what actually changes when every engineer has an agent — not aspirational future-casting, but real metrics from 89% more PRs per engineer and the admission that they still haven't figured out token costs, junior growth, or what happens when no one reads the code anymore.

---

## Precis

Two engineering leaders at very different points in the hype cycle — Rajeev running AI-native transformation inside a 7,000-person public company, Thomas fresh from leaving GitHub to build a startup — compare notes on what's real. Rajeev has metrics (89% more PRs, 42% faster cycle time, 51% of security vulns caught by agents) and the conviction that efficiency framing "misses the point." Thomas has the honesty to admit he's drowning in investor emails while still dealing with HR systems and insurance forms. The conversation hits the big structural questions — role collapse, verification over code review, span of control, token cost inversion — with more candor than answers. The most interesting moments are the tensions neither resolves: "don't be a manager" from a CTO, "zero lines of code" as both aspiration and admission, and the Homer Simpson car as a warning about agent-generated feature explosion.

---

## Key Quotes

> "The discussion about using AI to produce more efficiency and maybe smaller teams and fewer engineers is missing the point."

Rajeev's opening thesis. He's not wrong that creativity and joy matter, but this is also the CTO of a public company with 7,000 engineers telling you headcount won't shrink. The self-interest is visible, but so is the genuine belief that "what can you create now that you couldn't before" is the better question. Both can be true.

> "Agents are as smart as the context you give them."

The single most actionable technical insight. Atlassian's RoboDev beats Devin on SWE-bench not because of model choice (it uses Anthropic) but because of the "teamwork graph" — a knowledge graph of who works with whom on which PRs and Jira issues. This is context engineering as competitive advantage, and it's the argument for why incumbents with rich internal data might have an edge over startups with better models. See [[Agent Memory and Context]], [[Maybe Coding Agents Don't Need a Bigger Memory]].

> "Anytime somebody asks me about career path as an engineer, the first thing I tell people is don't be a manager."

Rajeev says this flatly, then admits he only became a manager because he "found decisions being made that I didn't understand." The advice is authentic — he predicts 20–50 direct reports per manager, fewer management layers, leaders coding again — but the subtext is complicated. He's a CTO telling engineers not to follow his path. [[Martin Fowler and Kent Beck on Reinventing Software]] covers the same "re-soloing" pattern from a different angle.

> "What you get is the Homer Simpson car" — lots of features, no process, no creativity.

Thomas's warning about agent auto-merge + auto-deploy pipelines. When every backlog item becomes a PR and every PR auto-merges, you get feature explosion without curation. The bottleneck shifts from production to taste. This is the product-management problem that nobody at the summit had a session on — see [[The Pragmatic Summit]]'s critical analysis of the missing evaluation/QA track.

> "Coding is fun again."

Thomas's closing note, and the most emotionally honest moment of the talk. He describes agents handling the drudgery — build errors, unit tests, boilerplate — and building three native Mac menu-bar apps in SwiftUI "without ever looking at the code." The "raise your hand if you love writing unit tests" bit is cheap theater but the feeling is real. Agents as drudgery-removers resonate more than agents as 10x-amplifiers. [[Building When It Feels Like There's Nothing Left to Build]] and [[The Joy and Power of Understanding]] tackle the philosophical side of this.

> "Nobody wants to fire people to offset the cost of the tokens."

Thomas names a problem nobody else is talking about: token costs are flexible, developer salaries are fixed. When productive developers burn more tokens, you get an inversion where managers might need to *slow down* their best people. Fifty years of "make developers more productive" as unquestioned good, suddenly complicated by a line item on the cloud bill. No solutions offered — this is the "we should figure this out" stage.

---

## Key Themes

- **#concept AI-native mindset** — Not about tools, about belief. Rajeev distinguishes between teams that "really believe in doing AI native work" and those that don't. The difference shows up in whether engineers write code or orchestrate agents. [[Agent Coding Workflow]] covers the practitioner's version of this spectrum.

- **#concept Bottleneck migration** — Coding becomes free; the constraint moves left (planning, speccing, intent) and right (CI/CD, deployment, incident resolution). This is the same pattern Fiona Fung describes in [[Running an AI-Native Engineering Org]] and Nicole Forsgren names in [[Nicole Forsgren on AI and Developer Productivity]].

- **#pattern Role collapse** — PM → product engineer, designer → design engineer. The Venn diagrams overlap until they're nearly concentric. Thomas extends this to marketers, comms people, assistants using Lovable/Replit. The question becomes: "do we find ways to let everybody be a builder?" [[Product-Minded Engineers in an AI-Native World]] and [[Specifications as the Product]] are the companion pieces.

- **#pattern Verification over inspection** — Shifting from line-by-line code review to verifying inputs/outputs/guardrails. Code review bots negotiate directly with coding agents. The human moves from reviewer to verifier. [[The End of Code Review]] makes the academic case; this is the practitioner version.

- **#tool RoboDev** — Atlassian's internal coding agent across the full SDLC. The key differentiator isn't the model (Anthropic) but the "teamwork graph" context layer. Beats Devin on SWE-bench. An existence proof that enterprise agents can outperform standalone tools when they have institutional context.

- **#concept Teamwork graph** — Internal knowledge graph mapping collaboration patterns across PRs and Jira issues. This is the "magic word" that makes RoboDev competitive. Context as infrastructure, not prompt engineering. [[Agent Memory and Context]] and [[Uber — Agentic Engineering Shift]] cover similar enterprise context strategies.

- **#concept Span of control expansion** — Rajeev predicts 20–50 direct reports per manager, citing Jensen Huang (40–50) and others. Flatter orgs, fewer pure managers, leaders who code. The career-ladder implications are unexplored but significant.

- **#person Rajeev Rajan** — CTO of Atlassian. Previously at Meta for 10 years. Built RoboDev and the AI-native SDLC at Atlassian. The "don't be a manager" CTO who bought his own laptop to start coding again.

- **#person Thomas Dohmke** — Former CEO of GitHub (2018–2025), now founder of Entire.io. The most prominent Git-platform executive now building something post-GitHub. Described as a believer in distributed teams and agent-assisted development.

---

## Critical Analysis

**The metrics are real but the framing is careful.** Rajeev's numbers — 89% more PRs, 42% faster cycle time — are impressive activity metrics. But [[Writing Code vs. Shipping Code]] already showed AI-driven commit gains attenuate from 180% to 30% at the release level. PR count isn't shipping. The 51% of security vulnerabilities caught by agents is genuinely interesting — that's a quality metric, not a throughput metric — but there's no breakdown of severity or false negatives. Atlassian is selling a transformation story as much as reporting one.

**"Zero lines of code" is a flex with a greenfield bias.** Rajeev's AI-native teams write no code manually — all agent orchestration. But he admits legacy codebases are hard and "we'll get there very soon." Every large company has 20 years of accumulated code. The zero-code vision works for new projects; the migration path for existing systems is hand-waved. This is the same gap between [[The Founder's Playbook]] (greenfield startup playbook) and the reality of brownfield enterprise development.

**The teamwork graph is the most under-discussed idea.** Everyone talks about models and prompts; Rajeev built infrastructure. The teamwork graph — mapping collaboration patterns across the org to give agents context — is a genuinely novel approach to the context problem. It's also a moat: startups can't build one because they don't have the interaction data. This is the strongest counter to Thomas's "CTO bought his own laptop" barb about incumbent speed.

**Thomas's honesty about the hype-reality gap is the talk's best feature.** He's launching a startup, getting "hundreds of emails from investors," and still filling out board insurance forms. He's "looking for the agent that solves all that for me." This is the most human moment — the guy who ran GitHub, who can get any meeting in Silicon Valley, admitting the tools don't exist yet. It's the counterweight to every "AI will eat everything" narrative.

**The Homer Simpson car is a real warning dressed as a joke.** Auto-merge + auto-deploy + agent-generated PRs = feature explosion without curation. Thomas names the problem but neither speaker has a solution. The bottleneck shifts from "can we build it" to "should we build it," and product management as a discipline isn't ready. This is the missing session from [[The Pragmatic Summit]] — nobody is talking about how product management changes when the production constraint vanishes.

**"Don't be a manager" is both good advice and a self-serving thing for a CTO to say.** Rajeev's argument — engineers should stay engineers, agents make IC work more rewarding, managers will have 50 direct reports and get squeezed — is internally consistent. But it's also the CTO of a company with thousands of engineers telling them not to seek the role he holds. The career-ladder question is real and neither speaker sketches what advancement looks like for a "master of agents" who doesn't manage people.

**Token cost inversion is a genuinely novel economic problem.** Thomas flags it and moves on, but this deserves more attention. For 50 years, "more productive developer" was unambiguously good. Now it has a line item. A developer who burns $5,000/month in tokens but ships 3× more is probably still net-positive, but the accounting category is new and the incentives are weird. Nobody wants to be the manager who says "please code slower to save on tokens." This is going to produce some spectacular organizational dysfunction before it gets solved.

**The junior engineer question is the largest unaddressed problem.** If seniors become "masters of agents" who verify rather than read code, what happens to the pipeline that produces seniors? Learning to code by reading generated code is like learning carpentry by watching robots build houses. The talk celebrates the democratization of building — PMs, marketers, assistants using Replit — but doesn't engage with the skill-atrophy risk. [[The Joy and Power of Understanding]] makes the case that struggle is necessary for mastery; this talk ignores it entirely.

---

## Related Pages

- [[The Pragmatic Summit]] — Event hub: all sessions from the February 2026 conference
- [[Running an AI-Native Engineering Org]] — Fiona Fung's parallel field report from leading Claude Code engineering
- [[Specifications as the Product]] — Specs as the durable artifact when code is disposable
- [[The End of Code Review]] — Verification over inspection, the academic case
- [[Agent Memory and Context]] — Context engineering as the real challenge; the teamwork graph is an instance
- [[Uber — Agentic Engineering Shift]] — Enterprise agentic transformation at comparable scale
- [[Nicole Forsgren on AI and Developer Productivity]] — Bottleneck shift from inner loop to outer loop
- [[Product-Minded Engineers in an AI-Native World]] — Role collapse from the PM/designer/engineer panel
- [[Writing Code vs. Shipping Code]] — The metrics skepticism: PR count ≠ shipped value
- [[Martin Fowler and Kent Beck on Reinventing Software]] — The "re-soloing" pattern and AI as amplifier
- [[Software Engineering at the Tipping Point]] — AI as 10× amplifier, shared fate
- [[Building When It Feels Like There's Nothing Left to Build]] — Joy as the answer when AI can build anything describable
- [[The Joy and Power of Understanding]] — Mastery requires struggle; the junior engineer problem
- [[The Founder's Playbook]] — AI-native startup playbook; greenfield bias
- [[Vibe Coding as a Team Sport]] — Verification workflows, the Kasparov insight
- [[Automating Myself Out of Development]] — Bottleneck migration from coding to review
- [[Agent Coding Workflow]] — Practitioner's daily loop: maturity spectrum from vibes to compound engineering
- [[Guardrails and Feedback Loops]] — The evaluation track the summit was missing

---

*Sources: [[raw/ytx-building-world-class-engineering-teams-age-of-ai]]*
*Last updated: 2026-07-04*
