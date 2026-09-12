# Code-First Developer

Khalil Stemmler's diagnosis of the earliest phase in his five-stage model of developer craft: the developer who can write code but hasn't yet learned to design, test, architect, or strategize — and the path out.

---

## The 5 Phases of Craftship

Stemmler proposes a developmental ladder for software developers, each phase representing "a distinct shift in your value system as a developer, how you think about code, and how you decompose and solve problems":

1. **Code-First** — can write code, hasn't learned craft
2. **Best Practice-First** — cargo-cults rules without understanding why
3. **Pattern-First** — recognizes recurring structures, applies design patterns
4. **Responsibility-First** — thinks in terms of what code is *for*, not what it *is*
5. **Value-First** — measures every decision against the 3 roles (customers, developers, users)

The four pillars every developer needs are **design, testing, architecture, and strategy** — and they all serve the goal of creating either *TimeGivenBack* (saved time) or *TimeEnriched* (better experience).

## The Code-First Trap

> "The Code-First developer has learned to code but has yet to learn to craft."

This phase is characterized by asking the wrong questions: *How do I write E2E tests that aren't flaky? How should I write unit tests? How should I organize code so it's scalable?* These are tool questions, not craft questions. The underlying issue is treating code as the end product rather than the medium.

Stemmler opens with a story about a colleague named Sean who introduced him to Domain-Driven Design and wrote C# so expressive that "I found myself grasping the essence of the domain simply by examining his code." The contrast is deliberate: Code-First developers produce code you have to reverse-engineer to understand; craft-level developers produce code that *teaches* you the domain.

## The Expert Junior Developer

> "Taking a breadth-first approach keeps you stuck at the surface."

The Expert Junior Developer is someone with years of experience who keeps cycling through frameworks and tools without ever going deep on fundamentals. They've spent five years learning React, then Vue, then Svelte — but can't design a system, can't reason about coupling, can't name the tradeoffs. This is the trap that breadth-first learning creates: surface fluency without depth.

Stemmler's advice is to go *depth-first* on a single stack: "Build an information system from end to end, making all the mistakes." Frontend *and* backend, for a real human user.

## The Value Differential & AI

> "The value of a single full-stack Pattern-First Developer with access to ChatGPT became greater than 5+ frontend developers all combined."

This is Stemmler's sharpest claim and the one that's aged best. He observed — pre-GPT-4, even — that AI tools were about to make code-generation ability a commodity and shift value toward people who could think in systems, not syntax. The "Value Differential" is the gap between what the market will pay for pure coding skill versus what it will pay for architectural judgment.

This connects directly to [[Writing Code vs. Shipping Code]], which finds empirically that AI-driven commit gains attenuate from 180% to 30% at the release level: writing code got cheap, but shipping software didn't. [[DDD Matters More When AI Writes Your Code]] makes the same bet from the modeling side: when AI writes the code, the scarce skill shifts from syntax to understanding the domain well enough to guide the agent — DDD's knowledge crunching, not framework expertise.

## Burnout and the Mastery Thesis

> "Burnout is when the level of effort and energy you spend doesn't match the level of fulfillment you should experience."

Stemmler's remedy for burnout is specific: pursue mastery, not novelty. Framework-hopping is exhausting because you're always a beginner. Depth work is energizing because you're always building on prior understanding. "Mastery is truly recession-proof" is the takeaway — but also: "Do less, more."

This echoes [[The Joy and Power of Understanding]] — Roztropiński's argument that LLMs are force multipliers but you must have force first, and struggle is necessary for mastery.

## The Path Forward

Stemmler's four-step prescription:

1. **Shift focus to Responsibilities** — pay attention to what your tools handle for you, not just how to use them
2. **Commit to fullstack** — you can't understand value if you only see half the system
3. **Build end-to-end for a real user** — the gap between tutorial projects and production systems is the whole game
4. **Zoom out** — examine what your tools are *really* for, not just how to operate them

## Critical Analysis

Stemmler's framework has real teeth, but it's worth naming what it leaves out. First, the five phases are presented as linear progression, but there's no phase for "Code-First who knows they're Code-First" — self-awareness of your level is a distinct skill and may be more valuable than progressing to the next rung. Someone who knows they're operating at Best Practice-First and compensates with humility and verification might outperform a deluded Value-First developer.

Second, the phases are *individualistic*. There's no "Team-First" phase, no acknowledgment that craft is socially mediated — that the best developer on a toxic team will produce worse work than a competent developer on a healthy one. [[Software Engineering at the Tipping Point]] makes the same point from the platform angle: every node in your ecosystem breaks at 10× scale.

Third, "Value-First" as the apex is a business-value framing that sidesteps whether some code is worth writing *because it's beautiful or interesting*. Stemmler would probably argue that beautiful code inherently serves the developer role (TimeEnriched), but the framework doesn't make space for craft as an intrinsic good. [[The Mundanity of Excellence]] complicates this: excellence isn't more effort, it's qualitatively different choices — and those choices include aesthetic ones.

That said, Stemmler's framework is genuinely useful as a diagnostic tool. The Expert Junior Developer is a real phenomenon. The burnout-mastery link is under-discussed. And the Value Differential has only grown since he wrote this in early 2024.

---

*Sources: [[summary/code-first-developer]]*
*Last updated: 2026-07-05*
