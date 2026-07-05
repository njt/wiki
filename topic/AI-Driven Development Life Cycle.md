# AI-Driven Development Life Cycle

AWS's Raja SP proposes a full-SDLC methodology that treats AI as a "central collaborator," not a bolt-on assistant. The AI-Driven Development Lifecycle (AI-DLC) replaces Agile terminology (sprints become "bolts" measured in hours/days, epics become "units of work") and restructures development into three phases — Inception, Construction, Operations — with structured human checkpoints at each boundary. The core bet: AI generates plans, seeks clarification, and implements them; humans make only the critical decisions that require business context.

---

## Key Quotes

> "AI creates plans, seeks clarification, & implements plans, while humans make critical decisions"

This is the article's north star, and it's the cleanest articulation of the middle ground between "AI as autocomplete" and "AI as autonomous developer." The emphasis on AI *seeking clarification* is the interesting bit — it positions the human as decision-maker, not reviewer.

> "Only humans possess the contextual understanding and knowledge of business requirements"

A claim that will be tested hard as context windows expand and agents accumulate organizational memory. Today it's true. Whether it holds at 10M-token contexts with multi-year persistence is an open question.

> "This shift from isolated work to high-energy teamwork accelerates innovation and delivery"

The article's vision of "mob elaboration" and "mob construction" reframes pair programming as team-scale AI validation sessions. Whether this is high-energy teamwork or high-overhead meeting culture depends entirely on execution.

---

## Key Themes

- #concept **AI as collaborator, not tool** — The thesis that AI belongs at the center of the SDLC, not at the edges. This goes further than most industry thinking, which still treats AI as an accelerator for existing processes rather than a reason to redesign them.

- #pattern **Human checkpoints, not human review** — AI does the work; humans validate at gated boundaries (inception, construction, operations). Similar to the planner/worker/judge pattern emerging independently in [[Scaling Long-Running Agents]] and [[maestro]].

- #concept **Persistent context compounding** — Each phase produces artifacts that feed the next. The AI doesn't reset between phases; context accumulates. This is the methodology-level version of what [[Planning With Files]] and [[Agent Memory and Context]] tackle at the session level.

- #tool **Amazon Q Developer** — The implementation vehicle. AI-DLC is explicitly tied to AWS's toolchain, which raises the obvious question: methodology or product marketing?

- #pattern **Terminology as theology** — Replacing "sprint" with "bolt" and "epic" with "unit of work" is a classic Consultant Move: rename things to claim novelty. But names do shape behavior, and Agile's terminology carries baggage that may not serve AI-native workflows.

---

## Critical Analysis

**What's useful:** The three-phase structure with explicit human gates is genuinely practical. It maps cleanly onto how teams actually use coding agents today — generate, review, deploy — and formalizes something that's currently ad-hoc in most shops. The emphasis on AI *asking* for clarification rather than just executing is under-discussed in the agent literature and deserves attention.

**What's suspicious:** This reads like AWS consulting marketing. The author leads "Developer Transformation Programs" and cites "100+ large customers." The methodology is tightly coupled to Amazon Q Developer and references a "white paper" — the classic enterprise sales funnel. That doesn't make it wrong, but it explains the ambition-to-evidence ratio. Where are the case studies? The metrics? The teams that adopted this and measured results?

**What's missing:** Any honest discussion of when this breaks. What happens when the AI hallucinates during inception and the team doesn't catch it until construction? What's the rollback mechanism? The article describes a happy path where AI generates good plans, humans validate them, and context compounds beneficially. Real projects have bad plans, confused AI, and compounding errors. The [[Compound Engineering]] answer — "add a system, not manual review" — is notably absent.

**The Agile comparison problem:** AI-DLC defines itself against Agile, but it inherits Agile's weakest property: it's a prescriptive methodology sold as universal. The wiki already documents practitioners who get results with wildly different approaches — [[Ralph]]'s bare while-loop, [[TextForge Case Study]]'s six-layer discipline, [[Recursive Mode]]'s phase-gated artifacts. One-size methodologies have a poor track record in software, and AI-native development is too young for orthodoxy.

**Who this is for:** Enterprise teams with existing AWS relationships who need a structured framework to sell AI adoption upward. The methodology provides cover — "we're not just letting AI write code, we're following AI-DLC" — which has real organizational value independent of whether the methodology itself is optimal.

---

*Sources: [[summary/ai-driven-development-life-cycle]]*
*Last updated: 2026-05-14*
