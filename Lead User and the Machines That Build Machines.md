# Lead User and the Machines That Build Machines

Brad Feld extends Eric von Hippel's 48-year research program on user-driven innovation into the age of AI coding agents, arguing that the boundary between user and manufacturer is dissolving into a prompt. His core claim: all future end-user software innovation will come from the user, with machines acting as the manufacturer.

---

## Key Quotes

> "Innovation comes from users, not manufacturers."

Von Hippel's original provocation from ~1978, which Feld reminds us was genuinely radical at the time. The assumption then was that manufacturers invented and users consumed. Von Hippel spent nearly five decades proving the opposite. Feld's piece is, at one level, an attempt to explain why this old idea is suddenly the organizing principle for the next era of software.

> "The user has the need and the sticky information. The machine has the ability to write, test, and ship the software."

This is the cleanest articulation of the division of labor Feld is proposing. **Sticky information** — von Hippel's term for knowledge that's expensive to transfer because it's context-dependent and tacit — stays with the user. The machine handles execution. This is not the same as "AI replaces programmers." It's "the user who knows the problem becomes the programmer, and the machine is the compiler."

> "The manufacturer is becoming a machine that the user operates."

Feld spent his doctoral years studying the boundary between user and manufacturer. His conclusion, 40 years later: the boundary dissolves because the manufacturer stops being a person or organization and becomes software. This is a more precise framing than most "dark factory" rhetoric — it's not that humans are removed from the loop, it's that the *manufacturer role* is automated. The user remains.

> "The boundary between us was a prompt."

Describing his experience building a plugin he needed — *The First User* pattern, a riff on von Hippel's Lead User. He didn't build it for a market; he built it because he had the need and the sticky information. The machine built it because he could describe what he wanted. This is [[Intent Is the Interface]] made personal.

> "I use the word 'machine' instead of 'agent.' Agent is a meaningless buzzword that means everything and nothing."

Feld's terminological choice is deliberate and worth taking seriously. A machine has defined inputs, outputs, and a job. It forces specificity about what something *does* rather than what it *is*. In a field drowning in vague terminology, this is a useful discipline. It also connects to the manufacturing metaphor that runs through dark factory discourse.

> "Be obsessed about your competitors' products and what they do and then completely ignore them and play your own game."

A venture capitalist's koan. The tension between competitive awareness and strategic independence is the whole point — you need both, held in productive contradiction.

## Key Themes

- #concept — **Lead User theory**: innovation originates with users who face needs before the market, not with manufacturers who serve existing demand
- #concept — **Sticky information**: the knowledge needed to solve problems is expensive to move and resides with the user; this is *why* user-driven innovation works
- #concept — **The First User**: Feld's extension — when the manufacturer is a machine you operate via prompts, *every* user becomes a potential Lead User
- #pattern — **Machines, not agents**: naming as discipline; define inputs, outputs, and the job, and the architecture follows
- #concept — **Theory of Constraints** (Goldratt): find the constraint, fix it, repeat — Feld's operating system for deciding where to focus across his Three Machines (Product, Customer, Company)
- #person — **Eric von Hippel**, MIT professor who built the academic case for user-driven innovation over 48 years
- #person — **Brad Feld**, VC (Foundry Group) and von Hippel student, now building at intensitymagic.com
- #comparison — The user→machine→software pipeline vs. the traditional manufacturer→product→user pipeline

## Critical Analysis

**The "all" in "all future end-user software innovation" is doing a lot of work.** Feld means it — not most, *all*. This is a stronger claim than von Hippel ever made (von Hippel argued users were a *major* source of innovation, not the exclusive one). Feld's argument works for the category of software he seems to have in mind: tools, plugins, utilities, automations — things individual users need and can describe. It's less obviously true for infrastructure software, developer tools, or systems where the user's sticky information isn't sufficient to specify the solution. You can't prompt your way to a database engine unless you understand query optimization.

**The fuzziness around "end-user" matters.** Feld's own example (a plugin) sits at the boundary — he's a sophisticated technical user, not a generic end-user. The Lead User concept was always about *leading-edge* users, not average ones. The question Feld doesn't answer is how far down the expertise curve this pattern extends. If you need to be a von Hippel "Lead User" to describe what you want precisely enough for a machine to build it, the revolution is real but narrow.

**"Agent is a meaningless buzzword" is fighting words, and mostly right.** Feld's objection isn't just semantic crankiness. The term "agent" has been stretched to cover everything from autocomplete to autonomous organizations, which means it communicates almost nothing. His insistence on "machine" — defined inputs, outputs, job — is a useful corrective. But it also sidesteps the autonomy question. Machines don't decide *what* to do; agents arguably do. The distinction matters for architectures where the system needs to make choices the user didn't anticipate. Feld's framing works beautifully for "user specifies, machine executes." It's less clear for "machine identifies problems the user didn't know they had."

**The connection to dark factory discourse is underdeveloped.** Feld name-checks Dan Shapiro and the Software Factory / Dark Factory spectrum but doesn't engage with the tensions those concepts raise. [[Don't Fear the Dark Factory]] argues the harness is the factory. [[The Dark Factory is a DOT File]] argues the pipeline spec is the durable artifact. Feld is making a different argument — about *who* drives innovation, not *how* the factory works — but the two arguments need to be in conversation. If the user provides sticky information and the machine executes, what does the harness look like? Who validates the output? Feld doesn't say.

**What's genuinely new here:** Feld connects user-innovation theory (a 50-year academic tradition) to the current moment in a way nobody else has. Most dark factory writing is engineering-forward — it starts from the technology and asks what it enables. Feld starts from the economics of information (sticky knowledge, user needs) and arrives at the same place from the opposite direction. The convergence is what makes the argument interesting. Two independent lines of reasoning — one from innovation studies, one from software engineering practice — pointing at the same conclusion: the user-manufacturer boundary is collapsing.

**The Three Machines framework deserves more air.** Feld mentions it in passing — Product, Customer, Company as three machines, with Theory of Constraints as the focusing mechanism — but it's clearly where his practical energy is going (intensitymagic.com). This is the bridge between the intellectual argument (von Hippel) and the operational question (how do you actually build a company this way?). The fact that he's six months into building rather than blogging suggests he's serious about answering it.

**This is a speculative essay from a practitioner, not a research paper.** Feld is clear about this — he's "pondering" a question he's thought about for 40 years, not claiming to have proven anything. The essay's value is in the synthesis: connecting von Hippel's user-innovation framework to the machine-as-manufacturer moment, and naming the pattern (The First User) while he's living it. Worth reading alongside [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] for the Shapiro framework Feld references, and [[Lean Software Production]] for the post-craft manufacturing metaphor taken seriously.

---

*Sources: [[raw/brad-feld-lead-user-machines]]*
*Last updated: 2026-07-05*
