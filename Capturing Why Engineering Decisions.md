# Capturing Why Engineering Decisions

A 64-comment HN thread on the eternal problem of capturing *why* engineering decisions were made, not just *what* was decided. The OP tried ADRs, PR templates, and a Notion doc — all failed because "every solution requires someone to manually write something. Nobody does." The thread is half genuine practitioner wisdom and half stealth product pitch, but the practitioner half is excellent.

---

## Key Quotes and Arguments

### Documentation survives when it lives next to the code

> "documentation survives when it lives next to the code" — lowenbjer

ADRs and Confluence die because they're separate from the code. File-level headers, good READMEs, and per-folder documentation work for humans, LLMs, and search. This is the thread's central thesis and it's correct — it's the same reason [[LLM Wiki]] and [[Wuphf — Karpathy-Style Agent Wiki]] put knowledge in markdown next to the repo rather than in a separate tool.

lowenbjer also described an LLM-as-judge git hook that checks PRs for consistency with existing docs and blocks merges if updates are needed. This is [[claude-ctrl]] logic applied to documentation: enforcement, not suggestion.

### The code is the only thing you can trust

> "the code is the only thing I can trust to be there" — al_borland

Through platform migrations and tooling changes, code comments survive. Jira tickets don't. Confluence pages don't. Slack threads certainly don't. This is the [[Long Live Systems of Record]] argument applied to decision rationale: where does the truth live?

al_borland uses template code comments for "why" information: "This previously used X, but moved to Y because Z" or "This is ugly because the clean way doesn't work due to W." Not glamorous, but it survives.

### Write it for yourself, not for posterity

> "I do it mostly for me because I find it invaluable as I prefer writing shit down instead of relying on my flaky memory." — hysan

hysan writes lengthy PR descriptions with collapsed sections — not for the team, but for their future self. This is the thread's most underrated insight: documentation survives when the author is the primary beneficiary. Altruistic documentation ("write this for the next person") dies. Self-interested documentation ("write this so I can search it later") lives. The same dynamic explains why al_borland's code comment templates and physicles's rude Q&A doc actually get maintained.

### Busy engineers will do the easiest thing

> "a busy engineer trying to hit a deadline is just going to do the easiest thing" — hermitcrab

The OP's product idea — passively extract rationale from PRs/Slack/tickets and auto-draft docs for one-click approval — runs into this wall. hermitcrab worked on design rationale recording 25 years ago and identified three reasons people don't document *why*: career security, legal liability, and time cost. The first two are under-discussed in the thread but possibly the most important.

### For the first time, good docs pay dividends

> physicles: LLMs love reading good docs and keeping them current

This is the novel economic argument in the thread. Before LLMs, documentation was a pure cost center with deferred and uncertain payoff. Now it's training data for the tools that write your code. physicles maintains a personal "rude Q&A document" answering questions like "Why Kafka?" as a self-reminder — and as context for LLMs. This inverts the documentation economics entirely, similar to how [[Specifications as the Product]] inverts the code/spec relationship.

### ADRs are point-in-time records, not living documents

> soniclettuce: "you don't update ADRs, you write new ones"

Multiple commenters converged on this: ADRs are RFCs, not wiki pages. They capture the decision *at the time* with the context *at the time*. You don't maintain them; you supersede them. This is the IETF model (nonameiguess) and it works because it removes the maintenance burden that kills most documentation efforts.

### New hires are the best documenters

> soniclettuce: one workplace had new hires document all their onboarding questions/answers, which quickly fixed incorrect docs

> physicles: "new hires are in the best position to update docs because they remember what it's like not to know"

This is a [[Fresh Eyes]] argument applied to documentation: the person who just struggled through the gaps is best positioned to fill them. But it only works if the culture rewards it (lwhsiao: "hire people that value writing").

### Chesterton's Fence and the danger of removal

> sdeframond: sometimes the way to understand a fence is to remove it and see what happens

> 4star3star: "it's a lot harder to notice when valid data DISAPPEARS" than when erroneous data appears

The Chesterton's Fence exchange is one of the thread's best micro-debates. sdeframond argues that many decisions have no real reason and you can just test by removal. 4star3star counters that the failure mode of removing something without understanding it is subtle — data silently vanishing — and much harder to catch than a crash or an error. Both are right. The takeaway isn't "never remove things" but "when you remove something you don't understand, write a test for the property you think doesn't matter."

