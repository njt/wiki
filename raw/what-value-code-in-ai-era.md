---
url: https://jerf.org/iri/post/2026/what_value_code_in_ai_era/
date_fetched: 2026-10-03
---

# How Do We Value Code In A World Of Free Code?

Two strands are making me ponder the “value” of code in our new AI
era. One is the increasing cheapness of *producing* a line of code,
trending toward zero. As a professional who produces lines of code,
among other things, this trend is something I need to be thinking
about for many obvious reasons. The other is the question of how to
moderate a Reddit programming
community
in a way that keeps the community as healthy as possible.

It turns out these have some surprising overlap.

It is, of course, too soon in the AI revolution to expect anyone to be able to produce a complete metric for evaluating the value of code. Consider this to be open musings and an invitation to conversation, not a claim that I’ve got it all worked out.

One thing AI does not change, and indeed may make worse:

# A Line Of Code Is Still A Liability

All else being equal… and let me be the first to observe that that
phrase is doing a lot of heavy lifting, as the idiom goes… a program
that is 200 lines long is better than a program that is 1000 lines
long. That is to say, for two programs of otherwise equivalent utility
to an owner1, short readable code is broadly better than long
readable code.

The more mature programmers have known this for a while. But this
first flush of AI programming is meeting a management culture that was
already again trying to figure out how to objectively measure software
engineering output2, and sense has left the building, chased
out by the promise of dancing chart lines of dollars going up, up,
up. Lines of Code as a metric has been appealing for decades anyhow,
but now LoC offers a manager or a CxO the opportunity to select a
metric that is going ballistic and gives them a chart of Number Go
Up that will appeal to
an investment market that has gone AI Crazy. It is unreasonable to
expect the business people to reject the offer.

For decades we barely held back management, and it still periodically required the ritual sacrifice of some failed project to remind them that this is not a measure of value. We’ll probably win this in the long term again but not until some more major projects crack their heads open worse than ever against this brick wall.

If we are all lucky, or unlucky… which it is I will leave up to the reader to evaluate… the sacrifice this time around might even end up being Microsoft Windows.

A capability of your software stack is an asset. A line of code is a liability.

Therefore, an AI system that can rapidly produce capabilities is producing assets. But an AI system that is rapidly producing lines of code is producing liabilities. This is no contradiction; things always have both costs and benefits. However you get yourself into trouble when you fail to account things correctly; start putting costs into the benefits column or vice versa and you are inevitably going to make some gargantuan mistakes.

No amount of advancement in AI is going to change the fact that lines of codes are a liability, because…

# AIs Are Finite

AIs will forever in this universe be finite. This is independent of their architecture, how expensive or cheap they become, how good at their job they are, or any other characteristic. It’s possible they will become so intelligent that we entirely lose the ability to analyze their output, but I like to keep my prognostication firmly pre-Singularity, for the simple reason that I don’t really have a choice.

And they’ll *still* be finite. Just beyond our comprehension.

I think many people are so bedazzled by AI right now that they are conceiving of it as infinite, or that it will be someday. Obviously without consciously examining the proposition, since it is one of those things that once brought to the conscious level is easily seen to be obviously wrong.

We can make certain logical deductions based on that finitude.

An infinite AI would not care how many lines of code you shove at it. A finite AI does. A finite AI will always have some sort of limit. The AI will always do a better job with code that consumes fewer resources rather than more. The AI will always do better with code that is designed in such a way that it requires less cognition rather than more to understand.

AIs may be slinging lines of code around by the trillions in mere minutes in the future and this will still be true.

Sidebar: Will we need code at all in an AI era? Yes. Current AIs are vast, vast sets of numbers that we perform trillions of operations on for their output. Consider asking an AI to add a dozen numbers that are ten digits each. It will be, by necessity, literally trillions of times more expensive than it would be to do it with a conventional CPU, with a much higher probability of getting it wrong. As AIs get more capable that ratio is likely to continue to grow, probably by orders of magnitude. The time may come that humans are completely uninvolved in the code, but code itself isn’t going anywhere because that performance advantage is not only not going anywhere, but it will continue to grow. The AIs will absolutely build code in some form for their own efficiency’s sake. The people I’ve seen who ask if perhaps in the future absolutely everything will back to AIs are perhaps the worst examples of thinking of AIs as infinite.


