# Event Storming for Fun and Profit — Daniel Terhorst-North (Craft 2025)

Dan North's Craft 2025 talk on event storming, delivered as a live demonstration of its own thesis: the talk was planned by event-storming itself, so its structure is a Disney-formula narrative ("once upon a time… until finally"). The technique is presented as collaborative story-building whose goal is shared knowledge — "everybody knows what everyone knows" — with applications to business processes, legacy systems, and greenfield design, plus a substantial facilitation-and-psychology toolkit.

---

## What It Argues

Event storming (Alberto Brandolini's technique) is narrative construction, not documentation. Every storm follows the Disney movie formula — initial status quo, "one day," "because of this," "until finally," new status quo — and you start at the end: put "until finally" on the wall first, then fill time flowing left to right, with spare space on both sides "because you're going to discover… you shouldn't have started here."

The mechanics are trivial on purpose: orange stickies for domain events in the past tense, stimuli (commands, external events, time events like "the first of the month happened"), pink stickies for questions and puzzles placed diagonally so they stand out, and view/read models. Who to invite: "people with questions, people with answers, and someone with stationery." The stop condition is epistemic, not artifact-shaped: everyone arrives knowing one part of the elephant; the storm ends when everybody knows what everyone knows.

Three applications give the talk its spine:

- **Business processes** — strip the vestigial steps ("we do it because we've always done it that way") *before* automating, because automation does two things: makes a process deterministic, and calcifies it. Automate first and you bake in work "no one knows why. It's just always been there."
- **Legacy systems** — the first dump is suspiciously fast because people are reciting the data model, not the behavior. North overlays a data-flow diagram to prove it, then calls the required decompression phase "clearing your throat." The quarry is Feathers' seams.
- **New applications** — when pink stickies outnumber orange, "none of us knew." The deliverable is not a design; it is "let's pay down some of this uncertainty" and a reconvene date. Skip this and you get "the usual ball of mud."

The signature story: a trading firm provisioning servers in 30 days — an eternity when a trading opportunity's window is shorter than the lead time. Once the whole procurement chain was on a wall, the room re-sequenced it and parallelized steps (including CIO pre-approval) for 12 days; then the data-center guy noticed there are really only three server types — "money makers," dev machines, "donkeys" — and a few thousand dollars of pre-stocked hardware took it to hours. "We haven't done anything." No new technology; just the first end-to-end view the process had ever had.

## Key Quotes

> "This is a talk about a talk. In fact, it's a talk about this talk."

The opening frame, and the strongest move in the talk: the slides *are* the wall of stickies. If event storming is story-building, the only honest way to teach it is to build one in front of the audience.

> "We want everybody to know what everyone knows."

The goal in one line — shared knowledge as the product, which is why the output ("a bunch of post-its on a wall") matters less than the room.

> "The second thing you do is you bake it, you calcify it."

Automation's dirty secret, stated as a trade-off rather than a gotcha: determinism is bought by freezing whatever the process happened to be, vestigial steps included. This is the single most reusable idea in the talk for anyone about to write a workflow engine.

> "They weren't telling me how the system worked. They're telling me exactly what the data model was."

The diagnostic insight behind "clearing your throat." People locked inside a system for years can only emit its schema; you have to let them get that out of their system before behavior becomes visible.

> "The work was not, let's design a system. The work was, let's pay down some of this uncertainty."

Greenfield storms that produce mostly pink stickies are *succeeding*, not failing. A wall of questions is a research backlog, and mistaking it for a failed design session is how teams end up building on assumed knowledge.

> "There's only one truth, there's only one reality. So when people are in conflict, it's because they have different views on that single reality."

Goldratt's "universal harmony" as a facilitation instrument: find where the parties last agreed, work forward from the divergence. Conflict is data about where world models split.

> "There's arguing to be right and there's arguing to learn. And generally we're very, very good at the former."

Paired with "listening, waiting to speak" versus "listening like you don't know the answer." The facilitator's stance, and the talk's most quotable moral.

> "Everyone is trying to help."

Satir's positive intent — immediately qualified by North himself: "people think positive intent means everyone's nice and it doesn't because some people are a holes." The empathy question is "what must be true for them such that this behavior is the helpful thing to do?" — and the saboteur's answer is usually "they believe you are going to harm the organization." The concession that sociopaths exist, "often in leadership roles," is delivered deadpan and is the honest limit of the framework.

> "Trust Me Once… If it doesn't work, we'll never do it again. And all it cost you is a morning."

His adoption model: lower the safety barrier by framing any new practice (event storming, pair programming) as a cheap reversible experiment. For genuinely closed-minded bosses: Grace Hopper's forgiveness over permission — "JFDI. Just flipping do it."

> "I've never seen this process laid out end to end before."

The head of infrastructure, about his own process. Played for laughs, but it is the whole thesis in one sentence: the intervention was visibility.

## Key Themes

#concept Shared knowledge as the deliverable · #concept narrative construction (the Disney formula) · #pattern visualization alone is the intervention · #pattern clearing your throat · #pattern Trust Me Once · #tool event storming (Brandolini) · #tool Pomodoro + photos + OCR for searchable walls · #person Daniel Terhorst-North · #person Alberto Brandolini · #person Virginia Satir · #person Eli Goldratt · #person Mike Feathers · #person Grace Hopper

## Analysis

The meta-structure is the argument. North could have lectured about story-building; instead the talk *is* an event-stormed artifact, which makes the Disney-formula claim self-evidencing. It also quietly demonstrates the technique's most underappreciated property: a well-formed narrative exposes gaps ("what am I forgetting?") that a slide deck hides.

The strongest claim — visualization alone is the intervention — is well-chosen but rests on one anecdote. The 30-days-to-hours server story is the kind of result that sells the technique, and it is honest about the mechanism (re-sequencing, pre-approval, pre-stocking; nothing exotic). But it is one win, and the gist's own digest notices that North offers no failure modes, no counterexamples, and no way to quantify value for a skeptical CFO beyond "trust me once." A talk this confident about "one of the most self working techniques I've ever come across" should survive contact with a storm that flopped; we never hear about one.

That confidence contains a contradiction worth naming. If the technique were self-working, it would not need the archetype playbook — disruptor, null, wallflower, helper, last-word person, surprise star — or the Satir/Goldratt psychological apparatus. What North actually demonstrates is that event storming is cheap to *run* and expensive to *facilitate well*; the stationery is the trivial part, the group therapy is the craft. He half-admits it: "You really do. You have to at least understand the psychology of the group… I'm an amateur at this. Go talk to Kat Hicks."

The unanswered questions are real and the digest lists them fairly. Remote-first teams get a shrug ("the energy is not nearly the same") from someone who is simultaneously "a massive fan of remote work" for inclusivity — a tension named, never resolved. The path from stickies to aggregates, process managers, and code gets "it depends." Nobody owns the pink stickies between sessions. And the politics of process surgery go unexamined: the CIO sign-off that quietly became "pre-approval" removed a gate that presumably existed for a reason, and the head of infrastructure never having seen his own process end to end is treated as a joke rather than the governance red flag it is. Stripping steps from a process in front of the people whose status those steps encode is exactly where this technique most needs facilitation skill — which loops back to the contradiction above.

Still, the durable contributions here are portable well beyond event storming: strip before you calcify; let the data-model recital happen before asking how things actually work; treat a wall of questions as an uncertainty-paydown plan, not a failed design; treat conflict as divergent views of one reality; and price adoption in mornings, not migrations.

## Related Pages

- [[Domain Storytelling]] — the sibling collaborative-modeling method; that page already carries an explicit Event Storming comparison table (no timeline, actor-cooperation focus), and North's talk supplies the practitioner mechanics and facilitation depth that comparison gestures at.
- [[DDD Matters More When AI Writes Your Code]] — argues domain modeling gains value as code generation gets cheap; event storming is the discovery practice that feeds DDD aggregates and bounded contexts, and North's "clusters reveal cohesive subsystems" is that argument's front end.
- [[Evolutionary Architecture — Maciej Jedrzejewski (Craft Budapest)]] — uses event storming in its chapter-one case study to surface commands, actors, and policies before pushing mutation logic inside objects; North's talk is the full method behind that one step.
- [[Essentials from a Real-World Microservices Journey — Sander Hoogendoorn (Craft 2025)]] — same conference, and Hoogendoorn's no-ceremonies way of working cites event storming among the few practices he kept; North explains why it survives when sprints and retrospectives don't.

---
*Sources: [[raw/event-storming-for-fun-and-profit-daniel-terhorst-north-craft-2025]], [[summary/event-storming-for-fun-and-profit-daniel-terhorst-north-craft-2025]]*
*Last updated: 2026-09-13*
