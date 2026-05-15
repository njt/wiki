# The Pragmatic Summit

Gergely Orosz's inaugural one-day curated conference for senior engineers and engineering leaders, held February 11, 2026 in San Francisco. Application-based admission, ~400 attendees, $499. Single-track main stage with three parallel breakout tracks. 28 speakers — every one a practitioner building real products, not a thought leader or vendor evangelist. Sponsored by Statsig and Linear.

The conference is structured around one question: what actually changes when AI enters the engineering org? The program splits into main-stage keynotes on reshaping craft, data vs. hype, and org design, then three breakout tracks: Lessons from Building (Cursor, Vercel, Ramp), Frameworks for (Fowler & Beck, Simon Willison, Chip Huyen), and Leading (product engineering, high-performing teams, Uber's agentic shift).

---

## The Sessions

The main stage carries the editorial argument. Each session is a thesis statement.

### Main Stage

**How AI is Reshaping the Craft of Building Software** — Vijaye Raji (OpenAI CTO Applications), Tibo Sottiaux (Codex Head of Engineering), Gergely Orosz. The anchor session: a grounded conversation on AI coding tools and agent workflows. Raji brings the model-builder's perspective, Sottiaux the agent-harness perspective, Orosz the practitioner's. The inclusion of Codex (not just OpenAI) signals that the agent layer — not the model — is where craft is being reshaped.

**Data vs. Hype: How Orgs Actually Win with AI** — Laura Tacho (CTO, DX). The "show me the numbers" slot. Tacho runs the company that measures developer productivity across organizations. The talk description itself is a thesis: "why outcomes vary so widely across companies." Implicit argument — the variance isn't in tool access, it's in how teams integrate AI into their workflow.

**Building World-Class Engineering Orgs in the Age of AI** — Rajeev Rajan (CTO, Atlassian), Thomas Dohmke (CEO, Entire), Gergely Orosz. The org-design session. The framing is precise: "what changes when the bottleneck shifts from writing code to intent, review, and verification." This echoes the [[Specifications as the Product]] thesis — code generation stops being the constraint, and everything upstream (intent, design) and downstream (review, verification) becomes the new engineering surface.

**Closing Thoughts: Thomas Dohmke on Entire.io** — The exclusive reveal. Dohmke left GitHub's CEO seat to found Entire.io, described as "the world's next developer platform." The summit was his first public discussion of the company, team, and plans. Significant: a former GitHub CEO doesn't build another Git host — he builds something post-GitHub. Worth watching.

### Breakouts: Lessons from Building

**Cursor** — Sualeh Asif (Cursor co-founder) interviewed by Alex Xu and Sahn Lam (ByteByteGo). System design and engineering decisions behind the leading AI code editor. Pairs with [[Scaling Long-Running Agents]] for Cursor's published architecture insights.

**Vercel** — Malte Ubl (CTO) on the evolution of v0 and the engineering of the v0 agent. Ubl is one of the deepest thinkers on agent-native interfaces — see [[json-render]]. Steve Huynh interviews, bringing the practitioner lens.

**Ramp's Policy Agent** — Four Ramp engineers walk through turning an internal AI experiment into a production product. The arc from prototype to production-ready is the hardest part of applied AI; this is the field report. Parallels [[Minions — Stripe's One-Shot Coding Agents]] as a fintech company's internal agent story.

### Breakouts: Frameworks for

**Martin Fowler & Kent Beck** on "Reinventing Software — Again and Again." The two most influential methodologists in software engineering, on the mistakes engineers keep repeating across every step change. Gergely moderates. This is the historical-pattern-recognition session: AI isn't the first transformation, and the failure modes are predictable.

**Simon Willison** on "Engineering Practices that Make Coding Agents Work." The most prominent open-source developer in the AI-tooling space, explaining his hands-on workflow with coding agents. Pairs directly with [[Designing Agentic Loops]], where Willison names the meta-skill of agent wrangling.

**Chip Huyen** on "Building When It Feels Like There's Nothing Left to Build" — moving from AI product decisions to running AI systems in production. Huyen's focus on the infra/ops side of AI is the counterweight to the "agents write all the code" narrative.

### Breakouts: Leading

**Product Engineering Teams in an AI-native World** — Panel with Michelle Lim (Flint co-founder), Tuomas Artman (Linear co-founder), Drew Hoskins (Temporal), Margaret-Ann Seger (Statsig VP Product). The thesis: product engineering matters MORE with AI, not less. Skills leaders should look for shift. Artman's presence connects to [[Managing Agents via Kanban Boards]] — Linear is both tool and philosophy.

**Nicole Forsgren** on high-performing engineering teams in the age of AI. Forsgren wrote the book (literally, *Accelerate*) on measuring engineering effectiveness. The question: do the conditions for high performance change when AI enters the workflow?

