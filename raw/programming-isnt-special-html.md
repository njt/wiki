---
url: https://blog.glyph.im/2026/10/programming-isnt-special.html
date_fetched: 2026-10-10
---

## Creative Work

Writers went on strike to get protections against “AI”. Thousands of artists have signed open letters in protest of “AI”. There are so many copyright lawsuits from creative industry groups against AI that there’s a whole dedicated website for it. Popular YouTubers absolutely hate it. If they’re also musicians, they REALLY hate it. Across all creative industries, there is a concerted push to reject this technology.

Yet, almost unique among creative fields, many experienced programmers remain convinced that it’s fine to use “AI” for programming. We do seem to hate it, and it’s making us all miserable, and what it’s doing to our industry, but we are using it anyway.

A lot of the justification of this resignation seems to be to be because programming is not Art. If the tool can get the job done, and the job is just functional, then why does it matter?

It does matter, though.  It matters because we shouldn’t be using AI to produce Art, and *programming is Art*.

## Art can be Mundane

Some people will say that programs cannot be art because programs are *functional*, rather than being *expressive*.  Programs are mundane whereas art is transcendent.

This is based on a distorted understanding of what Art actually *is*.

In John Berger’s “Ways of Seeing”, he names this type of distortion “mystification”. His example of this process is both amusing and illustrative. I encourage you to read it in its entirety.

In summary, though: Berger critiques the florid prose of an art historian describing a commissioned group portrait, including phrases like “subtle modulations of the deep, glowing blacks” and “harmonious fusion”. The portrait is described as sublime, in nearly ecstatic terms.

Berger reveals that the *reality* of this portrait is that a poor old painter needed some work, and some officials probably thought it might be nice to have an official portrait.  So they paid some money to the poor old man, and he painted it, and then they had a painting.  It’s a well-executed portrait of a group of people.  Beautiful, even.  But it was work commissioned for a fairly mundane purpose and it suited that purpose just fine.  It was not, and is not, a divine relic.

Culturally, we are prone to mystifying painting, and sculpture, and film, and music. We imbue them with “subtle modulations”. We ignore their functional aspects — we desire decoration, amusement, and distraction — and focus on their emotional impact.

Don’t get me wrong: I love me some good aesthetic philosophy. I think it’s *great* to really examine our reactions to artwork and to try and gain a deeper understanding of our culture and our selves through media analysis. If anything we really need to do more of it.

This does *not* mean that the creation of such works is *mystical* or that it should be venerated beyond any other sort of labor.

Not least of which other types of labor that are adjacent to, but not as culturally venerated, as fine art. We tend to mystify the work of a novelist, but to denigrate the work of a journalist. In reality, the functional prose of the journalist is no less important and deserves no less respect.

Although they might be far below the ethereal realm that novelists inhabit in our collective imagination, even journalists receive more respect and thus more mystification than lowly copywriters. Yet, there is no transcendental distinction between “novelist” and “copywriter”; the many of the skills are the same, and the distinction is merely an accident of commerce and opportunity.

In fact, many famous writers have famously inhabited both roles.  This is not an accident!  Working with words professionally, even (perhaps especially) mundane words, is excellent *practice* for working with words in a more purely artistic context, because even mundane creativity is still artistic.

## Code can be Beautiful

One thousand Internet years ago, when I was in my late teens, I would describe myself as a “code poet”. I was relentlessly mocked for this as what the Youth would today call “being cringe”, and at the time was referred to as “pretentious”.

I succumbed to the peer pressure, removed it from my email signature and my bio. While I still believed strongly in the parallels, I accepted that — socially, at least — comparing code to poetry, or indeed to Art, was a silly thing to do.

However, I never abandoned the idea, in my heart.

The thing that I am most well-known for, the invention of `Deferred`, was specifically an *aesthetic* reaction to the tedium of passing `callback` and `errback` parameters to every single remote procedure call in an RPC client/server application.  Those two callbacks got the job done just fine.  But they were ugly, and annoying to work with.

`Deferred` is an intentional poem about asynchronous task execution, with a deliberate eye to the aesthetics of the problem and the experience of using it. It was influential *because* of its focus on aesthetics.

I do not want to overstate the beauty or profundity of this minor contribution, or indeed its durability.  That a poem exists does not mean it is a *great* poem, merely that it is a poem.

