# Inspect at Ramp — Culture Lessons from a Multiplayer Coding Agent (Bruggeman)

Zach Bruggeman's AI Engineer Code 2026 talk is a culture talk wearing a
product's clothes: Ramp built its own multiplayer coding agent, Inspect, which
now writes ~75% of Ramp's shipped code, and the interesting part is not the
agent but the default-public, one-instance-for-everyone way the org adopted
it.

---

## What the talk argues

Inspect is Ramp's in-house collaborative coding agent: every session runs
multiplayer by default, integrates with Slack, Linear, Data Dog and databases,
and includes design primitives, review agents, and verification flows so a
builder goes "from prompt to shipping" inside one tool. Bruggeman explicitly
de-emphasises the technology ("spinning up sandboxes in under 5 seconds") —
that stopped being the differentiator once the concepts spread — and offers
three lessons about culture instead.

**1. Build tools that kill information silos.** The maths of per-engineer
agent setup at a multi-thousand-person company is brutal: 15 minutes each,
multiplied across everyone, is "days lost" — and the setup cost recurs as
best practices move, with knowledge spreading unequally. Ramp's answer is a
single company-wide Inspect that is always a solid baseline. Tweaking your own
setup is allowed, but the baseline is good enough that nobody ever needed a
mandate.

**2. Trust is built in public.** Every session is public by default, so
"it's actually easier to work in public than in private." A PR carries the
whole conversation, tools used, screenshots, even a video of the verification
agent clicking through the app. The centrepiece anecdote: a bug report from
engineer Andrew; Inspect diagnoses the bug as the very telemetry
instrumentation added to debug it; nobody believes it; a second, more specific
pass finds the Hono library calling fetch without reading or cancelling the
response body (a Cloudflare Workers limitation); the patch deploys and errors
stop. The point is not the fix but the thread — before Inspect this would
have been a ticket, a private debug, a "fixed" message. Instead a skeptic
became a collaborator in an hour.

**3. Think bigger together.** Against "reverence for the lone wolf operator":
individual leverage without sharing "is all going to collapse in on itself."
The proof is Review Buddy, a code review agent two Ramp engineers built
unprompted on Inspect's automation triggers — better than the off-the-shelf
review products Ramp kept rotating through, and the push that made the Inspect
team rethink what Inspect could be.

---

## Key quotes

> "We now see around 75% of all of RAMP's code being written and shipped by
> Inspect."

The number is doing a lot of work — with no definition of "written and
shipped" in the talk, treat it as a morale stat rather than a measured one.
Still, coming from an internal builder rather than a vendor marketing deck,
it's notable.

> "We really don't have to run these formal sit-down here's-how-you-use-AI-
> effectively training sessions... it's actually easier to work in public
> than in private."

This is the strongest claim in the talk and the most transferable. Public
sessions as ambient training replaces the onboarding program entirely — and
it only works because the default is public, not optional. You can ask Inspect
to summarise how a teammate structures prompts or breaks down gnarly problems,
which turns colleagues' sessions into a prompt corpus.

> "They can see the exact conversation that you had, all the tools that
> Inspect used, before and after screenshots, or even a video recording of the
> verification agent clicking through the app in the browser to prove its work
> holds up."

Review here is not a diff dropped over a wall — it's evidence. The session is
"an easy to share living artifact of how that change came to be."

> "If we don't build outwards, collaborate with other people... I think this
> is all going to collapse in on itself."

An unusually anti-solo take for a coding-agent talk, and a direct hit on the
lone-genius framing that dominates the discourse.

---

## Themes

#concept #tool #pattern #person

---

## Opinionated take

The talk is smartest where it's least glamorous. The one-shared-instance
decision is quietly radical: most orgs treat agent configuration as a
per-engineer art project, and Bruggeman's argument that this produces
"disparate outcomes across an organization" reframes personal agent tweaking
as a coordination failure, not a skill. The counter-argument — that a
mandate-shaped baseline ossifies and kills the experimentation that finds
better patterns — is half-answered: you may tweak, but the baseline keeps
absorbing the wins.

The Andrew story is the real product demo, and it's honest about the mess:
nobody trusted anybody ("I don't know if Inspect trusts anyone"), the robot's
first diagnosis was dismissed, and the win only landed after a human pushed it
to be more specific. That's a more accurate picture of agent-assisted
debugging than most success stories — trust was earned through the public
argument, not granted up front.

Where the talk is weakest: the 75% claim and the closing pitch
(inspect.com/ieie waitlist) blur internal lesson and external marketing. And
"it's easier to work in public" is a claim that depends heavily on Ramp's
culture of psychological safety; the same default in a blame-prone org reads
as surveillance. The lesson generalises only with that caveat attached.

---

## How this connects

- Strengthens [[Ramp — Lessons from Building a New AI Product]] with the
  cultural half of the same company's story: that note covers single-agent
  architecture and evals; this covers the adoption and trust layer of the very
  same tool.
- Complicates [[Vibe Coding as a Team Sport]]: both argue AI coding is
  collaborative, but Inspect makes the collaboration mechanically unavoidable
  (public-by-default sessions) rather than aspirational.
- Nuances [[The Harness Is the Company]]: Inspect is a concrete instance of a
  harness becoming organisational infrastructure — one instance, one face,
  company-wide identity ("we dress it up on holidays").
- Extends [[Agentic Code Review]] with the Review Buddy origin story: the
  review agent wasn't a platform decision but two engineers' side project
  built on existing automation triggers.

---
*Sources: [[raw/zach-bruggeman]], [[summary/zach-bruggeman]]*
*Last updated: 2026-11-26*