### Sometimes there is no good reason

> wesselbindt: "the dev had a hammer and the codebase was starting to look an awful lot like a nail"

> rich_sasha: "many decisions happen for no good reason, by accident, or for outdated reasons"

The thread's most honest contribution. Not every decision has a satisfying rationale. Sometimes Redis was chosen because the dev wanted Redis on their resume. Documenting the *absence* of a reason is itself valuable — it tells the next person "you can change this."

### LLMs can extract rationale from what already exists

> iSnow: built an agentic framework that distills ADRs from transcribed Teams meetings, "recording the WHY without someone having to do the job." Works "surprisingly well."

> andrewf: LLMs might be good at answering questions from an unorganized mass of timestamped data — "all the stuff that exists anyway without any extra continuous effort"

Multiple commenters are already building the OP's product idea, and some report it works. The key insight: the conversation that led to the decision already happened — in Slack, in PR comments, in meeting transcripts. The problem isn't capturing new information; it's extracting structure from existing information that's already scattered across platforms. This is a fundamentally different problem than "get engineers to write more prose," and LLMs are good at it.

---

## Key Themes

- `#pattern` Documentation-as-code: keep rationale in the repo, not in a separate tool
- `#concept` ADRs as point-in-time RFCs, not living documents
- `#concept` The documentation economics inversion: LLMs make docs pay dividends
- `#tool` LLM-assisted documentation extraction and enforcement
- `#pattern` New hires as documentation force-multipliers
- `#concept` Career security and liability as blockers to honest rationale

---

## Critical Analysis

The thread is better than its framing. The OP's post is a transparent setup for a product pitch (sph calls it directly: "Cut to the chase, what are you selling?"), but the HN crowd responded with genuine craft wisdom that's far more valuable than the product being warmed up for.

**The best idea nobody's acting on:** hermitcrab's observation that people don't document *why* partly because it reduces career security and opens them to prosecution. This is the real obstacle and nobody's product addresses it. You can't automate away the incentive to be the only person who understands a system. The solution isn't tooling — it's culture and employment contracts.

**The ADR consensus is useful:** ADRs work when treated as point-in-time records you supersede, not living documents you maintain. The maintenance burden is what kills documentation efforts, and the IETF RFC model sidesteps it entirely. This should be more widely adopted.

**The LLM inversion is real but dangerous:** physicles is right that LLMs make good docs valuable in a new way, but this also creates a perverse incentive to write documentation *for the LLM* rather than for humans. If docs become LLM training data first and human reference second, you get the [[Creative Firewall]] problem in reverse — optimized for the machine, degraded for the person.

**The self-interest insight:** hysan's comment reveals a dynamic the OP's product framing misses entirely. When documentation's primary beneficiary is the author (future-self search), it gets written. When it's altruistic (the next hire), it doesn't. Tooling should design for the self-interested case first.

**The NASA point:** actionfromafar notes this whole approach only works when "why" is an actual required deliverable. In organizations where decisions are tracked as compliance artifacts (NASA, nuclear, medical devices), rationale documentation already exists. For everyone else, the question is whether LLM extraction can make it cheap enough to become *de facto* required.

**The thread's blind spot:** Nobody discusses what happens when the rationale is "we made a bad call." Engineering cultures that can't admit mistakes produce documentation that lies. The first requirement for capturing real *why* is psychological safety, and no git hook provides that.

**What actually works**, synthesized from the thread:

1. Code comments for local "why" (al_borland's templates)
2. ADRs as point-in-time RFCs for architectural decisions (nonameiguess, soniclettuce)
3. New hires document their onboarding discoveries (soniclettuce, physicles)
4. LLMs as documentation linters and enforcers (lowenbjer)
5. Culture of writing as a hiring filter (lwhsiao)
6. Commit messages as breadcrumb trails (hammadfauz, Willamin)
7. LLM extraction from existing artifacts — meetings, Slack, PR comments (iSnow, andrewf, pxue)
8. Self-interested documentation: write for your future self, not for posterity (hysan, al_borland)

What doesn't work: any system that requires someone to voluntarily open a separate tool and write prose for an audience they'll never meet.

---

*Sources: [[raw/hn-capturing-why-engineering-decisions]]*
*Last updated: 2026-05-22*
