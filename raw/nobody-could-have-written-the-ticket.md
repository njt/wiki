---
url: https://www.jamesdrandall.com/posts/nobody-could-have-written-the-ticket/
date_fetched: 2026-09-20
---

A turn-based 4X strategy game I built from scratch — custom WebGPU engine, eight civilisations, no install. Play it in your browser right now.

*This is part of a pair of articles on the impacts of agent driven development - you can read thoughts on the review process here.*

I threw away the entire AI for Annhexation and started again. Annhexation is my 4X strategy game (which incidentally you can wishlist on Steam!).

Not refactored it. Deleted it, wrote a new spec, built the second one from scratch. If you look in the source you’ll find a package called ai-v2. The first version worked, in the sense that it played legal moves, expanded sensibly, never fell over. It was also no fun at all to play against, didn’t generate interesting decisions, and as I tinkered with it I realised that no amount of tuning was going to fix that, because the problem was in the shape of the thing I’d specified rather than in its rules and weights.

Version two is much better. But I could not have written the version two spec first because all the insights in it came from having built version one and played against it over and over. The thrown-away implementation was the instrument that produced the better specification. If you’re interested in how the final version works I’ve written about it here.

I’ve been thinking about that a lot, because there’s a model of agentic development going round at the moment that has no room in it for what I just described. At least as often presented.

## The model I don’t believe in

The pitch is that you have a backlog, you point an agent at a ticket, it reads the ticket, understands the codebase, makes the change, writes the tests, opens the PR, responds to your review comments. You do that at volume and the backlog goes down.

I want to be clear that the individual steps in that work. I’ve built systems that do a chunk of it. I’ve developed parts of Annhexation in this way and built project specific tools to support it. Building Annhexation without this technology, this approach, would have taken me years. Its not a trivial project, its large, with many subsystems, and full of nuance. I want to be clear this isn’t a piece about the tools being bad, they’re really not, in fact I think they’re better than most people arguing about them realise.

Rather it’s a piece about what this model assumes. And what it assumes is that the ticket is a sufficient description of the work.

## A ticket is a hypothesis

A ticket isn’t the work. It’s the residue of a decision, or series of decisions, somebody already made.

Somebody thought about the problem, formed a view about what should happen, and wrote down enough for a person to act on. All the expensive part - the deciding, the weighing, the discarding of three worse options - happened before the ticket existed. What’s left in the ticket is the conclusion with the reasoning boiled off. Do this. And here’s how you know its done.

That’s fine, as long as everyone treats it as what it is: a hypothesis about what would be good, written by someone who hadn’t yet seen it running.

But an agent doesn’t treat it as a hypothesis. Rather it treats it as a contract, and it satisfies it.

## Where the line actually falls

Now I don’t actually think there’s anything new here, good product teams know this. And this isn’t a general argument against pointing agents at tickets. There’s a real dividing line, and it isn’t difficulty. It’s whether the acceptance criterion exists outside a human head.

A crash with a stack trace, a dependency that needs bumping, a calculation returning the wrong number for a known input. These are cases where reality supplies the specification. The ticket is a complete statement of the work, there’s something external for the agent to be right or wrong against, and if it’s wrong you’ll know. That whole category is a fair target for automation, and I’d defend building for it.

Design work is the other side of the line. Not because it’s harder, but because the criterion lives in a person, and often doesn’t exist yet even there. Nobody knows whether a mechanic is interesting until it exists and someone has played it. Nobody knows if a feature will resonate with customers until they can try it. All companies have delivered what they were sure were killer features that quietly died.

Everything below is about that second category.

## What actually happened with eight civilisations

Annhexation has eight asymmetric civilisations, three victory paths, deterministic combat, and an AI in three layers. All of that was specified up front - I worked on what turned out to be a large set of game design documents. And I’m not making a case for working without specs: I wrote them, they were reasonable, some of them were quite detailed, and they drove the early development. Without them I’d have been working blindly with or without agents.

But what’s in the game now bears very little resemblance to any of it.

Not because the specs were careless. Rather because a civilisation that reads as distinctive on paper turns out to play identically to another one by turn 40. Because an asymmetry that looks interesting in a design document turns out to just be worse, and nobody picks it. Because the mid game sagged. Because a victory path that ought to create tension instead creates a race that’s decided by turn 15. None of that is visible before the thing exists. You find it by playing.

And I responded to those discoveries in three quite different ways, at three different scales:

**On the fly.** Something’s off, you’re already in the code, you change it. Half a mechanic gets adjusted before you’ve articulated what was wrong with it.

**A new sub-spec.** The discovery is big enough to need its own thinking, but the surrounding design holds. You write the smaller thing down and build it properly.

**Bin it and rewrite.** The spec was wrong at the root. That’s the AI v1 story. No amount of adjustment gets you there, so you throw the implementation away and write a better specification, which you can now write because you’ve seen the failure.

A ticket-driven agentic flow can do the first of those in a limited way. It can’t do the third at all. It can help you write the new spec, yes, but only because you’ve seen the failure.

## Fidelity is the failure mode

This is the bit I’d want anyone building these systems to sit with for a minute.

The agent’s problem isn’t that it might get it wrong. It’s that it gets it right.

It will implement the hypothesis with total fidelity and it will never come back and say this civilisation is boring by turn 40, because it hasn’t played the game and nothing in the loop asks it to. A person implementing that same spec usually says something on the way. Halfway in they’ve felt the shape of it, and they push back, or they quietly build the better thing and mention it at standup.