Our aesthetic culture around programs is more like folk epic poetry than fine art, so the influence of this contribution is less about its specific enduring power than it is about its influence on what came next; from MochiKit.Async to JQuery Deferred to JavaScript Promises and eventually to `async`/`await`; a long chain of different artisans each adding something of their own until the original has all but dissolved. (And I wasn’t the “original” here, either, as I drew heavily from the E language’s Promises, among other things.)

In order to make code into a deliberate artistic expression, one must have spent quite a bit of time contemplating the problem domain.  Without having experienced the tedium of manually passing a thousand callback parameters, I would have had neither the skill, nor indeed the *motivation*, to bother creating such a thing.

Now, *most* code does not have to be like this. Most code does not *get* to be like this. Most code is functional, workday code. Most code could not make a lady weep. It’s just copy-writing, if you will.

As I explained previously, most writing couldn’t do that either. Most writing is just copy-writing, too. Most visual art is advertising. Most live music performance is background music in bars that will go largely ignored.

However, code that *is* intentionally aesthetically designed tends to be important, both socially and technologically.

We do have *some* tradition of self-mystification in software.  As Abelson memorably put it, “Programs must be written for people to read, and only incidentally for machines to execute.”, so we have long had some conception of programs as highly expressive, even if we can’t always agree on what they’re expressing or to whom.  We will occasionally wax poetical about the philosophical implications of a particular piece of software. This is not unique to a single piece of software, either; more than one community has indulged in similar philosophizing.

The expressive and aesthetic qualities of software are not limited to reading source code or interacting with other programmers via APIs, either. For example, every year, Federico Viticci does a review of Apple’s new operating system, which is (among other things) an aesthetic critique. Such a project would not be possible if the software did not have an aesthetic impact on its users.

Not to mention that every video game review is also a software review.

### A Brief Aside about Software Literacy

It does make me a bit sad that we don’t have much of a critical reading tradition in the software community. Literate Programming is often praised, but rarely practiced.

Moreover, it makes me sad that users have a pretty jumbled idea of what goes into making software, that programming literacy is pretty low, and that modern programming practices often *deliberately* produce a bad mental model of what the software is doing so it’s even harder for the user to understand.  The aesthetic experience of software is often *wildly* detached from its internal state.

While all of these problems predate AI by years or indeed decades, that’s no reason to enthusiastically make them worse.

## Defend The Mundane

If we use AI to erase all the copy-writing, all the graphic design, all the boring mundane art, and yes, all the boring custom WordPress theme development, then we will be removing all the practical opportunities for the vast amounts of *practice* and *contemplation* required for people to elevate their craft to eventually achieve great things. Education is great, but the majority of true skill development happens on the job and always has.

This doesn’t mean that we can’t use abstractions, or automation, to make our work easier. Programming is the art *of* abstraction, of understanding how to compose smaller ideas into bigger ones, of how to understand the automation of a larger system by understanding the rules that automate smaller ones and understanding how to combine them.

When we use “AI” to *eliminate* that understanding rather than raise it up to a higher level, to entirely destroy that creative decision-making process, we do a disservice both to ourselves as programmers and to our users.  We would be doing a disservice to our users and our downstream fellow developers in the same way that a visual artist would be doing a disservice to their viewers or a musician would be doing a disservice to their listeners if they served them auto-generated filler instead of their own creative output.

Slop is slop, no matter the medium.

Each mundane project has some tiny chance — let’s say, something like 0.1% — of achieving greatness.  If we do a single project with AI, then sure, whatever, there’s almost no chance that *that* project was going to be the one hit to create that career-defining moment for an engineer working on it.  If we make a habit of doing *all* projects that way, though, we take the *total* likelihood of those moments of greatness from “definitely sometimes” to “never”.

The precisely appropriate ways in which to resist AI encroachment on all software development lie well beyond the margins of this one short post. How much you can resist and which specific uses you should resist are up to you. But it *is* worth resisting in software just as much as it would be worth resisting in any creative medium.

Programming isn’t special. It’s just Art, and Art is the most human — and thus, the most universal — thing that there is.

## Acknowledgments

Thank you to my patrons who are supporting my writing on this blog. If you like what you’ve read here and you’d like to read more of it, or you’d like to support my various open-source endeavors, you can support my work as a sponsor!