So, one way to value code is code that takes less of the AI’s
cognitive power to understand. At the present time, this happens to
largely overlap with what takes less human cognition to understand
too3. Code that documents the intention of the code and the
“why” of the code, and code that is clearly written and idiomatic for
the target language, is still better code for AI as well.

One problem with the current AIs is that they will choke themselves
out under their own line-of-code liabilities faster than they
should. In my limited experience, one of the problems I have with even
the best AIs is that I still have a hard time getting them to write
code that uses invariants. For instance, if I have a “username” in my
system, I will create a type for that
username
and make it so if you have a `Username` value in the rest of the code,
it is guaranteed that it could only have gotten into that type by
virtue of being validated as a legal `Username`, for whatever the
local definition of “valid”
is.

I have not had much luck getting AIs to produce code on its own that creates or maintains these invariants. I don’t think I’ve yet seen an AI produce a class like this on its own, and if I feed it some pre-existing code that already maintains this invariant, it quite often wants to crack the class open and make a change that breaks the invariant, or it wants to redundantly double-check validity elsewhere because just based on the way it understands the code today it doesn’t understand it’s already impossible for those checks to fail, or do something else like that.

Over the past decades I’ve worked with humans that worked in a very similar manner. I often thought of how they operated as just spraying code over a problem like a firefighter sprays foam on a fire. Some problem with XSS on a particular page? Spray out some check on the spot for this particular issue. Ignore that we have already sprayed out dozens of other such checks, all inconsistent with each other. Don’t even ponder the question of whether one could construct an architecture that prevents XSS in a way that doesn’t require this sort of constant firefighting and code spraying. Just spray more code at the problem. There is apparently no problem that can’t be solved by spraying more code, not even the problem of having too much code.

AIs are even *better* at this style of coding, which is to say, even
worse. AI is great at blasting out lines of code right now, but it
isn’t anywhere near as good at producing high-quality lines of
code. Partially I suspect this is because what I’m calling “high
quality” is not necessarily well-represented in the training set. I’m
still providing quite a bit of value in my AI coding by forcing it to
follow best practices that on its own it doesn’t want to follow. I
also find the auto-generated unit tests that I’ve seen, even from the
latest models, to be maybe a good start but rarely penetrating to the
level of whether the code does what it is *supposed* to do, rather
than testing that it does what it does. (Which is, at least, a start,
but not actually the question we need answered.)

Even in strongly-typed languages, AI is often writing
*scripts*. Everywhere. Everything it slaps down is really script
code. We know from previous human history that you can in fact get
quite a way with this scripting code. I’ve seen human-produced
monstrosities of systems that are, under the hood, just decade’s worth
of sprayed scripts on top of a problem, and they made money.

So I’d say to sum this point up, AIs are finite, they tend to produce the sort of code that is cognitively expensive to process, even for AIs, and one way to valuate code is to value code that AIs are better at working with, even if they can’t produce it very well on their own yet. Such code can be taken farther and have more value extracted from it by both humans and AIs than slop script code, and is therefore, more valuable code.

# Tested Against the Real World

As an engineer, I have a constitutional aversion to corporate jargon, but underneath the buzzwords that get bent, spindled, and mutilated by every business person on Earth trying to extract the value from every bit of jargon they can get their hands on, there is usually a kernel of truth in the term’s original meaning. In this case, the term “solution” comes to mind. Companies don’t want to sell you databases or code to email your customers or ways of collecting you money, they want to sell you “solutions” to your problems. The idea is that the products are just a means to an end, and that end is to take your money and in return make a problem of yours go away entirely, or at least, as close to that as possible.

This is generally thought of in a business context, but it applies to normal individuals as well.

