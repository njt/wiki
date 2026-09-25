# I Don't Want the Details

Michael Heap's short essay on a moment that sounds dismissive but isn't: an SVP interrupts his incident explanation with "I don't want the details — I want to know what we're changing." The essay turns that into a critique of why-did-this-happen postmortems, arguing that understanding an issue is not the same as fixing it, and that empathy for reasonable people can become the organisation's excuse for not changing anything.

---

Heap was dragged into a call with his engineering counterpart's boss — an SVP — after something went wrong. He started explaining how it happened and was cut off: "I know that if we get into the details, the reasons will be perfectly reasonable... Then it'll happen again. So I don't want the details. I want to know what we're changing."

His first reaction was that this sounded dismissive. His realisation is the pivot of the piece: the executive was *presupposing* competence and reasonableness, and skipping straight to the part that matters. Trust, not impatience.

The argument against the conventional postmortem:

> Understanding an issue is not the same as fixing it. A good explanation can make things worse. Once everyone agrees that the behaviour was reasonable, the urgency to change anything disappears.

This is the sharpest claim in the essay. The standard incident ritual — timeline, decision reconstruction, dependency map, "that makes sense" — actively *dissolves* the pressure to change. A blameless postmortem done poorly doesn't just fail to fix things; it launders the failure into folklore and moves on.

The replacement question:

> **What are we changing so that the same class of failure is less likely next time?**

Note the move from *instance* to *class*. "Alice was on holiday and Bob thought the Widgets team owned it" is not a cause; it's a system with ambiguous ownership that happens to fail when someone is away. Same for requirements changing inside the launch window, and for on-call engineers drowned in low-value alerts. Each excuse is really a design specification for a fix.

The test for corrective actions:

> If your corrective action depends on people remembering a conversation from six months ago, you don't have a corrective action. You have organizational folklore.

And the corollary: "If everyone involved in the incident left the company tomorrow, would the fix still work?" That is a memorably brutal durability test — it converts vague commitments ("communicate better", "be more careful") into checkable engineering statements.

He also puts a governor on his own thesis:

> Not every failure deserves a new process. That's how you build environments that no-one wants to work in.

But the escape hatch must be explicit: "We are consciously accepting this risk" versus "we said we'd try harder and everyone felt better." The enemy is not accepted risk; it's unaccountable reassurance.

## Key themes

- #concept — explanation as displacement activity: understanding without changing
- #pattern — class-of-failure framing: turn every incident excuse into a system-design question
- #concept — organisational folklore: fixes that live in people's memories instead of the system
- #person — the SVP whose "I don't want the details" reframes executive impatience as trust

## Opinionated take

This is a genuinely useful piece because it attacks the postmortem ritual from an angle most blameless-culture writing ignores. Blameless postmortems were supposed to fix blame; Heap points out they can produce something worse — a *consensus of reasonableness* that makes change feel unnecessary. "Nobody is at fault" slides imperceptibly into "nothing needs to change."

The "organizational folklore" line is the one worth stealing. It's a precise diagnosis of why so many postmortem action items die: they were never actions, they were sentiments with a due date. The leaving-the-company test is the practical companion — it's essentially the bus-factor test applied to corrective actions, and it should be a standard field on any incident template.

Where I'd push back: the essay underplays how often the *details are the change*. "Make ownership unambiguous when someone is unavailable" sounds crisp until you try to do it, and doing it well requires exactly the detail work the SVP waved off. The framing works because Heap already understood the failure; the risk is that leaders adopt the slogan without the diagnosis, and "tell me what we're changing" becomes pressure for cosmetic process theatre — a new checklist nobody owns. The line between "skip the empathy, give me the fix" and "skip the investigation entirely" is thinner than the essay admits.

## Relations to the wiki

This essay nuances [[AI Handles Incidents, Engineers Lose Touch with Their Systems]] from the opposite direction: that piece worries about engineers losing system understanding when agents handle incidents, while Heap worries about organisations *faking* understanding through postmortem ritual — both are failures of the understanding-to-change pipeline, at different levels.

It strengthens [[If AI Is Doing the Investigation, Version the Investigation]]'s instinct that incident investigation must produce a durable artifact: Heap's "would the fix still work if everyone left?" test is exactly the durability criterion such versioned investigations should be judged against.

It connects to [[Bad Data in Production — Response Playbook]]'s blameless-review stance, adding the sharper warning that blamelessness without a change mandate becomes absolution — the playbook's "review blamelessly" step needs Heap's follow-through question to avoid degenerating into folklore.

---
*Sources: [[raw/i-dont-want-the-details]], [[summary/i-dont-want-the-details]]*
*Last updated: 2026-09-25*
