---
url: https://www.technometria.com/p/authentication-is-largely-solved
date_fetched: 2026-09-11
---

**Summary**: *My new book, *Authorization in Action*, is out from Manning. It’s about the question that comes after we know who you are: what are you allowed to do, under what conditions, on whose behalf, and in what context? Authentication is largely a solved problem; authorization is not, and AI agents are about to make that gap impossible to ignore.*

My new book, *Authorization in Action*, is now available from Manning. It’s a book about the question that comes after we know who you are: *what are you allowed to do?* Knowing who someone is doesn’t tell you what they should be able to touch, change, or spend. For most of the last two decades I’ve worked on the first question, proving who someone is; this book is about why the second one is now the one that matters, and how to build systems that answer it well.

The short version of the argument is one I didn’t expect to be making a few years ago. Authentication, the act of proving who someone is, has become about as good as we can reasonably ask; passkeys and FIDO have quietly closed most of the gap that kept me up at night in 2005. Authorization, the act of deciding what someone can actually do, is still mostly improvised, buried in application code, and reinvented badly on every team. That imbalance is the reason the book exists.

**How I Got Here**

When I was CIO for the State of Utah in 2001, I noticed that nearly every problem that reached my desk had an identity component hiding inside it. Consolidating directories so every employee could have a `utah.gov` address, moving the state’s website to a new domain, sorting out who could touch which system: none of these looked like identity problems on the surface, yet all of them turned on knowing who someone was and what they were permitted to do. I didn’t have language for it then, but that observation set the direction for everything I’ve worked on since.

A few years later Doc Searls, Kaliya Young, and I started the Internet Identity Workshop(IIW), and it has now met 42 times. IIW has become the place where the identity community works on its hardest problems together. Along the way I wrote three books; *Digital Identity* helped enterprises build identity strategies, *The Live Web* moved the focus from organizations to individuals with their own data and APIs, and *Learning Digital Identity* finally named the thing I’d been circling for years: identity is fundamentally about *relationships, not identifiers*. Each book was really an attempt to answer a question the previous one had raised.

While I was finishing *Learning Digital Identity*, the next question came into focus, and conversations at IIW confirmed it. Knowing who someone is tells you almost nothing about what they should be able to do; the interesting, unsolved work all lives on the far side of authentication. That’s the work my new book takes on.

**The Second Question**

The reason the second question is harder is that it’s not one question but several, and they’re the ones that actually govern what happens inside a system. What can this person do; under what conditions; on whose behalf; and in which context? Miss any one of them and the rest stop meaning much; the same request can be right for a manager at noon from the office and wrong for a contractor at midnight from an unknown device. A login gets you through the door, but it says nothing about which rooms you can enter, what you can carry out, or whether the person who sent you had the standing to send you at all.

In late 2022 I joined AWS Identity, and one of the teams we worked closely with was building Amazon Verified Permissions and the Cedar policy language. Working at that scale showed me something the theory hadn’t: fine-grained authorization doesn’t only make a system safer, it makes it more usable. When the rules about who can do what are explicit and external to the code, you can hand people exactly the access they need without either drowning them in prompts or handing over the keys to everything. Getting authorization right is how a system stays both safe and pleasant to use, and those goals stop fighting each other.

**Then Came the Agents**

What finally convinced me the world needed this book was AI. As agents begin acting on people’s behalf, “what is this thing allowed to do, and on whose authority?” stops being a background concern and becomes the whole game. An agent that can read your calendar, spend your money, or send mail as you is only as trustworthy as the boundaries around it, and those boundaries are authorization. I’ve argued before that authorization is the hard problem in agentic AI, and nothing since has changed my mind.

This is where the technical question turns into a human one. If we can’t say precisely what an agent may do on our behalf, we’re left choosing between agents that can’t do anything useful and agents we have to trust blindly; neither of those is a world I want to live in. Authorization is the infrastructure that lets us delegate real authority to software while keeping it bounded, accountable, and revocable. That’s not a convenience feature. It’s what makes it possible to act through machines without surrendering to them.

**What’s in the Book**

*Authorization in Action* follows a fictional company, ACME, as it works its way from the tangled access-control code most teams start with toward policy-based authorization it can actually reason about. The book uses that story to get concrete about the models and mechanics—relationships, roles, attributes, policy languages, and the architecture that separates deciding from enforcing—rather than leaving them as abstractions. This is the book I needed on my own shelf while I was solving these problems as an enterprise architect at BYU, so it’s aimed at the developers and architects who have to make these decisions everyday

Identity tells us who is involved. Authorization determines what happens next. I’ve come to believe that second question is quietly becoming the heart of security, governance, and trust online, and I hope the book makes the case as clearly to you as the last few years have made it to me. You can find *Authorization in Action* at Manning and coming soon to Amazon.

Phil, I agree with your observations and conclusions. I also like that you have brought the Cedar Policy Language (CPL)into the discussion. When it comes to setting actual boundaries, guardrails and authorizations for AI agents, putting such a set of rules-based controls into practice is essential. The examples that Sarah Cecchetti demonstrated with her use of CPL and her "Clawdrey Hepburn" standalone installation of the AI agent on its own Mac mini (clawdery.com) were illustrative, informative, and extremely relevant to this topic.

P.S. Looks like we are in sync: after I left this comment I just noticed that Sarah herself has penned the foreword to your book !