It doesn’t matter how smart AI models get. If Bob the new accountant
hire prompts an AI to “solve my accounting problems” and it produces
in ten minutes what would today take a team of 10 *and* all the AI
assistance available in 2026 a year to generate, that may be a big
pile of working code, but it isn’t actually a *solution*. The system
has not been tested against reality. The system hasn’t been dragged to
court and proved itself there. It hasn’t proved that it can flex with
changes over time. No matter if it is immaculate with no overt bugs,
perfect accessibility, perfect integration with any number of systems,
perfect internal testing, etc., there is simply no way that a code
base that was called into existence ten minutes ago can be *tested
against the real world*.

After an initial round of “oh my gosh my tier 1 support staff can call
entire business backend systems into existence holy cow that’s
awesome!”, I think the *real* value of “I have an accounting system
that has been tested in the field for many years, has successfully
stood up in court against X, Y, and Z lawsuits, has proved itself by
use by thousands of companies, has stood up to these other challenges”
is honestly going to seem even *more* appealing in a world where
anyone can instantiate a system than it does now.

Moreover, if Bob the new account is capable of calling into existence
several million lines of code over a lunch break, so is the rest of
the world. It is likely that Bob’s AI will actually be choking on its
need to interact with a world that has been created by everyone else
*also* being able to call into existence millions of lines of code
over a lunch break, and the problem Bob has to face in that era are
probably much larger than the problems he would face today.

To give just a single example of what we could see in the future that we don’t see today, suppose Bob lives in a world where because everyone can do this, every single HOA in the United States has been given taxation authority. The taxation regimes we have today already critically depend on computers, so what will be possible when we can use AI to implement that my HOA taxes any drink that comes in a can (for litter control, of course), but the one down the street taxes only sugary canned drinks (for health control, of course), and all of the tens or hundreds of thousands of other ones can do their own thing? And what other sorts of regulatory regimes will end up installed, to say nothing of all the other ways things can be complicated in the future?

In the end, highly tested code is a *solution*, and the untested
system that Bob invoked from the Magic Code Gods half an hour ago is
on the *liability* side of the balance sheet.

Software as a Service isn’t going anywhere. It is certainly in for
various shocks, but no matter how good the AIs get at generating code
no 10-person company is going to want to have a bespoke accounting
system, a bespoke website, a bespoke cloud deployment system, a
bespoke entire business infrastructure. They don’t want a bespoke code
bases; they want their problems to *go away*. Bespoke systems still
mean that the problems are theirs. Making them “easier” to solve still
means they’re my problems to solve, and probably there’s more of them
too.

By Amdahl’s law, removing all the coding time just means all the other aspects of dealing with a system turn into 100% of the costs. The code costs loom large today, but if they drop to zero there would still be all the other associated costs of having their own bespoke solution that business don’t want to deal with because they are not their core competency.

The more code is exposed to the real world, the more value it has and demonstrates. This was true already, but AI is going to sharpen that until it is undeniable.

The limiting factor on this type of value is going to be that there
isn’t enough “real world” to go around for all the AI code being
generated. What will happen for SaaS is that SaaS companies will be
empowered to wrangle even *larger* code bases than they do today, and
then they will suck up all the “real world testing” for themselves and
generate the value, and the gap in value between what the SaaS company
offers and what Bob the front-desk guy can call up will actually be
even *larger* than it is today, because the SaaS will be wrangling 100
times the code and 100 times the capabilities with 100,000x the
testing against the real world.

# Valuating Social Media Posts

As I’ve been trying to moderate /r/golang in the last year or so, I’ve had to deal with the proliferation of little projects popping up and annoying the community by suddenly flooding the community of everything else. Hacker News has a similar problem (discussion thread). I imagine plenty of other programming communities are as well.

At first I tried to segregate projects based on apparent effort. But even since I started doing that, AI has gotten a lot better, harder to detect, and as I watched my own gut reactions to my own moderation actions, I began to realize it wasn’t really the correct distinction to cut on.