That channel doesn’t exist. Worse, its absence is invisible, because a faithful implementation of a bad spec looks exactly like a faithful implementation of a good one. Same green tests, same clean diff, same tidy PR description. There’s no signal in the artefact that tells you the specification was wrong.

There’s a second loss that’s easier to miss. Building is how you find out. You learn things while implementing that change what should be implemented - the balance problem you only see with the actual numbers in front of you, the mechanic that turns out to interact badly with another. Delegate the implementation and that information is generated inside the agent, and it doesn’t reach you. You get the artefact without the learning.

## This is waterfall

I’d been dancing round saying this, so let me just say it: ticket in, finished solution out, is waterfall.

Not “a bit like waterfall”. The same control structure. Waterfall’s failure was never the documentation - people get distracted by the documentation. It was the single forward pass with no return path. Requirements go one way, and there’s no mechanism by which building the thing is allowed to change what the thing should be.

Ticket to agent to PR to merge is a single forward pass with no return path, running considerably faster.

And here’s why nobody notices: waterfall was survivable in practice because people cheated. Most real waterfall projects had an informal back channel doing the work that the process denied. The corridor conversation, the requirement quietly renegotiated, the engineer who built the sensible thing and got forgiveness afterwards. The process on paper never worked. The humans inside it made it work by ignoring it.

An agent doesn’t cheat. It does exactly what the ticket says.

If we allow agents to run from ticket to merge then we’ve built the first faithful implementation of waterfall anyone’s managed, and faithfulness turns out to be the flaw.

## What they’re actually good for

So here’s where I land, and it’s not where the negative half of this post might suggest.

The gain from these tools isn’t getting it right first time. That was never available and it isn’t now. The gain is that iteration got cheap.

Iteration always happened in any healthy product cycle. The question was only ever what it cost. And what an agent compresses is exactly the expensive middle bit - the two weeks of implementation that stood between having a hypothesis and finding out whether it was any good. Fantastic ideas would never be surfaced to a customer down to this. Growth opporuntities sidelined on the back of risk and cost.

Which changes something real. Throwing away two weeks of your own work is a decision people avoid, and they avoid it for reasons that have nothing to do with engineering. Throwing away an afternoon is easy. So you can afford more hypotheses, and you can afford to kill the bad ones earlier, and that’s a genuine improvement in how a product gets made rather than a marginal speedup.

I’m reasonably sure the AI v1 rewrite would have taken me longer to *decide on* if the implementation had cost more. I can easily imagine a game studio never actually doing it - the cost being too high. Gamers (and I’m one) often scratch their heads at the seemingly obvious flaws that make it to release. This is one of the reasons why. It was too expensive top iterate and fix.

But it caps out, and it caps out quickly, because evaluation didn’t get cheaper. You still have to play the game. Someone still has to use the feature and notice it’s dull. That’s human, it runs at human speed, and it doesn’t compress. Make the build step ten times faster and the loop gets ten times faster only if judgement keeps up, which it doesn’t - so what you actually get is a shorter build step and the same bottleneck sitting immediately after it, now more obviously the bottleneck than it was before. You can build tools, and I have, to accelerate the feedback once you start to understand the failure states - but you still have to understand the failure states.

There’s also a trap sitting underneath the whole “just write better tickets” response, which is the one I’d expect to this post. A specification precise enough to be unambiguous is isomorphic to the program. Push the ticket-writing far enough to make the agent reliable on design work and you have written the code in a worse language, more slowly, with no compiler.

## Three things I think are true

Confidence attached, as is becoming usual.

- 
**The specifiable fraction of a backlog is a real ceiling on ticket-driven agents.**Reasonably confident. It’s not a property of the tools, it’s a property of your product. How much of your board has an acceptance criterion that exists outside somebody’s head. My guess is well under half for most product teams and lower still for anything pre-product-market-fit, but that’s a guess though one founded on working on a lot of products.
- 
**The value shows up as iteration count, not as throughput.**Moderately confident. If you’re measuring tickets closed you’ll conclude these tools are working brilliantly while the product gets worse. The thing to watch is how many hypotheses you tested and how many you killed.
- 
**Somebody has to keep playing the game.**High confidence. Whatever the ratio ends up being, it’s bounded by how much contact a human still has with the running thing, because that contact is the only place the feedback comes from.

## Where this landed

The Annhexation AI is on its second version and I’d say it’s had four or five significant reworkings since. I created new sub specs (naval warfare turned out to require a really different set of goals and tasks), on-the-fly changes, and a substantial rethink of the diplomacy mechanism. It’s still not finished. In truth it never will be. I’m still tuning behaviour now, months after the game was otherwise done, because I keep playing it and keep finding things.

Not one of those changes could have been a ticket written in advance. Every one of them came from sitting in front of the thing, playing lots of games (real and simulated), and noticing things. I’ve accelerated this process by evolving my analysis tools in parallel - but similarly I couldn’t spec those up front beyond a core because the process of understanding what to analyse, what to look for, is similarly iterative.

That’s the part these tools don’t touch. It’s not a gap they’re closing slowly - it’s a different activity, and it’s the one that determines whether what you built is any good. The agents make everything around it cheaper, which is genuinely useful, transformational even, and I’d not go back.

But somebody still has to play the game. Somebody still has to have taste and judgement.

August 2026

Forty-two years after I first started programming, I built a browser-native 4X from scratch — a custom WebGPU engine, eight civilisations, and an AI opponent I'm still teaching to play. No install, no sign-up. Play it right now.