**Uber's Agentic Shift: From Tools to Culture** — Ty Smith (Principal Engineer) and Anshu Chadha (Director of Engineering) on building agentic culture across Uber's engineering org. The "from tools to culture" framing is the most honest: tool adoption is the easy part. Culture change is the actual work.

---

## Key Themes

- **#concept** The bottleneck has moved — from writing code to intent, review, and verification. Code generation is solved; everything around it is the new problem.
- **#concept** Practitioner-only speaker lineup — no academics, no pure thought leaders, no vendor evangelists. Everyone ships.
- **#concept** Agentic culture, not agentic tools — Uber's talk title is the clearest expression: the hard shift is cultural, not technical.
- **#pattern** Newsletter-to-conference pipeline — *The Pragmatic Engineer* joins Stratechery and Lenny's Podcast in extending a media brand into live events.
- **#pattern** Three-track breakout structure (Build / Frame / Lead) mirrors the career ladder: ICs build, staff+ frame, EMs/VPs lead. Not an accident.
- **#tool** Entire.io — Thomas Dohmke's post-GitHub developer platform, publicly discussed for the first time at this summit.

---

## Critical Analysis

The program design is the strongest signal. Orosz didn't just book big names — he structured the day around a coherent argument: AI changes craft (main stage), here's what building with it looks like (Lessons from Building), here's how to think about it (Frameworks), here's how to lead through it (Leading). That's editorial discipline most conferences lack and most editors would recognize.

The weakest link is the main stage's corporate heft. Atlassian, OpenAI, and a former GitHub CEO — these are safe, blue-chip choices. The breakout track is where the interesting bets live: Ramp's four-engineer deep-dive on Policy Agent, Chip Huyen in stealth, Simon Willison the open-source indie. The main stage validates the ticket price; the breakouts deliver the signal.

Thomas Dohmke using the summit as the reveal venue for Entire.io is a coup. Conferences compete for exclusives. Getting a former GitHub CEO's first public discussion of his new company positions the summit as a destination for important announcements, not just good talks.

The Ramp session is the most important format choice. Four engineers talking about one product, from experiment to production. That's a case study, not a talk. More conferences should do this — the granularity is what makes it useful. One person giving a 30-minute overview of "how we built X" is a blog post. Four people each owning their layer is a masterclass.

The Fowler/Beck session is either the best or worst slot, depending on delivery. Two legends talking about "mistakes engineers keep repeating" could be the most useful framing session of the day — or it could be the self-congratulatory "we've seen this before" talk that ignores how AI is genuinely different. The moderator (Orosz) matters enormously here.

What's missing: no sessions on evaluation, testing, or quality assurance for AI-generated code. The main stage mentions "review and verification" as the new bottleneck but there's no dedicated session on how to do it. Given how many wiki pages orbit this problem ([[Guardrails and Feedback Loops]], [[Harness Engineering]], [[Compound Engineering]], [[Demystifying Evals for AI Agents]]), its absence from the program is notable. Perhaps year two.

The curation bet — application-based, 400 people — either scales or it doesn't. If year two expands to 800, the hallway quality drops. If it stays at 400, the economics have to come from somewhere else (recordings, sponsors, satellite events). The newsletter's business model doesn't depend on the conference, so staying small is viable. But the pressure to grow is real: a sold-out conference with people turned away is leaving money on the table, and the pragmatic case for capturing that value is in the newsletter's DNA.

---

## Related Pages

- [[Designing Agentic Loops]] — Simon Willison on the meta-skill of agent wrangling
- [[Scaling Long-Running Agents]] — Cursor's published architecture insights
- [[Specifications as the Product]] — the bottleneck-shift thesis this conference builds around
- [[Compound Engineering]] — adding systems rather than manual review; relevant to Ramp's Policy Agent journey
- [[Minions — Stripe's One-Shot Coding Agents]] — parallel fintech agent story
- [[Harness Engineering]] — the engineering theory behind "verification over generation"
- [[Guardrails and Feedback Loops]] — the missing session; what the program should address next year
- [[Managing Agents via Kanban Boards]] — Linear/Artman's philosophy
- [[Agent-Native Architectures (Every)]] — "AI-native" org design principles
- [[ThoughtWorks Future of Software Engineering Retreat]] — the "middle loop" of supervisory engineering
- [[The Next Two Years of Software Engineering]] — labor market implications
- [[Inside OpenAI's In-House Data Agent]] — Codex agent deployment at scale
- [[json-render]] — Vercel's generative UI framework; Malte Ubl's work
- [[Software Engineering Craft]] — Fowler and Beck's domain
- [[The Plan Is the Program]] — intent as the atomic unit when code generation is free
- [[How Intercom Uses Claude Code]] — enterprise agent deployment at comparable scale
- [[Radical Accountability]] — taste is all that's left when AI removes time constraints
- [[AI Coding Tools Create More Bugs Than They Fix]] — the Data vs. Hype session's implied counterpoint

---

*Source: [[raw/pragmatic-summit-2026]]*
*Last updated: 2026-05-15*