What the real problem is, in my opinion, is that you can now create a rather complicated project without having to run it past reality to speak of. Even the minimal amount of reality a 2010s programmer would need to use, just to personally test the project seems to basically work, and which was still not really anywhere near enough testing to make something valuable, is no longer necessary. If you can not today call a full HTTP proxy into existence simply by asking for it and describing how you’d like to configure it, we are clearly no more than months away from it being nothing more than a few sentences typed into a box.

And while perhaps the proximal reason we are suddenly flooded with
posts about projects of these sort is their recent breathtaking ease,
I would submit the reason why these projects are *valueless* to the
communities in question isn’t just their ready availability, though
that doesn’t help, but their lack of *testing against reality*.

If you have a 100% AI codebase, and I’ll stick with HTTP proxying since it seems to be one of the popular projects, but it has been running in production and serviced a few trillion web proxy requests in a highly heterogeneous environment while still doing whatever it is it is supposed to do, that’s valuable.

If you have a 100% human-crafted codebase, but its sum total of exposure to the real world is that the human author has written some unit tests based on their understanding of a few RFCs and pushed a few test pages through by manually running requests through telnet, that’s not particularly valuable. Likewise, obviously, for a 100% AI codebase with the same characteristics.

So one of the things I’m looking for is whether a project has users other than the developer, or if the developer is perhaps already using it hard enough to be testing it to some non-trivial degree. For instance, a lot of code open sourced by a company that has been using it for years fits into this category. Code called into existence two days ago does not.

This certainly correlates to the effort put into a code base, but it
is also definitely not the same, and provides a completely different
evaluation rubric than *how much AI was used* (a tempting rubric still
in common use but increasingly irrelevant) or *how much effort did the
developer put in*. There are plenty of valuable code bases that may
not involve much apparent “effort” but attain value by being widely
used, and sadly, plenty of code in the world that is some remarkable
effort by someone but has never seen any exposure to the real world at
all.

# End

The situation is developing quickly and I reserve the right to change my own opinions as quickly. How I valuate code may change in another six months. But for the moment this seems to me to be the thing to keep an eye on.

Keep an eye out for the day we start having to use AI to deal with the additional complications created by the use of AI to generate lots of cheap code in the first place. Keep an eye out for the first API you see that is clearly designed with the assumption that your AI is just going to be using it anyhow so it doesn’t need to be particularly clean because it doesn’t need to fit into a human mind. That’s when you will really start to see even AI-generated code that interacts with things like this need to be validated against the real world, as AI intelligence starts getting burned on the use of other AI intelligence.

- 
Note the difference between “equivalent functionality” and “equivalent utility to a given user”. Between two programs of “equivalent functionality”, generally shorter is generally better too, with a quirky little value drop at the extreme as the code becomes code golf rather than readable code, but two programs are almost never technically truly equal in functionality which makes them hard to compare. The observation here is that what really matters is the utility to the user, especially in commercial contexts. An all-singing, all-dancing, vast solution to some problem where the business only uses 3% of it would be even more valuable in most ways if it *only*had that 3% of functionality. To the extent that you’re bothered by that, it’s probably some variant of “but what if they need it later” or “but what if they sometimes use more”, but for the sake of argument, they really are just using that 3%. ↩︎
- 
Under the false credo of “You can’t improve it if you can’t measure it”. There is, as is usually the case, a kernel of truth in there, but this is generally presented as an absolute, that it is absolutely impossible to improve if it can’t be measured in some concrete way. This is false, and past a certain point, the more seriously this is taken, the more certain characteristic and increasingly deadly failure modes will be seen. Do a search on Goodhart’s Law for more info. ↩︎ 
- 
I phrase that carefully. It may not be true forever. I think that given that our coding AIs today produce human-comprehensible code that it would be *highly*advantageous for the human race to continue to insist on that going forward. It has been lost in AI money-grabbing frenzy but a few years ago we were talking a lot about AIs being “legible”, that they could explain their actions in human-comprehensible terms. “Running code we can examine” is the highest grade of legibility any AI system has produced to date. We should not give that away.We will, of course. The Number Must Go Up. We aren’t very good at this sort of thinking. But we shouldn’t. ↩︎
