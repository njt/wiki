---
url: https://news.ycombinator.com/item?id=46574276
date_fetched: 2026-07-05
backfilled: true
---

I don't understand the stance that AI currently is able to automate away non-trivial coding tasks. I've tried this consistently since GPT 3.5 came out, with every single SOTA model up to GPT 5.1 Codex Max and Opus 4.5. Every single time, I get something that works, yes, but then when I start self-reviewing the code, preparing to submit it to coworkers, I end up rewriting about 70% of the thing. So many important details are subpar about the AI solution, and many times fundamental architectural issues cripple any attempt at prompting my way out of it, even though I've been quite involved step-by-step through the whole prototyping phase.

I just have to conclude 1 of 2 things:

1) I'm not good at prompting, even though I am one of the earliest AI in coding adopters I know, and have been consistent for years. So I find this hard to accept.

2) Other people are just less picky than I am, or they have a less thorough review culture that lets subpar code slide more often.

I'm not sure what else I can take from the situation. For context, I work on a 15 year old Java Spring + React (with some old pages still in Thymeleaf) web application. There are many sub-services, two separate databases,and this application needs to also 2-way interface with customer hardware. So, not a simple project, but still. I can't imagine it's way more complicated than most enterprise/legacy projects...

I’ve come back to the idea LLMs are super search engines. If you ask it a narrow, specific question, with one answer, you may well get the answer. For the “non-trivial” questions, there always will be multiple answers, and you’ll get from the LLM all of these depending on the precise words you use to prompt it. You won’t get the best answer, and in a complex scenario requiring highly recursive cross-checks— some answers you get won’t be functional.

It’s not readily apparent at first blush the LLM is doing this, giving all the answers. And, for a novice who doesn’t know the options, or an expert who can scan a list of options quickly and steer the LLM, it’s incredibly useful. But giving all the answers without strong guidance on non-trivial architectural points— entropy. LLMs churning independently quickly devolve into entropy.

I wish LLMs were good at search. I've tried to evaluate them many times for their quality at answering research questions for astrophysics (specifically numerical relativity). If they were good at answering questions, I'd use them in a heartbeat

Without exception, every technical question I've ever asked an LLM that I know the answer to, has been substantially wrong in some fashion. This makes it just.. absolutely useless for research. In some cases I've spotted it straight up plagiarising from the original sources, with random capitalisation giving it away

The issue is that once you get even slightly into a niche, they fall apart because the training data just doesn't exist. But they don't say "sorry there's insufficient training data to give you an answer", they just make shit up and state it as confidently incorrect

LLMs got good at search last year. You need to use the right ones though - ChatGPT Thinking mode and Google AI mode (that's https://www.google.com/ai - which is NOT the same as regular Google's "AI overviews" which are still mostly trash) are both excellent.

> LLMs got good at search last year. You need to use the right ones though - ChatGPT Thinking mode and Google AI mode (that's https://www.google.com/ai - which is NOT the same as regular Google's "AI overviews" which are still mostly trash) are both excellent.

I disagree. You might have seen some improvements in the results, but all LLMs still hallucinate quite hard on simple queries where you prompt them to cite their sources. You'll see ChatGPT insist quite hard that the source of their assertions is the 404 link that it asserts is working.

This is just completely the opposite to what i've experienced within Claude and Gemini. Sources are identified and if inaccessible are not included in the citations. I recently tried a quite specific search aimed towards finding information about specific memo's and essays cited within a 90s memo by bill gates, and it was succesful at finding a vast majority of them, something google search failed with.

I don't want to say that it's a skill issue, but you may just be using the wrong tools for the job.

>Otherwise, the variational principle is not well-defined.

Which is borderline gibberish

>The theory still has 2 propagating DOF per spacetime point

This is pretty good too

>(lapse and shift act as Lagrange multipliers, not dynamical fields).

This is also as far as I'm aware just wrong, as the gauge conditions are nonphysical. In general, lapse and shift are generally always treated as dynamical fields

Its full answer reads like someone with minimal understanding of physics trying to bullshit you. Then I asked it if the BSSN formalism is strictly equivalent to the ADM formalism (it isn't, because it isn't covariant)

This answer is actually more wrong, surprisingly

>Yes — classically, the BSSN formalism is equivalent to ADM, but only under specific conditions. In practice, it is a reparameterization plus gauge fixing and constraint handling, not a new theory. The equivalence is more delicate than ADM ↔ GR.

The ONE thing that doesn't change in the BSSN formalism is the gauge conditions

>Rewriting the evolution equations, adding terms proportional to constraints.

This is also pretty inadequate

>Precise equivalence statement

>BSSN is strictly equivalent to ADM at the classical level if:

...

>Gauge choices are compatible
>(e.g. lapse and shift not over-constraining the system)

This is complete gibberish

It also states:

>No extra degrees of freedom are introduced

I don't think chatgpt knows what a degree of freedom is

>Why the equivalence is more subtle than ADM ↔ GR
>1. BSSN is not a canonical transformation

>Unlike ADM ↔ GR:

>BSSN is not manifestly Hamiltonian

>The Poisson structure is not preserved automatically

>One must reconstruct ADM variables to see equivalence

This is all absolute bollocks. Manifestly hamiltonian is literally gibberish. Neither of these formalisms have a "poisson structure" whatever that means, and sure yes you can construct the adm variables from the bssn variables whoopee

>When equivalence can fail

>Discretized (numerical) system -> Equivalence only approximate

Nobody explain to chatgpt that the ADM formalism is also a discretiseable series of PDEs!

>BSSN and ADM describe the same classical solutions of Einstein’s equations, but BSSN reshapes the phase space and constraint handling to make the evolution well-behaved, sacrificing manifest Hamiltonian structure off-shell.

We're starting to hit timecube levels of nonsense

It also gets the original question completely wrong: The BSSN formalism isn't covariant or coordinate free - there's an alterative bssn-like formalism called cBSSN (covariant bssn), which is similar to ccz4 and z4cc (both covariant). Its an important property that the regular BSSN formalism lacks, which is one of the ways you can identify it as being not a strict equivalence to the ADM formalism on mathematical grounds. So in the ADM formalism you can express your equations in polar coordinates, but if you make that transformation in the BSSN formalism - its no longer the same

This has actually gotten significantly worse than last time I asked chatgpt about this kind of thing, its more confidently incorrect now

> Without exception, every technical question I've ever asked an LLM that I know the answer to, has been substantially wrong in some fashion.

The other problem that I tend to hit is a tradeoff between wrongness and slowness. The fastest variants of the SOTA models are so frequently and so severely wrong that I don't find them useful for search. But the bigger, slower ones that spend more time "thinking" take so long to yield their (admittedly better) results that it's often faster for me to just do some web searching myself.

They tend to be more useful the first time I'm approaching a subject, or before I've familiarized myself with the documentation of some API or language or whatever. After I've taken some time to orient myself (even by just following the links they've given me a few times), it becomes faster for me to just search by myself.

>> at answering research questions for astrophysics

I googled for "helium 3" yesterday. Google's AI answer said that helium 3 is "primarily sourced from the moon", as if we were actively mining it there already.

There are probably thousands of scifi books where the moon has some forms of helium 3 mining. Considering Google pirated and used them all for training it makes sense that it puts it in present tense.

I wonder how much memory and computing time goes into making them, vs. a typical "proper" LLM prompt. It's like the freebies you get with a Christmas cracker.

The entire situation of web search for LLMs is a mess. None of the existing providers return good or usable results; and Google refuses to provide general access to theirs. As a result, all LLMs (except maybe Gemini) are severely gimped forever until someone solves this.

I seriously believe that the only real new breakthrough for LLM research can be achieved by a clean, trustworthy, comprehensive search index. Maybe someone will build that? Otherwise we’re stuck with subpar results indefinitely.

YaCy does a pretty good job, and is free, and you can run yourself, so the quality/experience is pretty much up to you. Paired together with a local GPT-OSS-120b with reasoning_effort set to high, I'm getting pretty good results. Validated with questions I do know the answer to, and seems alright although could be better of course, still getting better results out of GPT5.2 Pro which I guess is to be expected.

The point of my comment was that the AI/LLM is almost irrelevant in light of low quality search engine APIs/indexes. Is there a way to validate the actual quality and comprehensiveness of YaCY beyond anecdata?

Yeah, if that's how you feel about your own abilities, then I guess that's the way it is. Not sure what that has to do with YaCy or my original comment.

I assume that should be qualified with some basic amount of evidence beyond “I said so”? Anyways, thanks for pointing me in the direction of YaCy, will try it out.

It's funny for me to read this. They don't exhibit "confidence". You are just getting the most accurate text that it can produce. Of course, the training data doesn't contain "I don't know" for questions, that would be really bad training data! If you are getting "attitudes", it would be because you are triggering some kind of dialogue-esque data with your prompts (or the system prompt might be doing that).

Expecting the LLM to say "sorry I don't know" would be like expecting google search to return "we found some pages but deemed them wrong, so we won't show you any".

An example I had last month. Some code (dealing with PDF's) package ran into a resources problem in production. LLM suggested an adaptation to the segment that caused the problem, but that code pulled in 3 new non-trivial dependecies. Added constraints and the next iteration it dropped 1 of the 3. Pushed further and it confirmed my suggestion that the 2 remaining dependencies could be covered just by specifying an already existing parameter in the constructor.

The real problem btw was a bug introduced in the PDF handeling package 2 versions ago that caused resource handeling problems in some contexts, and the real solution was roling back to the version before the bug.

I'm still using AI daily in my development though, as as long as you sort of know what you are doing and have enough knowledge to evaluate it is very much a net productivity multiplier for me.

> But giving all the answers without strong guidance on non-trivial architectural points— entropy. LLMs churning independently quickly devolve into entropy.

Typical iterative-circular process "write code -> QA -> fix remarks" works because the code is analyzable and "fix" is on average cheaper than "write", therefore the process, eventually, converges on a "correct" solution.

LLM prompting is on average much less analyzable (if at all) and therefore the process "prompt LLM -> QA -> fix prompt" falls somewhere between "does not converge" and "convergence tail is much longer".

This is consistent with typical observation where LLMs are working better: greenfield implementations of "slap something together" and "modify well structured, uncoupled existing codebase", both situations where convergence is easier in the first place, i.e. low existing entropy.

I very much agree with how you’ve categorized the initial state condition that is amenable to LLM assisted SWE and works well to a greater state of beneficial order. And implicitly I also agree most of the complement to that set of applied contexts yields ~medium to not so productive results.

But what do you mean by “LLM prompting is on average much less analyzable” ? Isn’t structured prompting (what that should optimally look like) the most objective and well defined part of the whole workflow. it’s the lowest entropy part of the situation, we know pretty well what a good LLM prompt is and what will be ineffective, even LLMs “know” that. Do you mean “context engineering” is hard to optimize around ? That’s often thought of interchangeably I think, but regardless that has in fact become the “hard problem” (user facing) in effectively leveraging LLM for dev work. Ever since the reasoning class models were introduced I think, it became more about context engineering in practice than prompting. Nowadays from the very onset Even resuming a session efficiently often requires a non-trivial approach that we’ve already started to design patterns and built tools around, (like CLI coding workflows adding /compact as user directive, etc).

I’m not a software engineer by trade, so I can’t pretend to know what that fully entails at the tail ends of enterprise scale and complexity, but I’ve spent a decent amount of time programming and as far as LLMs go, I think there’s probably somewhere down the road where we get so methodical about context engineering and tooling and memory management, all of the vast still somewhat nebulous surrounding space and scaffolding to LLM workflows that have a big impact on productive use of them—we may eventually engineer that aspect to an extent that will be able to much more consistently yield better results across more applied contexts than the “clean code”/“trivial app” dichotomy.
But … I think the depth of additional effort and knowledge and skill required by human user to do this optimal context engineering (once we fully understand how even) to get the best out of LLMs… I think that quickly just converges to — what it means to be a competent software engineer already. the meta layers around just “writing code” that are required to build robust systems and maintain them, the amount of work required to coerce non-deterministic models into effectively internalizing that, or at minimum not fvcking it up… that juice might not be worth the squeeze when it’s essentially what a good developer’s job is already. If that’s true then there will likely remain a ceiling of finite productivity you can expect from LLM assisted development for a long time… (I conjecture).

> Do you mean “context engineering” is hard to optimize around ? That’s often thought of interchangeably I think,

The so called "context" is part of the prompt.

> we may eventually engineer that aspect to an extent that will be able to much more consistently yield better results across more applied contexts than the “clean code”/“trivial app” dichotomy.

> the amount of work required to coerce non-deterministic models into effectively internalizing that,

That's, essentially, the point here. You write a prompt (or context, or memory, or whatever people want to call it to make themselves feel better), get code out, test the code and get test failures. Now what? Unless the problem is obvious lack of information in the prompt (i.e. something was not defined), there are no methodical ways to patch the prompt in a way that consistently fixes the error.

You can take program code, apply certain analytical rules on it and exhaustively define all the operations, states and side effects the program will have. That might be an extremely hard exercise to do in full, but in the end this is what it means to be analyzable. You can take a reduced set of rules and heuristics and quickly build a general structure of the operations and analyze deficiencies. If you are given a prompt, regardless of how well structured it is, you cannot, by definition, in general tell what the eventual output is going to look like without invoking the full ruleset (i.e. running the prompt through an LLM), therefore average fix of a prompt is effectively a full rewrite, which does not invoke the shortcut I have invoked.

They don't even really do that IME. If I ask Claude or ChatGPT to generate terraform for non-trivial but by no means obscure or highly unusual setups, they almost invariably hallucinate part of the answer even if a documented solution exists that isn't even that difficult. Maybe vibe coding JavaScript is that much better, or I'm just hopeless at prompting, but I feel a few dozen lines of fairly straightforward terraform config shouldn't require elaborate prompt setups, or I can just save some brain cycles by writing it myself.

For better or for worse have spent a large amount of time in terraform since 0.13 and I can confidently say LLM's are very, very bad at it. My favorite is when it invents internal functions (that look suspiciously like python) that do not exist, even when corrected, it will still keep going back to them. A year or two ago there were bad problems with hallucinated resource field names but I haven't seen that as much these days.

It however, is pretty good at refactoring given a set of constraints and an existing code base. It is decent at spitting out boilerplate code for well-known resources (such as AWS), but then again, those boilerplate examples are mostly coming straight from the documentation. The nice thing about refactoring with LLM's in terraform is, even if you vibe it, the refactor is trivially verifiable because the plan should show no changes, or the exact changes you would expect.

>I’ve come back to the idea LLMs are super search engines.

Yes! This is exactly what it is. A search engine with a lossy-compressed dataset of most public human knowledge, which can return the results in natural language. This is the realization that will pop the AI bubble if the public could ever bring themselves to ponder it en masse. Is such a thing useful? Hell yes! Is such a thing intellegent? Certainly NO!

That’s one of the most important features, though. For example, LLMs can analyze a code base and tell you how it works in natural language. That demonstrates functional understanding and intelligence - in addition to exceeding the abilities of the majority of humans in this area.

You’d need a very no-true-Scotsmanned definition of intelligence to be able exclude LLMs. That’s not to say that they’re equivalent to human intelligence in all respects, but intelligence is not an all-or-nothing property. (If it were, most humans probably wouldn’t qualify.)

LLMs being intelligence or not is not really that interesting. It's just matter of how you define intelligence. It matters maybe to the AI CEOs and their investors because of marketing.

What matters is how useful LLMs actually are. Many people here say it is useful as advanced search engine and not that useful as your coworker. That is very useful but most likely not something the AI companies want to hear.

> You’d need a very no-true-Scotsmanned definition of intelligence to be able exclude LLMs.

The thing is, that intelligence is an anthropocentric term. And has always been defined in a no-true-Scotsman way. When we describe the intelligence of other species we do so in extremely human terms (except for dogs). For example we consider dolphins smart when we see them play with each other, talk to each other, etc. We consider chimpanzees when we see them use a tool, row a boat, etc. We don’t consider an ant colony smart when they optimize a search for food sources, only because humans don’t normally do that. The only exception here are dogs, who we consider smart when they obey us more easily.

Personally, my take on this is that intelligence is not a useful term in philosophy nor science. Describing a behavior as intelligent is kind of like calling a small creature a bug. It is useful in our day to day speech, but fails when we want to build any theory around it.

In the context of "AI", the use of the word "intelligence" has referred to human-comparable intelligence for at least the last 75 years, when Alan Turing described the Turing Test. That test was explicitly intended to test for a particular kind of human equivalent intelligence. No other animal has come close to passing the Turing Test. As such, the distinction you're referring to isn't relevant to this discussion.

> Personally, my take on this is that intelligence is not a useful term in philosophy nor science.

The Turing test was debunked by John Searle in 1980 with the Chinese room thought experiment. And even looking past that, the existence, and the pervasiveness, of the Turing test proves my point that this term is and always has been extremely anthropocentric.

In statistics there has been a prevailing consensus for a really long time that artificial intelligence is not only a misnomer, but also rather problematic, and maybe even confusing. There has been a concerted effort the past 15 years to move away from this term onto something like machine learning (machine learning is not without its own set of downsides, but is still miles better then AI). So honestly my take is not that hot (at least not in statistics; maybe in psychology and philosophy).

But I want to justify my take in psychology. Psychometricians have been doing intelligence testing for well over a century now, and the science is not much further along then it was a century ago. No new prediction, no new subfields, etc. This is a hallmark of a scientific dead end. And on the flip side, psychological theories that don‘t use intelligence at all are doing just fine.

While I agree, I can't help but wonder: if such a "super search engine" were to have the knowledge on how to solve individual steps of problems, how different would that be from an "intelligent" thing? I mean that, instead of "searching" for the next line of code, it searches for the next solution or implementation detail, then using it as the query that eventually leads to code.

Having knowledge isn't the same as knowing. I can hold a stack of physics papers in my hand but that doesn't make me a physics professor.

LLMs possess and can retrieve knowledge but they don't understand it, and when people try to get them to do that it's like talking to a non-expert who has been coached to smalltalk with experts. I remember reading about a guy who did this with his wife so she could have fun when travelling to conferences with him!

I've spent a lot of time thinking about that - what if the realization that we need is not that LLMs are intelligent, but that our own brains work in the same way as the LLMs. There is certainly a cognitive bias to believe that humans are somehow special and that our brains are not simply machinery.

The difference, to me, is that an LLM can very efficiently recall information, or more accurately, a statistical model of information. However, they seem to be unable to actually extrapolate from it or rationalize about it (they can create the illusion of rationalization be knowing what the rationalization would look like). A human would never be able to ingest and remember the amount of information that an LLM can, but we seem to have the incredible ability of extrapolation - to reach new conclusions by deeply reasoning about old ones.

This is much like the difference in being "book smart" and "actually smart" that some people use to describe students. Some students can memorize vast amounts of information, pass all tests with straight A's, only to fail when they're tasked with thinking on their own. Others perform terribly on memorization tasks, but naturally are gifted at understanding things in a more intuitive sense.

I have seen heaps of evidence that LLMs have zero ability to reason, so I believe that there's something very fundamental missing. Perhaps the LLM is a small part of the puzzle, but there doesn't seem to be any breakthroughs that seem like we might be moving towards actual reasoning. I do think that the human brain can very likely be emulated if we cracked the technology. I just don't believe we're close.

Even though I think it's true that it's lossy, I think there is more going on in an LLM neural net. Namely that when it uses tokens to produce output, you essentially split the text into millions or billions of chunks, each with probability of those chunks. So in essence the LLM can do a form of pattern recognition where the patterns are the chunks and it also enables basic operations on those chunks.

That's why I think you can work iteratively on code and change parts of the code while keeping others, because the code gets chunked and "probabilitized'. It can also do semantic processing and understanding where it can apply knowledge about one topic (like 'swimming') to another topic (like a 'swimming spaceship', it then generates text about what a swimming spaceship would be which is not in the dataset). It chunks it into patterns of probability and then combines them based on probability. I do think this is a lossy process though which sucks.

Maybe it's looked down upon to complain about downvotes but I have to say I'm a little disappointed that there is a downvote with no accompanying post to explain that vote, especially to a post that is factually correct and nothing obviously wrong with it.

I personally had the completely opposite takeaway: Intelligence, at its core, really might just be a bunch of extremely good and self-adapting search heuristics.

We actually do, and often - depending on who our speaker is, our relationship with them, the tone of the message, etc. Maybe our intellect is not fully an LLM, but I truly wonder how much of our dialectical skills are.

You're describing the same answer with different phrasing.

Humans do that, LLMs regularly don't.

If you phrase the question "what color is your car?" a hundred different ways, a human will get it correct every time. LLMs randomly don't, if the token prediction veers off course.

Edit:

A human also doesn't get confused at fundamental priors after a reasonable context window. I'm perplexed that we're still having this discussion after years of LLM usage. How is it possible that it's not clear to everyone?

Don't get me wrong, I use it daily at work and at home and it's indeed useful, but there's is absolutely 0 illusion of intelligence for me.

that would be true if not for LLM making up answers where none exists.

Like, I've seen Claude go thru source code of the program, telling (correctly!) what counters are in code that return value I need (I just wanted to look at some packet metrics), then inventing entirely fake CLI command to extract those metrics

>It’s not readily apparent at first blush the LLM is doing this, giving all the answers.

Now I'm wondering if I'm prompting wrong. I usually get one answer. Maybe a few options but rarely the whole picture.

I do like the super search engine view though. I often know what I want, but e.g. work with a language or library I'm not super familiar with. So then I ask how do I do x in this setting. It's really great for getting an initial idea here.

Then it gives me maybe one or two options, but they're verbose or add unneeded complexity. Then I start probing asking if this could be done another way, or if there's a simpler solution to this.

Then I ask what are the trade-offs between solutions. Etc.

It's maybe a mix of search engine and rubber ducking.

Agents are, like for OP, a complete failure for me though. Still can't get them to not run off into a completely strange direction, leaving a minefield of subtle coding errors and spaghetti behind.

I’ve recently created many Claude skills to do repeatable tasks (architecture review, performance, magic strings, privacy, SOLID review, documentation review etc). The pattern is: when I’ve prompted it into the right state and it’s done what I want, I ask it to create a skill. I get codex to check the skill. I could then run it independently in another window etc and feed back to adjust…but you get the idea.

And almost every time it screws up we create a test, and often for the whole class of problem. More recent it’s been far better behaved. Between Opus, skills, docs, generating Mermaid diagrams, tests it’s been a lot better. I’ve also cleaned up so much of the architecture so there’s only one way to do things. This keeps it more aligned and helps with entropy. And they’ll work better as models improve. Having a match between code, documents and tests means it’s not just relying on one source.

Prompts like this seem to work: “what’s the ideal way to do this? Don’t be pragmatic. Tokens are cheaper than me hunting bugs down years later”

I'm not going to argue about how capable the models are, I personally think they are pretty capable.

What I will argue is that the LLMs are not just search engines. They have "compressed" knowledge. When they do this, they learn relations between all kinds of different levels of abstractions and meta patterns.

It is really important to understand that the model can follow logical rules and has some map of meta relationships between concepts.

Thinking of a LLM as a "search engine" is just fundamentally wrong in how they work, especially when connected to external context like code bases or live information.

Well, it's "a search engine that applies some transformations on top of the results" doesn't sound to me as a terrible way to think about LLMs.

> can follow logical rules

This is not their strong suite, though. They can only follow through a few levels on their own. This can be improved by agent-style iterations or via invoking external tools.

Let's see how this comment ages why don't we. I've understood where we are going and if you look at my comment history. I have confidence that in 12 months time. One opinion will be proved out with observations and the other will not.

For the "only few levels" claim, I think this one is sort of evident from the way they work. Solving a logical problem can have an arbitrary number of steps, and in a single pass there is only so many connection within a LLM to do some "work".

As mentioned, there are good ways to counter this problem (e.g. writing a plan and then iteratively going over those less-complex ones, or simply using the proper tool for the problem: use e.g. a SAT solver and just "translate" the problem to and from the appropriate format)

Nonetheless, I'm always open to new information/evidence and it will surely improve a lot in a year. As for reference, to date this is my favorite description of LLMs: https://news.ycombinator.com/item?id=46561537

There's been a notable jump over the course of the last few months, to where I'd say it's inevitable. For a while I was holding out for them to hit a ceiling where we'd look back and laugh at the idea they'd ever replace human coders. Now, it seems much more like a matter of time.

Ultimately I think over the next two years or so, Anthropic and OpenAI will evolve their product from "coding assistant" to "engineering team replacement", which will include standard tools and frameworks that they each specialize in (vendor lock in, perhaps), but also ways to plug in other tech as well. The idea being, they market directly to the product team, not to engineers who may have specific experience with one language, framework, database, or whatever.

I also think we'll see a revival of monolithic architectures. Right now, services are split up mainly because project/team workflows are also distributed so they can be done in parallel while minimizing conflicts. As AI makes dev cycles faster that will be far less useful, while having a single house for all your logic will be a huge benefit for AI analysis.

This doesn't make any sense. If the business can get rid of their engineers, then why can't the user get rid of the business providing the software? Why can't the user use AI to write it themselves?

I think instead the value is in getting a computer to execute domain-specific knowledge organized in a way that makes sense for the business, and in the context of those private computing resources.

It's not about the ability to write code. There are already many businesses running low-code and no-code solutions, yet they still have software engineers writing integration code, debugging and making tweaks, in touch with vendor support, etc. This has been true for at least a decade!

That integration work and domain-specific knowledge is already distilled out at a lot of places, but it's still not trivial. It's actually the opposite. AI doesn't help when you've finally shaved the yak smooth.

If the business can get rid of their engineers, then why can't the user get rid of the business providing the software?

A lot of businesses are the only users of their own software. They write and use software in-house in order to accomplish business tasks. If they could get rid of their engineers, they would, since then they'd only have to pay the other employees who use the software.

They're much less likely to get rid of the user employees because those folks don't command engineer salaries.

So instead of paying a human that "commands an engineer salary" then they'll be forced to pay whatever Anthropic or OpenAI commands to use their LLMs? I don't see how that's a better proposition: the LLM generates a huge volume of code that the product team (or whoever) cannot maintain themselves. Therefore, they're locked-in and need to hope the LLM can solve whatever issues they have, and if it can't, hope that whatever mess it generated can be fixed by an actual engineer without costing too much money.

Also, code is only a small piece and you still need to handle your hosting environment, permissions, deployment pipelines, etc. which LLMs / agentic workflows will never be able to handle IMO. Security would be a nightmare with teams putting all their faith into the LLM and not being able to audit anything themselves.

I don't doubt that some businesses will try this, but on paper it sounds like a money pit and you'd be better off just hiring a person.

It’s the same business model as consulting firms. Rather than hiring a few people for 65k each, a VP will bring in a consulting firm for 10M and get a bloated, half-working solution that costs even more to get working. The VP doesn’t care though because he ends up looking like a big shot in front of the other execs.

There are lots of developer agencies that hire developers as contractors that companies can use to outsource development to in a cheaper way without needing to pay for benefits or HR. They don't necessarily make bad quality software, but it doesn't feel humane.

Unless we're talking about some sketchy gig work nonsense, the "agency" is a consultancy like any other. They are a legitimate employer with benefits, w2, etc. It's not like they're pimps or something!

Those devs aren't code monkeys and they get paid the same as anyone else working in this industry. In fact, I think a lot of the more ADHD type people on here would strongly prefer working on a new project every 6 months without needing to find a new employer every time. The contracts between the consultancy and client usually also include longer term support than the limited time the original dev spent on it.

> If the business can get rid of their engineers, then why can't the user get rid of the business providing the software?

I have't checked the stats lately, but at one point most software written was in non-tech companies for the single business. The first 1/2 of my career was spent writing in-house software for a company that did everything from custom reporting and performance tracking to scraping data of automated phone dialers. There's so much software out there that effectively has a user base of a single company.

In some cases that could happen; in particular there may be a lot of UI and cross-app-integration style stuff that starts to get offloaded to users, so users can have AI code up their own UI for using some services together in the way that they want.

But in most cases businesses still need to own their own logic and data, so businesses will still be owning plenty of their own software. Otherwise customers could just write software to buy all your business's products for 99% off!

> Ultimately I think over the next two years or so, Anthropic and OpenAI will evolve their product from "coding assistant" to "engineering team replacement"

The way I see it, there will always be a layer in the corporate organization where someone has to interact with the machine. The transitioning layer from humans to AIs. This is true no matter how high up the hierarchy you replace the humans, be it the engineers layer, the engineering managers, or even their managers.

Given the above, it feels reasonable to believe that whatever title that person has—who is responsible for converting human management's ideas into prompts (or whatever the future has the text prompts replaced by)—that person will do a better job if they have a high degree of technical competence. That is to say, I believe most companies will still want and benefit if that/those employees are engineers. Converting non-technical CEO fever dreams and ambitions into strict technical specifications and prompts.

What this means for us, our careers, or Anthropic's marketing department, I cannot say.

That reminds me of the time where 3GL languages arrived and bosses claimed they no longer needed developers, because anyone could write code in those English-like languages.

Then when mouse-based tools like Visual Basic arrived, same story, no need for developers because anyone can write programs by clicking!

Now bosses think that with AI anyone will be able to create software, but the truth is that you'll still need software engineers to use those tools.

Will we need less people? Maybe. But in the past 40 years we have been increasing the developers productivity so many times, and yet we still need more and more developers because the needs have grown faster.

My suspicion is that it will be bad for salaries, mostly because it'll kill the "looks difficult" moat that software development currently has. Developers know that "understanding source code" is far from the hard part of developing software, but non-technical folks' immediate recoiling in the face of the moon runes has kept our profession pretty easy to justify high pay for for ages. If our jobs transition to largely "communing with the machines", then we'll go from a "looks hard, is hard" job, to a "looks easy, is hard" job, which historically hurts bargaining power.

I don't think "looks difficult" has been driving wages. FAANG etc leadership knows what's difficult and what's not. It's just marginal ROI. If you have a trillion-dollar market and some feature could increase that by 0.0001%, you hire some engineers to give it a try. If other companies are also competing for the same engineers for the same reasons, salaries skyrocket.

I wonder if the actual productivity changes won't end up mattering for the economics to change dramatically, but change in terms of a rebound in favour of seniors. If I was in school 2 years ago, looking at the career prospects and cost of living, I just straight up wouldn't invest in the career. If that happens at a large enough scale, the replenishment of the discipline may reduce, which would have an effect on what people who already had those skills could ask for. If the middle step, where wild magical productivity gains don't materialize in a way that reduces the need for expert software people who can reasonably be liable for whatever gets shipped, then we'll stick around.

Whether it looks easy or not doesn't matter as much imo. Plumbing looks and probably is easy, but it's not the CEOs job to go and fix the pipes.

I think this is the right take. In some narrow but constantly broadening contexts, agents give you a huge productivity edge. But to leverage that you need to be skilled enough to steer, design the initial prompt, understand the impact of what you produce, etc. I don't see agents in their current and medium term inception as being a replacement of engineering work, I see it as a great reshuffling of engineering work.

In some business contexts, the impact of more engineering labor on output gets capped at some point. Meaning once agent quality reaches a certain point, the output increase is going to be minimal with further improvements. There, labor is not the bottleneck.

In other business contexts, labor is the bottleneck. For instance it's the bottleneck for you as an individual: what kind of revenue could you make if you had a large team of highly skilled senior SWEs that operate for pennies on the dollar?

Labor will shift to where the ROI is highest is what I think you'll see.

To be fair, I can imagine a world where we eventually fully replace the "driver" of the agent in that it is good enough to fulfill the role of a ~staff engineer that can ingest very high level business context, strategy, politics and generate a high level system design that can then be executed by one or more agents (or one or more other SWEs using agents). I don't (at this point) see some fundamental rule of physics / economics that prevents this, but this seems much further ahead from where we are now.

I actually think it’s the opposite. We’ll see fewer monorepos because small, scoped repos are the easiest way to keep an agent focused and reduce the blast radius of their changes. Monorepos exist to help teams of humans keep track of things.

Could be. Most projects I've worked on tend to span multiple services though, so I think AI would struggle more trying to understand and coordinate across all those services versus having all the logic in a single deployable instance.

The way I see feature development in the future is, PM creates a dev cluster (also much easier with a monolith), has AI implement a bunch of features to spec, AI provides some feedback and gets input on anywhere it might conflict with existing functionality, whether eventual consistency is okay, which pieces are performance criticial, etc., and provides the implementation, a bunch of tests for review, and errata about where to find observability data, design decisions considered and chosen, etc. PM does some manual testing across various personas and products (along with PMs from those teams), has AI add feature flags, launches. The feature flag rollout ends up being the long-pole, since generally the product team needs to monitor usage data for some time before increasing the rollout percentage.

So I see that kind of workflow as being a lot easier in a monolithic service. Granted, that's a few years down the road though, before we have AI reliable enough to do that kind of work.

> Most projects I've worked on tend to span multiple services though, so I think AI would struggle more trying to understand and coordinate across all those services versus having all the logic in a single deployable instance.

1. At least CC supports multiple folders in a workspace, so that’s not really a limitation.

2. If you find you are making changes across multiple services, then that is a good indication that you might not have the correct abstraction on the service boundary. I agree that in this case a monolith seems like a better fit.

Agreed on both counts. Though for the first one it's still easier to implement things when bugs create compile or local unit/integration test errors rather than distributed service mismatches that can only be caught with extensive distributed e2e tests and a platform for running them, plus the lack of distribution cuts down significantly on the amount of code, edge cases, and deployment sequencing that needs to be taken into account.

For the second, yeah, but IME everything starts out well-factored, but almost universally evolves into spaghetti over time. The main advantage monoliths have is that they're safer to refactor across boundaries. With distributed services, there are a lot more backward-compatibility guarantees and concerns you have to work through, and it's harder to set up tests that exercise everything e2e across those boundaries. Not impossible, but hard enough that it usually requires a dedicated initiative.

If you research how something like Cursor works I don't think you would believe it is inevitable. The jump that would have to happen for it to replace engineers entirely is insurmountable. They can keep expanding contexts and coming up with clever ways to augment generation but I don't see it ever actually having full vision on the system, product and users.

Beyond that it is incredibly biased towards existing code & prompt content. If you wanted to build a voice chat app, and you said "should I use websockets or http?" It would say Websockets. It won't override you and say "Use neither, you should use webRTC", but an experienced engineer would spot that the prompt itself is flawed instantly. LLMs just will bias towards existing tokens in the prompt and won't surface data that would challenge the question itself.

Unless you, well, state in AGENTS.md that prompts may offer suboptimal options in which case it's the machine's duty to question them, treat the prompter like a coworker and not a boss.

Sit down and re-read your comment one night with your "I am an engineer and will solve this as an engineering problem" hat firmly on. If you stop thinking of LLMs as lobotimized coworkers trapped inside an API wrapper and instead as computational primitives then things become much more interesting and the future becomes clearer to see.

There's no chance LLMs will be an engineering team replacement. The hallucination problem is unsolvable and catastrophic in some edge cases. Any company using such a team would be uninsurable and sued into oblivion.

Writing software is actually one of the domains where hallucinations are easiest to fix: you can easily check whether it builds and passes tests.

If you want to go further, you can even require the LLM to produce a machine checkable proof that the software is correct. That's beyond the state of the art at the moment, but it's far from 'unsolvable'.

If you hallucinate such a proof, it'll just not work. Feed back the error message from the proof checker to your coding assistant, and the hallucination goes away / isn't a problem.

"...recently released LLMs, such as GPT-5, have a much more insidious method of failure. They often generate code that fails to perform as intended, but which on the surface seems to run successfully, avoiding syntax errors or obvious crashes. It does this by removing safety checks, or by creating fake output that matches the desired format, or through a variety of other techniques to avoid crashing during execution."

The trend for LLM generated code is to build and pass tests but do not deliver functionality needed.

I've recently seen Opus, after struggling for a bit, implement an API by having it return JSON that includes instructions for a human to manually accomplish the task I gave it.

> Haha. How do you reconcile a proof with actual code?

You can either proof your Rust code correct, or you can use a proof system that allows you to extract executable code from the proofs. Both approaches have been done in practice.

Rust code can have arbitrary I/O effects in any parts of it. This precludes using only Rust's type system to make sure code does what spec said.

The most successful formally proven project I know, seL4 [1], did not extracted executable code from the proof. They created a prototype in Haskell, mapped (by hand) it to Isabelle, I believe, to have a formal proof and then recreated code in C, again, manually.

As if Lean does not allow to circumvent it's proof system (the "sorry" keyword).

Also, consider adding code to the bigger system, written in C++. How would you use Lean to prove correctness of your code as part of the bigger system?

I mean, it's somewhat moot, as even the formal hypothesis ("what is this proof proving") can be more complex than the code that implements it in nontrivial cases. So verifying that the proof is saying the thing that you actually want it to prove can be near impossible for non-experts, and that's just the hypothesis; I'm assuming the proof itself is fully AI-generated and not reviewed beyond running it through the checker.

And at least in backend engineering, for anything beyond low-level algorithms you almost always want some workarounds: for your customer service department, for engineering during incident response, for your VIP clients, etc. If you're relying on formal proof of some functionality, you've got to create all those allowances in your proof algorithm (and hypothesis) too. And additionally nobody has really come up with a platform for distributed proofs, durable proof keys (kinda), or how to deal with "proven" functionality changes over time.

You focused on writing software, but the real problem is the spec used to produce the software, LLMs will happily hallucinate reasonable but unintended specs, and the checker won’t save you because after all the software created is correct w.r.t. spec.

Also tests and proof checkers only catch what they’re asked to check, if the LLM misunderstands intent but produces a consistent implementation+proof, everything “passes” and is still wrong.

This is why every one of my coding agent sessions starts with "... write a detailed spec in spec.md and wait for me to approve it". Then I review the spec, then I tell it "implement with red/green TDD".

Users don't want changes that rapidly. There's not enough people on the product team to design 20x more features. 20x more features means 400x more cross-team coordination. There's only positive marginal ROI for maybe 1.5-2x even if development is very cheap.

The premise is in progress. We are only at the beginning of the fourth year of this hype-phase, and we haven't even reached AGI yet. It's obviously not perfect, maybe never will, but we are not a the point yet were we can conclude which future is true. The singularity hasn't happend yet, so we are still moving with (llm-enhanced) human speed at the moment, meaning things need time.

Maybe, but you're responding to a thread about why AI might or might not be able to replace an entire engineering team:

> Ultimately I think over the next two years or so, Anthropic and OpenAI will evolve their product from "coding assistant" to "engineering team replacement", which will include standard tools and frameworks that they each specialize in (vendor lock in, perhaps), but also ways to plug in other tech as well.

This is the context of how this thread started, and this is the context in which DrammBA was saying that the spec problem is very hard to fix [without an engineering team].

Might be good to define the (legacy) engineering team. Instead of thinking 0/1 (ugh, almost nothing happens this way), the traditional engineering team may be replaced by something different. A team mostly of product, spec writers, and testers. IDK.

The job of AI is to do what we tell it to do. It can't "create a spec" on its own. If it did and then implemented that spec, it wouldn't accomplish what we want it to accomplish. Therefore we the humans must come up with that spec. And when you talk about a software application, the totality of its spec written out, can be very complex, very complicated. To write and understand, and evolve and fix such a spec takes engineers, or what used to be called "system analysts".

To repeat: To specify what a "system" we want to create does is a highly complicated task, which can only be dones by human engineers who understand the requirements for the system, and how parts of those requirements/specs interact with other parts of the spec, what are the consequences of one (part of the) spec to other parts of it. We must not writ e"impossible specs" like draw me a round square. Maybe the AI can check whether the spec is impossible or not, but I'm not so sure of that.

So I expect that software engineers will still be in high demand, but they will be much more productive with AI than without it. This means there will be much more software because it will be cheaper to produce. And the quality of the software will be higher in terms of doing what humans need it to do. Usability. Correctness. Evolvability. In a sense the natural language-spec we give the AI is really something written in a very high-level programming-language - the language of engineers.

BTW. As I write this I realize there is no spell-checker integrated into Hacker News. (Or is there?). Why? Because it takes developers to specify and implement such a system - which must be integrated into the current HN implementation. If AI can do that for HN, it can be done, because it will be cheap enough to do it -- if HN can exactly spell out what kind of system it wants. So we do need more software, better software, cheaper software, and AI will helps us do that.

A 2nd factor is that we don't really know if a spec is "correct" until we test the implemented system with real users. At that point we typically find many problems with the spec. So somebody must fix the problems with the spec, evolve the spec and rinse and repeat the testing with real users -- the developers who understand the current spec and why it is is not good enough.

AI can write my personal scripts for me surely. But writing a spec for a system to be used by thousands of humans, still takes a lot of (human) work. The spec must work for ALL users. That makes it complicated and difficult to get right.

Same, and similarly something like a "create a holistic design with all existing functionality you see in tests and docs plus new feature X, from scratch", then "compare that to the existing implementation and identify opportunities for improvement, ranked by impact, and a plan to implement them" when the code starts getting too branchy. (aka "first make the change easy, then make the easy change"). Just prompting "clean this code up" rarely gets beyond dumb mechanical changes.

Given so much of the work of managing these systems has become so rote now, my only conclusion is that all that's left (before getting to 95+% engineer replacement) is an "agent engineering" problem, not an AI research problem.

In order to prove safety you need a formal model of the system and formally defined safety properties that are both meaningful and understandable by humans. These do not exist for enterprise systems

An exhaustive formal spec doesn't exist. But you can conservatively proof some properties. Eg program termination is far from sufficient for your program to do what you want, but it's probably necessary.

(Termination in the wider sense: for example an event loop has to be able to finish each run through the loop in finite time.)

You can see eg Rust's or Haskell's type system as another light-weight formal model that lets you make and proof some simple statements, without having a full formal spec of the whole desired behaviour of the system.

Yeah, but with all respect, that is a totally uninteresting property in an enterprise software system where almost no software bugs actually manifest as non-termination.

The critical bugs here are related to security (DDoS attacks, authorization and authentication, data exfiltration, etc), concurrency, performance, data corruption, transactionality and so forth. Most enterprise systems are distributed or at least concurrent systems which depend on several components like databases, distributed lock managers, transaction managers, and so forth, where developing a proper formal spec is a monumental task and possibly impossible to do in a meaningful way because these systems were not initially developed with formal verification in mind. The formal spec, if faithful, will have to be huge to capture all the weird edge cases.

Even if you had all that, you need to actually formulate important properties of your application in a formal language. I have no idea how to even begin doing that for the vast majority of the work I do.

Proving the correctness of linear programs using techniques such as Hoare logic is hard enough already for anything but small algorithms. Proving the correctness of concurrent programs operating on complex data structures requires much more advanced techniques, setting up complicated logical relations and dealing with things like separation logic. It's an entirely different beast, and I honestly do not see LLMs as a panacea that will suddenly make these things scale for anything remotely close in size to a modern enterprise system.

That is true and very useful for software development, but it doesn't help if the goal is to remove human programmers from the loop entirely. If I'm a PM who is trying to get a program to, say, catalogue books according to the Dewey Decimal system for a library, a proof that the program terminates is not going to help that much when the program is mis-categorizing some books.

Is removing the human in the loop really the goal, or is the goal right now to make the human a lot more productive? Because...those are both very different things.

I don't know what the goal for OpenAI or Anthropic really is.

But the context of this thread is the idea that the user daxfohl launched that these companies will, in the next few years, launch an "engineering team replacement" program; and then the user eru claimed that this is indeed more doable in programming than other domains because you can have specs and tests for programs in a way that you can't for, say, an animated movie.

OK, so you successfully argued that replacing the entire engineering team is hard. But you can perhaps still shrink it by 99%. To the point where a sole founder can do the remaining tech role part time.

I have no idea what will happen in a few years, maybe LLM tech will hit a wall and humans will continue to be needed in the loop. But today humans are definitely needed in the loop in some way.

Cool. That sure sounds nice and simple. What do you do when the multiple LLMs disagree on what the correct tests are? Do you sit down and compare 5 different diffs to see which have the tests you actually want? That sure sounds like a task you would need an actual programmer for.

At some point a human has to actually use their brain to decide what the actual goals of a given task are. That person needs to be a domain expert to draw the lines correctly. There's no shortcut around that, and throwing more stochastic parrots at it doesn't help.

Just because you can't (yet) remove the human entirely from the loop, doesn't mean that economising on the use of the humans time is impossible.

For comparison have a look at compilers: nowadays approximately no one writes their software by hand, we write a 'prompt' in something like Rust or C, and ask another computer program to create the actual software.

We still need the human in the loop here, but it takes much less human time than creating the ELF directly.

It’s not “economizing” if I have to verify every test myself. To actually validate that tests are good I need to understand the system under test, and at that point I might as well just write the damn thing myself.

This is the fundamental problem with this “AI” mirage. If I have to be an expert to validate that the LLM actually did the task I set out, and isn’t just cheating on tests, then I might as well code the solution myself.

From a PM perspective, the main differentiator between an engineering team and AI is "common sense". As these tools get used more and more, enough training data will be available that AI's "common sense" in terms of coding and engineering decisions could be indistinguishable from a human's over time. At that point, the only advantage a human has is that they're also useful on the ops and incident response side, so it's beneficial if they're also comfortable with the codebase.

Eventually these human advantages will be overcome, and AI will sufficiently pass a "Turing Test" for software engineering. PMs will work with them directly and get the same kinds of guidance, feedback, documentation, and conversational planning and coordination that they'd get from an engineering team, just with far greater speed and less cost. At that point, yeah you'll probably need to keep a few human engineers around to run the system, but the system itself will manage the software. The advantage of keeping a human in the loop will dwindle to zero.

I can see how LLMs can help with testing, but one should never compare LLMs with deterministic tools like compilers. LLMs are entirely a separate category.

Tests and proofs can only detect issues that you design them to detect. LLMs and other people are remarkably effective at finding all sorts of new bugs you never even thought to test against. Proofs are particularly fragile as they tend to rely on pre/post conditions with clean deterministic processing, but the whole concept just breaks down in practice pretty quickly when you start expanding what's going on in between those, and then there's multithreading...

Ah, most the problem in programming is writing the tests. Once you know what you need the rest is just typing.

I can see an argument where you can get none programers to create the input and output of said tests but if the can do that, they are basically programmers.

This is of course leaving aside that half the stated use cases I hear for AI are that it can 'write the tests for you'. If it is writing the code and the tests it is pointless.

Well - the end result can be garbage still. To be fair: humans also write a lot of garbage. I think in general most software is rather poorly written; only a tiny percentage is of epic prowess.

Yes. But it's no different from the question of how a non-tech person can make sure that whatever their tech person tells them actually makes sense: you hire another tech person to have a look.

These types of comments are interesting to me. Pre-chatGPT there were tons of posts how so many software people were terrible at their jobs. Bugs were/are rampant. Software bugs caused high profile issues, but likely so many more we never heard about.

Today we have chatGPT and only now will teams be uninsurable and sued into oblivion? LOL

If you've ever used Claude Code in brave mode, I can't understand how you'd think a dev team could make the same categories of mistakes or with the same frequency.

I am but a lowly IC, with no notion of the business side of things. If I am an IC at, say, a FANG company, what insurance has been taken out on me writing code there?

> If I am an IC at, say, a FANG company, what insurance has been taken out on me writing code there?

Every non-trivial software business has liability insurance to cover them for coding lapses that lead to data breaches or other kinds of damages to customers/users.

I use LLM's to write the majority of my code. I haven't encountered a hallucination for the better part of a year. It might be theoretically unsolvable but it certainly doesn't seem like a real problem to me.

I use LLMs whenever I'm coding, and it makes mistakes ~80% of the time. If you haven't seen it make a huge mistake, you may not be experienced enough to catch them.

honestly i think they got the low hanging fruit already. they're bumping up against the limits of what it can do and while it's impressive it's not spectacular

Maybe I'm easily impressed, but that LLMs even work to output basic human-like text to me is bananas, and I do understand a bit of how it works, yet it's still up there as "Amazing that huge airplanes even can fly" is for me.

When coding style has been established among a team, or within an app, there are a lot of extra hoops to jump through, just to get it to look The Right Way, with no detectable benefit to the user.

If you put those choices aside and simply say: does it accomplish the goal per the spec (and is safe and scalable[0]), then you can get away with a lot more without the end user ever having a clue.

Sure, there's the argument for maintainability, and vibe coded monoliths tend to collapse in on themselves at ~30,000 LOC. But it used to be 2,000 LOC just a couple of years ago. Temporary problem.

[0]insisting that something be scalable isn't even necessary imo

You can control some simple things that way. But the subtle stylistic choices that many teams agree on are difficult to articulate clearly. Plus they don’t always do everything you tell them to in the prompts or rule files. Even when it’s extremely clear sometimes they just don’t. And often the thing you want isn’t clear.

Except the fact that the idioms and patterns used means that I can jump in and understand any part of the codebase, as I know it will be wired up and work the same as any other part.

I think here “to the user” is referring to the end user, not the programmer (the user of the coding style). There is a comprehension benefit for the team working on the code, but there is no direct¹ benefit to the end user.

--------

[1] The indirect benefits of there possibly being a faster release cadence and/or fewer bugs, could also be for many other reasons.

But you could say the same about tests, documentation, CI, issue trackers or really any piece of technology used. So it's not a very interesting statement if so.

> When coding style has been established among a team, or within an app, there are a lot of extra hoops to jump through, just to get it to look The Right Way, with no detectable benefit to the user.

Morphing an already decent PR into a different coding style is actually something that LLMs should excel at.

Do you have any references for "vibe coded monoliths tend to collapse in on themselves at ~30,000 LOC"? I haven't personally vibed up anything with that many LOC, so I'm legitimately curious if we have solid numbers yet for when this starts to happen (and for which definitions of "collapse").

Just my experience in vibe coding apps from gpt-3.5 onwards (mostly NextJS or Node). In gpt-3.5, I had to really hand-hold it, getting it to write one function at a time, then a separate task to glue the functions together.

Now, it can build almost all of an app from a single prompt, but will start to rewrite utility functions, or modules, forgetting that they already exist. Some of this is still solvable with clever prompting, but if you're just attacking it without thinking, ~30,000 LOC seems to be the app 'size' that it will start to exhibit those behaviors.

At work, I have the same difficulty using AI as you. When working on deep Jiras that require a lot of domain knowledge, bespoke testing tools, but maybe just a few lines of actual code changes across a vast codebase, I have not been able to use it effectively.

For personal projects on the other hand, it has expedited me what? 10x, 30x? It's not measurable. My output has been so much more than what would have been possible earlier, that there is no benchmark because these level of projects would not have been getting completed in the first place.

Back to using at work: I think it's a skill issue. Both on my end and yours. We haven't found a way to encode our domain knowledge into AI and transcend into orchestrators of that AI.

> deep Jiras that require a lot of domain knowledge, bespoke testing tools, but maybe just a few lines of actual code changes

How do new hires onboard? Do you spend days of your own time guiding them in person, do they just figure things out on their own after a few quarters of working on small tickets, or are things documented? Basically AI, when working on a codebase, has the same level of context that a new hire would have, so if you want them to get started faster then provide them with ample documentation.

> Do you spend days of your own time guiding them in person, do they just figure things out on their own after a few quarters of working on small tickets

It is this rather than docs. I think you're absolutely right about our lack of documentation handicapping AI agents.

After you review, instead of rewriting 70% of the code, have you tried to follow up with a message with a list of things to fix?

Also: in my experience 1. and 2. are not needed for you to have bad results. The existing code base is a fundamental variable. The more complex / convoluted it is, the worse is the result. Also in my experience LLMs are constantly better at producing C code than anything else (Python included).

I have the feeling that the simplicity of the code bases I produced over the years, and that now I modify with LLMs, and the fact they are mostly in C, is a big factor why LLMs appear to work so well for me.

Another thing: Opus 4.5 for me is bad on the web, compared to Gemini 3 PRO / GPT 5.2, and very good if used with Claude Code, since it requires to reiterate to reach the solution, why the others sometimes are better first-shotter. If you generate code via the web interface, this could be another cause.

> After you review, instead of rewriting 70% of the code, have you tried to follow up with a message with a list of things to fix?

This is one of my problems with the whole thing, at least from a programming PoV. Even though superficially it seems like the ST:TNG approach to using an intelligent but not aware computer as a tool to collaboratively solve a problem, it is really more like guiding a junior through something complex. While guiding a junior (or even some future AGI) in that way is definitely a good thing, if I am a good guide they will learn from the experience so it will be a useful knowledge sharing process, that isn't a factor for an LLM (at least not the current generations). But if I understand the issue well enough to be a good guide, and there is no teaching benefit external to me, I'd rather do it myself and at most use the LLM as a glorified search engine to help muddle through bad documentation for hidden details.

That and TBH I got into techie things because I like tinkering with the details. If I thought I'd not dislike guiding others doing the actual job, I'd have not resisted becoming a manager throughout all these years!

> After you review, instead of rewriting 70% of the code, have you tried to follow up with a message with a list of things to fix?

I think this is the wrong approach, already by having "wrong code" in the context, makes every response after this worse.

Instead, try restarting, but this time specify exactly how you expected that 70% of the code to actually have worked, from the get go. Often, LLMs seem to make choices because they have to, and if you think they made the wrong choice, you can often find that you didn't actually specify something well enough, hence the LLM had to do something, since apparently the single most important thing for them is that they finish something, no matter how right or wrong.

After a while, you'll get better at knowing what you have to be precise, specific and "extra verbose" about, compared to other things. Something that also seems to depend on the model, like with how Gemini you can have 5 variations of "Don't add any comments" yet it does anyways, but say that once to GPT/Claude-family of models and it seems they get it at once.

There are some problems where this becomes a game of whack-a-mole either way you approach it (restart or modify with existing context). I end up writing more prompts than the code I could've written myself.

This isn't to say I don't think LLMs are an asset, they have helped me solve problems and grow in domains where I lack experience.

The biggest frustration with LLMs for me is people telling me I'm not promoting it in a good way. Just think about any product where they are selling a half baked product, and repeatedly telling the user you are not using it properly.

If you buy a table saw and can't figure out how to cut a straight line in a piece of wood with it - or keep cutting your fingers off - but didn't take any time at all to learn how to use it, that's on you.

Likewise a car, you have to take lessons and a test before you can use those!

Even if they're your boss? Remember that most people here are not independently wealthy, they're stuck answering to someone who may not have so level a take on these things as you do.

> and you'll be rehired and have to clean up the mess, then continue on

Not how this works. Yes, it happens sometimes, but there's no guarantee. Alternatives include:

- The rest of your team (or another tam) soaks up the additional work by working longer hours

- They hire someone else, or transfer someone from elsewhere

- The company accepts the lower output quality / whatever breakages result

- The breakages, even if unacceptable, only show up months down the line

So all that needs to happen is for your boss to believe they can replace you up to the point where they feel comfortable firing you. Whether that works or not is largely immaterial to the impact it thereafter has on your ability to pay rent / your mortgage / etc.

The fact that you could be fired at any time hasn't changed. That was true before any of this. Maybe this is a wake up call that it's a real risk, but the risk was always there and should be planned for

The more important thing though is that if LLMs can't replace people (remains to be seen) they won't lead to a net job loss. You'll find something else

The problem there is the boss, not the technology. If it isn’t an insane take on AI, it’d be on something else, and eventually will be. People quit bad managers, not bad jobs. If you have a bad manager, work on quitting them.

I think the problem is the techno fascist oligarchs that are peddling the snake oil that LLMs will wipe out all white collar jobs tomorrow. Managers usually answer to C suite, and the C suite is salivating at the idea of laying off 80% of staff

FWIW, I left my full time job some years ago to do my own thing, in part because pushing back on bad decisions was not really doing me any favors for my mental health. Glad to report I'm in a much better place after finding the courage to get out of that abusive relationship.

Some might argue the risk of not pushing back is far worse.

I was a contractor/consultant between 2020-2023; I have a problem w/ authority so it suited me. But work/life balance was awful--I have 2 kids now, and I can't do nothing for 6 weeks then work 100 hour weeks for 4 weeks. The maximum instability my life will tolerate is putting the kids to bed at 9 instead of 8:30 lol. I'm also in the Netherlands so there's also other benefits. Worker protections are very strong here, so it's highly unlikely I'll be fired or laid off; I can't be asked to work overtime; I can't be Slack'd after hours; I can drop down to 4 days a week no questions asked, when the kids were born I got a ton of paid leave, etc. Not to imply I work at some awful salt mine; I like my current gig and coworkers/leadership.

Anyway, this is a collective action problem. I don't take any responsibility for the huge plastic island in the Pacific, nor do I take any responsibility for the grift economy built on successive, increasingly absurd hype waves of tech (web 2.0, mobile, SPAs, big data, blockchain, VR, AI). I've also worked in social good, from Democratic presidential campaigns and recounts to helping connect people w/ pro bono legal services, which is to say I've done my time. There are too many problems for me to address, I get to pick which, if any, I battle, I am happy if my kids don't meltdown too much during the evening. Maybe when they're both in school I can take more risks or reformulate my work/life balance, but currently I'm focused on furthering the human race.

Same as any other technology. If MongoDB tell you that their solution is "web scale" it's still on you to evaluate that claim before picking the database platform to build your company on.

> If you buy a table saw and can't figure out how to cut a straight line in a piece of wood with it - or keep cutting your fingers off - but didn't take any time at all to learn how to use it, that's on you.

Of course - that's deterministic, so if you make a mistake and it comes out wrong, you can fix the mistake you made.

> Why should LLMs be any different?

Because they are not deterministic; you can't use experience with LLMs in any meaningful way. They may give you a different result when you run the same spec through the LLM a second time.

> Because they are not deterministic; you can't use experience with LLMs in any meaningful way. They may give you a different result when you run the same spec through the LLM a second time.

Lots of things, and indeed humans, are also as non-deterministic; I absolutely do use experience working with humans and non-deterministic things to improve my future interactions with them.

Table saws are kinda infamous in this regard: you may say that kick-back is hidden state/incomplete information rather than non-deterministic, but in practice the impact is the same.

> They may give you a different result when you run the same spec through the LLM a second time.

Yes kind of, but only different results (maybe) for the things you didn't specify. If you ask for A, B and C, and the LLM automatically made the choice to implement C in "the wrong way" (according to you), you can retry but specify exactly how you want C to be implemented, and it should follow that.

Once you've nailed your "spec" enough so there isn't any ambiguity, the LLM won't have to make any choices for you, and then you'll get exactly what you expected.

Learning this process, and learning how much and what exactly you have to instruct it to do, is you building up your experience learning how to work with an LLM, and that's meaningful, and something you get better with as you practice it.

> Yes kind of, but only different results (maybe) for the things you didn't specify.

No. They will produce a different result for everything, including the things you specify.

It's so easy to verify that I'm surprised you're even making this claim.

> Once you've nailed your "spec" enough so there isn't any ambiguity, the LLM won't have to make any choices for you, and then you'll get exactly what you expected

1. There's always ambiguity, or else you'll end up an eternity writing specs

2. LLMs will always produce different results even if the spec is 100% unambiguous for a huge variety of reasons, the main one being: their output is non-deterministic. Except in the most trivial of cases. And even then the simple fact of "your context window is 80% full" can lead to things like "I've rewritten half of your code even though the spec only said that the button color should be green"

> It's so easy to verify that I'm surprised you're even making this claim.

Well, to be fair, I'm surprised you're even trying to say this claim isn't true, when it's so easy to test yourself.

If I prompt "Create a function with two arguments, a and b, which returns adding those two together", I'll get exactly what I specify. If I feel like it using u8 instead of u32 was wrong, I add "two arguments which are both u8", then you now get this.

Is this not the experience you get when you use LLMs? How does what you get differ from that?

> 1. There's always ambiguity, or else you'll end up an eternity writing specs

There isn't though, at one point it does end. If it's worth going so deep into specifying the exact implementation is up to you and what you're doing, sometimes it is, sometimes it isn't.

> LLMs will always produce different results even if the spec is 100% unambiguous for a huge variety of reasons, the main one being: their output is non-deterministic.

Again, it's so easy to verify that this isn't true, and also surprising you'd say this, because earlier you say "always ambiguity" yet somehow you seem to also know that you can be 100% unambiguous.

Like with "manual" programming, the answer is almost always "divide and conquer", when you apply that with enough granularity, you can reach "100% umambiguity".

> And even then the simple fact of "your context window is 80% full" can lead to things like "I've rewritten half of your code even though the spec only said that the button color should be green"

Yes, this is a real flaw, once you go beyond two messages, the models absolutely lose track almost immediately. Only workaround for this is constantly restarting the conversation. I never "correct" an agent if they get it wrong with more "No, I meant", I rewrite my first message so there are no corrections needed. If your context goes beyond ~20% of what's possible, you're gonna get shit results basically. Don't trust the "X tokens context length", because "what's possible" is very different from "what's usable".

> If I prompt "Create a function with two arguments, a and b, which returns adding those two together", I'll get exactly what I specify. If I feel like it using u8 instead of u32 was wrong, I add "two arguments which are both u8", then you now get this.

This is actually a good example of how your spec will progress:

First pass: "Create a function [in language $X] with two arguments, a and b, which returns adding those two together"

Second pass: "It must take u8 types, not u32 types"

Third pass: "You are not handling overflows. It must return a u8 type."

Fourth pass: "Don't clamp the output, and you're still not handling overflows"

Fifth pass: "Don't panic if the addition overflows, return an error" (depending on the language, this could be "throw an exception" or return a tuple with an error field, or use an out parameter for the result or error)

For just a simple "add two numbers" function, the specification can easily exceed the actual code. So you can probably understand the skepticism when the task is not trivial, and depends on a lot of existing code.

So you do know how the general "writing specification" part is working, you just have the wrong process. Instead of iterating and adding more context on top, restructure your initial prompt to include the context.

DONT DO:

First pass: "Create a function [in language $X] with two arguments, a and b, which returns adding those two together"

Second pass: "It must take u8 types, not u32 types"

INSTEAD DO:

First pass: "Create a function [in language $X] with two arguments, a and b, which returns adding those two together"

Second pass: "Create a function [in language $X] with two arguments, a and b, both using u8, which returns adding those two together"

----

What you don't want to do, is adding additional messages/context on top of "known bad" context, so instead you should take the clue that the LLM didn't understand correctly as "I need to edit my prompt" not "I need to now after their reply, add more context to correct what was wrong". The goal should be to completely avoid anything bad, not correct it.

Together with this, you build up a system/developer prompt you can reuse across projects/scopes, that follows how you code. In that, you add stuff as you discover what's needed to be added, like "Make sure to always handle Exceptions in X way" or similar.

> > For just a simple "add two numbers" function, the specification can easily exceed the actual code. So you can probably understand the skepticism when the task is not trivial, and depends on a lot of existing code.

Yes, please be skeptical, I am as well, which I guess is why I am seemingly more effective at using LLMs than others who are less skeptical. It's a benefit here to be skeptical, not a drawback.

And yes, it isn't trivial to verify work that others have done for you, when you have a concrete idea of how it should be exactly. But as I managed to work with outsourced/contracting developers before, or even collaborate with developers in the same company as me, I also learned to use LLMs in a similar way where you have to review and ensure code follow the architecture/design you intended.

> First pass: "Create a function [in language $X] with two arguments, a and b, which returns adding those two together"

> Second pass: "Create a function [in language $X] with two arguments, a and b, both using u8, which returns adding those two together"

So it will create two different functions (and LLMs do love to ignore anything that came before and create a lot of stuff from scratch again and again). Now what.

What? No, I think you fundamentally misunderstand what workflow I'm suggesting here.

You ask: "Do X". The LLM obliges, gives you something you don't want. At this point, don't accept/approve it, so nothing has changed, you still have an empty directory, or whatever.

Then you start a brand new context, with iteration on the prompt: "Do X with Y", and the LLM again tries to do it. If something is wrong, repeat until you get what you're happy with, extract what you can into reusable system/developer prompts, then accept/approve the change.

Then you end up with one change, and one function, exactly as you specified it. Then if you want, you can re-run the exact same prompt, with the exact same context (nothing!) and you'll get the same results.

"LLMs do love to ignore anything that came before" literally cannot happen in this workflow, because there is nothing that "came before".

> No, I think you fundamentally misunderstand what workflow I'm suggesting here.

Ah. Basically meaningless monkey work of baby sitting an eager junior developer. And this is for a simple thing like adding two numbers. See how it doesn't scale at all with anything remotely complex?

> "LLMs do love to ignore anything that came before" literally cannot happen in this workflow, because there is nothing that "came before".

Of course it can. Because what came before is the project you're working on. Unless of course you end up specifying every single utility function and every single library call in your specs. Which, once again, doesn't scale.

> See how it doesn't scale at all with anything remotely complex?

No, I don't. Does outsourcing not work for you with "anything remotely complex"? Then yeah, LLMs won't help you, because that's a communication issue. Once you figure out how to communicate, using LLMs even for "anything remotely complex" becomes trivial, but requires an open mind.

> Because what came before is the project you're working on.

Right, if that's what you meant, then yeah, of course they don't ignore the existing code, if there is a function that already does what it needs, it'll use that. If the agent/LLM you use doesn't automatically does this, I suggest you try something better, like Codex or Claude Code.

But anyways, you don't really seem like you're looking for improving, but instead try to dismiss better techniques available, so I'm not even sure why I'm trying to help you here. Hopefully at least someone who wants to improve comes across it so this whole conversation wasn't a complete waste of time.

Strange. For a simple "add two integers" you now have to do five different updates to specs to make it non-ambiguous, restarting the work from scratch (that is, starting a new context) every time.

What happens when your work isn't to add two integers? How many iterations of the spec you have to do before you arrive at an unambiguous one, and how big will it be?

> Once you figure out how to communicate,

LLMs don't communicate.

> Right, if that's what you meant, then yeah, of course they don't ignore the existing code, if there is a function that already does what it needs, it'll use that.

Of course it won't since LLMs don't learn. When you start a new context, the world doesn't exist. It literally has no idea what does and does not exist in your project.

It may search for some functionality given a spec/definition/question/brainstorming skill/thinking or planning mode. But it may just as likely not. Because there are no actual proper way for anyone to direct it, and the models don't have learning/object permanence.

> If the agent/LLM you use doesn't automatically does this, I suggest you try something better, like Codex or Claude Code.

The most infuriating thing about these conversations is that people hyping AI assume everyone else but them is stupid, or doing something incorrectly.

We are supposed to always believe people who say "LLMs just work", without any doubt, on faith alone.

However, people who do the exact same things, use the exact tools, and see all the problems for what they are? Well, they are stupid idiots with skill issues who don't know anything and probably use GPT 1.0 or something.

Neither Claude nor Codex are magic silver bullets. Claude will happily reinvent any and all functions it wants, and has been doing so since the very first day it was unleashed onto the world.

> But anyways, you don't really seem like you're looking for improving, but instead try to dismiss better techniques available

Yup. Just as I said previously.

There are some magical techniques, and if you don't use them, you're a stupid Luddite idiot.

Doesn't matter that the person talking about these magical techniques completely ignores and misses the whole point of the conversation and is fully prejudiced against you. The person who needs to improve for some vague condescending definition of improvement is you.

Similarly, some humans seem to unable to too. The problem is, you need to be good at communication to effectively use LLMs, judging by this thread, it's pretty clear what the problem is. I hope you figure it out someday, or just ignore LLMs, no one is forcing you to use them (I hope at least).

I don't mind what you do, and I'm not "hyping LLMs", I see them as tools that are sometimes applicable. But even to use them in that way, you need to understand how to use them. But again, maybe you don't want, that's fine too.

"However, people who do the exact same things, use the exact tools, and see all the problems for what they are? Well, they are stupid idiots with skill issues who don't know anything and probably use GPT 1.0 or something."

It seems generally agreed that LLMs (currently) do better or worse with different programming languages at least, and maybe with other project logistical differences.

The fact that an LLM works great for one user on one project does not mean it will work equally great for another user on a different project. It might! It might work better. It might work worse.

And both users might be using the tool equally well, with equal skill, insofar as their part goes.

I'm glad you brought up the power tool analogy - I've bought a $40 soldering iron once, which looked just like the Weller that cost like 5x as much. There was nothing wrong with it on the surface, it was well built and heated up just fine.

But every time i tried to solder with it, the results sucked. I couldn't articulate why, and assumed I was doing something wrong (I probably was).

Then at my friends house, I got to try the real thing, and it worked like a dream. Again I can't pin down why, but everything just worked.

This is how I felt with LLMs (and image generation) - sometimes it just doesn't feel right, and I can't put my finger on what should I fix, but I come away often with the feeling that I needed to do way more tweaking than necessary and the results were just still mediocre.

No one knows what the actual "right way" to hold (prompt) an LLM is. A certain style or pattern to prompting may work in one scenario for one LLM, but change the scenario or model and it often loses any advantage and can give worse output than a different style/pattern.

In contrast table saws and cars have pretty clear rules of operation.

Table saws and cars are deterministic. Once uou learn how to use them, the experience is repeatable.

The various magic incantations that LLMs require cannot be learned or repeated. Whatever the "just one more prompt bro" du jour you're thinking of may or may not work at any given time for any given project in any given language.

Operating a car (i.e. driving) is certainly not deterministic. Even if you take the same route over and over, you never know exactly what other drivers or pedestrians are going to do, or whether there will be unexpected road conditions, construction, inclement weather, etc.
But through experience, you build up intuition and rules of thumb that allow you to drive safely, even in the face of uncertainty.

It's the same programming with LLMs. Through experience, you build up intuition and rules of thumb that allow you to get good results, even if you don't get exactly the same result every time.

> It's the same programming with LLMs. Through experience, you build up intuition and rules of thumb that allow you to get good results, even if you don't get exactly the same result every time.

Friend, you have literally described a nondeterministic system. LLM output is nondeterministic. Identical input conditions result in variable output conditions. Even if those variable output conditions cluster around similar ideas or methods, they are not identical.

The problem is that this is completely false. LLMs are actually deterministic. There are a lot more input parameters than just the prompt. If you're using a piece of shit corpo cloud model, you're locked out of managing your inputs because of UX or whatever.

Ah, we've hit the rock bottom of arguments: there's some unspecified ideal LLM model that is 100% deterministic that will definitely 100% do the same thing every time.

We've hit rock bottom of rebuttals, where not only is domain knowledge completely vacant, but you can't even be bothered to read and comprehend what you're replying to. There is no non-deterministic LLM. Period. You're already starting off from an incoherent position.

Now, if you'd like to stop acting like a smug ass and be inquisitive as per the commenting guidelines, I'd be happy to tell you more. But really, if you actually comprehended the post you're replying to, there would be no need since it contains the piece of the puzzle you aren't quite grasping.

Strange then that the vast majority of LLMs that people use produce non-deterministic output.

Funnily enough I had literally the same argument with someone a few months back in a friends group. I ran the "non-shitty non-corpo completely determenistic model" through ollama... And immediately got two different answers for the same input.

> Now, if you'd like to stop acting like a smug ass and be inquisitive as per the commenting guidelines,

Ah. Commenting guidelines. The ones that tell you not to post vague allusions to something, not to be dismissive of what others are saying, responding to the strongest plausible interpretation of someone says etc.? Those ones?

> Strange then that the vast majority of LLMs that people use produce non-deterministic output.

> I ran the "non-shitty non-corpo completely determenistic model" through ollama... And immediately got two different answers for the same input.

With deterministic hardware in the same configuration, using the same binaries, providing the same seed, the same input sequence to the same model weights will produce bit-identical outputs. Where you can get into trouble is if you aren't actually specifying your seed, or with non-deterministic hardware in varying configurations, or if your OS mixes entropy with the standard pRNG mechanisms.

Inference is otherwise fundamentally deterministic. In implementation, certain things like thread-scheduling and floating-point math can be contingent on the entire machine state as an input itself. Since replicating that input can be very hard on some systems, you can effectively get rid of it like so:

ollama run [whatever] --seed 123 --temperature 0 --num-thread 1

A note that "--temperature 0" may not strictly be necessary. Depending on your system, setting the seed and restricting to a single thread will be sufficient.

These flags don't magically change LLM formalisms. You can read more about how floating point operations produce non-determinism here:

In this context, forcing single-threading bypasses FP-hardware's non-associativity issues that crop up with multi-threaded reduction. If you still don't have bit-replicated outputs for the same input sequence, either something is seriously wrong with your computer or you should get in touch with a reputable metatheoretician because you've just discovered something very significant.

> Those ones?

Yes those ones. Perhaps in the future you can learn from this experience and start with a post like the first part of this, rather than a condescending non-sequitur, and you'll find it's a more constructive way to engage with others. That's why the guidelines exist, after all.

> These flags don't magically change LLM formalisms. You can read more about how floating point operations produce non-determinism here:

Basically what you're saying is "for 99.9% of use cases and how people use them they are non-deterministic, and you have to very carefully work around that non-determinism to the point of having workarounds for your GPU and making them even more unusable"

> In this context, forcing single-threading bypasses FP-hardware's non-associativity issues that crop up with multi-threaded reduction.

Translation: yup, they are non-deterministic under normal conditions. Which the paper explicitly states:

--- start quote ---

existing LLM serving frameworks exhibit non-deterministic behavior: identical inputs can yield different outputs when system configurations (e.g., tensor parallel (TP) size, batch size) vary, even under greedy decoding. This arises from the non-associativity of floating-point arithmetic and inconsistent reduction orders across GPUs.

--- end quote ---

> If you still don't have bit-replicated outputs for the same input sequence, either something is seriously wrong with your computer or you should get in touch with a reputable metatheoretician because you've just discovered something very significant.

Basically what you're saying is: If you do all of the following, then the output will be deterministic:

- workaround for GPUs with num_thread 1

- temperature set to 0

- top_k to 0

- top_p to 0

- context window to 0 (or always do a single run from a new session)

> The problem is that this is completely false. LLMs are actually deterministic. There are a lot more input parameters than just the prompt. If you're using a piece of shit corpo cloud model, you're locked out of managing your inputs because of UX or whatever.

When you decide to make up your own definition of determinism, you can win any argument. Good job.

Yes, that's my point. Neither driving nor coding with an LLM is perfectly deterministic. You have to learn to deal with different things happening if you want do do either successfully.

> Neither driving nor coding with an LLM is perfectly deterministic.

Funny.

When driving, I can safely assume that when I turn the steering wheel in the direction in turns. That the road that was there yesterday is there today (barring certain emergencies, that's why they are emergencies). That the red light in a traffic light means stop, and the green means go.

And not the equivalent "oh, you're completely right, I forgot to include the wheels, wired the steering wheel incorrectly, and completely messed up the colors"

> Operating a car (i.e. driving) is certainly not deterministic.

Yes. Operating a car or a table saw is deterministic. If you turn your steering wheel left, the car will turn left every time with very few exceptions that can also be explained deterministically (e.g. hardware fault or ice on road).

Claiming "completely" is mapping a boolean to a float.

If you tell an LLM (with tools) to do a web search, it usually does a web search. The biggest issue right now is more at the scale of: if you tell it to create turn-by-turn directions to navigate across a city, it might create a python script that does this perfectly with OpenStreetMap data, or it may attempt to use its own intuition and get lost in a cul-de-sac.

Wow. It can do a web search. And that is useful in the context of programming how? Or in any context?

The question is about the result of an action. Given the same problem statement in the same codebase it will produce wildly different results even if prompted two times in a row.

Even for trivial tasks the output may vary between just a simple fix, and a rewrite of half of the codebase. You can never predict or replicate the output.

To quote Douglas Adams, "The ships hung in the sky in much the same way that bricks don't". Cars and table saws operate in much the same way that LLMs don't.

> Wow. It can do a web search. And that is useful in the context of programming how? Or in any context?

Your own example was turning a steering wheel.

A web search is as relevant to the broader problems LLMs are good at, as steering wheels are to cars.

> Given the same problem statement in the same codebase it will produce wildly different results even if prompted two times in a row.

Do you always drive the same route, every day, without alteration?

Does it matter?

> You can never predict or replicate the output.

Sure you can. It's just less like predicting what a calculator will show and more like predicting if, when playing catch, the other player will catch your throw.

You can learn how to deal with reality even when randomness is present, and in fact this is something we're better at than the machines.

The original example was trying to compare LLMs to cars and table saws.

> Do you always drive the same route, every day, without alteration?

I'm not the one comparing operating machinery (cars, table saws) to LLMs. Again. If I turn a steering wheel in a car, the car turns. If input the same prompt into an LLM, it will produce different results at different times.

Lol. Even "driving a route" is probably 99% deterministic unlike LLMs. If I follow a sign saying "turn left", I will not end up in a "You are absolutely right, there shouldn't be a cliff at this location" situation.

Edit: and when signs end pointing to a cliff, or when a child runs onto the roads in front of you, these are called emergency situations. Whereas emergency situations are the only available modus operandi for an LLM, and actually following instructions is a lucky happenstance.

> It's just less like predicting what a calculator will show and more like predicting if, when playing catch, the other player will catch your throw

If you think that throwing more and more bad comparisons that don't work into the conversation somehow proves your point, let me dissuade you of that notion: it doesn't.

Now imagine the table saw is really, REALLY shit at being table saw and saw no straight angle anywhere during its construction. And they come with new one every 6 months that is very slightly less crooked but controls are all moved over so you have to tweak your workflow

It's not anyone's job to "promote it in a good way", we have no responsibility either for or against such tech.

The analogy would be more like: "yeah, the motor blew up and burned your garage, but please don't be negative - we need you to promote this saw in a good way".

Sure, it's important to "hold it right", but we're not in some cult here where we need to all sell this tech well beyond its current or future potential.

Have you seen the way some people google/prompt? It can be a murder scene.

Not coding related but my wife is certainly better than most and yet I’ve had to reprompt certain questions she’s asked ChatGPT because she gave it inadequate context. People are awful at that. Us coders are probably better off than most but just as with human communication if you’re not explaining things correctly you’re going to get garbage back.

People are "awful at that" because when two people communicate, we're using a lot more than words. Each person participating in a conversation is doing a lot of active bridge-building. We're supplying and looking for extra nonverbal context; we're leaning on basic assumptions about the other speaker, their mood, their tone, their meanings; we're looking at not just syntax but the pragmatics of the convo (https://en.wikipedia.org/wiki/Pragmatics). The communication of meaning is a multi-dimensional thing that everyone in the conversation is continually contributing to and pushing on.

In a way, LLMs are heavily exploitative of human linguistic abilities and expectations. We're wired so hard to actively engage and seek meaning in conversational exchanges that we tend to "helpfully" supply that meaning even when it's absent. We are "vulnerable" to LLMs because they supply all the "I'm talking to a person" linguistic cues, but without any form of underlying mind.

Folks like your wife aren't necessarily "bad" at LLM prompting—they're simply responding to the signals they get. The LLM "seems smart." It seems like it "knows" things, so many folks engage with them naturally, as they would with another person, without painstakingly feeding in context and precisely defining all the edges. If anything, it speaks to just how good LLMs are at being LLMs.

Until we get LLMs with deterministic output for a given prompt, there's no guarantee that you and me typing the same prompt will yield a working solution of similar quality.

I agree that it helps to add context, but then again assuming people aren't already doing it doesn't help in any way. You can add all the context there is and still get a total smudge out of it. You can select regenerate a few times and it's no better. There's nothing indisputably proving which part of your prompt the LLM will fixate on more and which one it will silently forget (this one's even more apparent with longer prompts).

It's like you buy Visual Studio and don't believe anyone who tells you that it's complex software with a lot of hidden features and settings that you need to explore in order to use it to its full potential.

I feel it's not worth the effort to spend time and learn the hidden features. whenever I use it to plug something new into a existing codebase it either gives something good at first shot or repeat the non working solution again and again. after such session I only get a feeling instead of spending the last 15 minutes on prompting this, I should have learnt these stuff and this learning would be useful for me forever.

I use LLMs as a better form of search engines and that's a useful product.

> I feel it's not worth the effort to spend time and learn the hidden features.

And that's the only issue here. Many programmers feel offended by an AI threatening their livelihood, and are too arrogant to invest some time in a tool they do deem below themselves—then proceed to complain how useless the tool is on the internet.

I'd really suggest taking antirez' advice at heart, and invest time in actually learning how to work with AI properly. Just because Claude Code has a text prompt like ChatGPT doesn't mean you know how to work with it yet. It is going to pay off.

> I should have learnt these stuff and this learning would be useful for me forever.

Oh, if only software worked like that.

Even a decade ago, one could reasonably say that half of what we proudly add to our CVs becomes obsolete every 18 months, it's just hard to predict which half.

A coding agent just beat every human in the AtCoder Heuristic optimization contest. It also beat the solution that the production team for the contest put together.
https://sakana.ai/ahc058/

It's not enterprise-grade software, but it's not a CRUD app with thousands of examples in github, either.

C'mon, there's post every other week that optimization never happens anymore because it's too hard. If AI can take all the crap code humans are writing and make it better, that sounds like a huge win.

Simple is the opposite of complex; the opposite of hard is easy. They are orthogonal. Chess is simple and hard. Go is simpler and harder than chess.

Program optimization problems are less simple than both, but still simpler than free-form CRUD apps with fuzzy, open ended acceptance criteria. It would stand to reason an autonomous agent would do well at mathematically challenging problems with bounded search space and automatically testable and quantifiable output.

(Not GP but I assume that's what they were getting at)

> If AI can take all the crap code humans are writing and make it better, that sounds like a huge win.

This sort of misunderstanding of achievements is what keeps driving the AI mania. The AI generated an algorithm for optimizing a well-defined, bounded mathematical problem that marginally beat the human-written algorithms.

This AI can't do what you're hyping it up to do because software optimization is a different kind of optimization problem - it's complex, underspecified, and it doesn't have general algorithmic solutions.

LLM may play a significant role in optimizing software some day but it's not going to have much in common with optimization in a mathematical sense so this achievement doesn't get us any closer to that goal.

> AHC058, held on December 14, 2025, was conducted over a 4-hour competition window. The problem involved a setting where participants could produce machines with hierarchical relationships, such as multiple types of “apple-producing machines” and “machines that build those machines.” The objective was to construct an efficient production planning algorithm by determining which types and hierarchies of machines to upgrade and in what specific order.

... so not a CRUD app but it beat humans at Cookie Clicker? :-)

So many people hyping AI are only thinking about new projects and don't even distinguish between what is a product and what is a service.

Most software devs employed today work on maintaining services that have a ton of deliberate decisions baked in that were decided outside of that codebase and driven by business needs.

They are not building shiny new products. That's why most of the positive hype about AI doesn't make sense when you're actually at work and not just playing around with personal projects or startup POCs.

Personally I've yet to see any high profile programming person (who's not directly invested into AI) endorse only coding by prompting.

Experienced coders that I follow, who do use AI tend to focus on tight and fast feedback loops, and precise edits (or maybe exploratory coding) rather than agentic fire-and-forget workflows.

Also, an interesting side note, I expected programmers I think of as highly skilled, who I know personally to reject AI from personal pride - that has not been the case. However 2 criticisms I've heard consistently from this crowd (besides the thing I mentioned before) was

- AI makes hosting and participating in coding competitions impossible, and denies them of brain-teasers and an ability to hone their skills.

- A lot of them are concerned about the ethics of training on large codebases - and consider AI plagiarism as much of an issue as artists do.

Like, yes, prompting is a skill and you need to learn it for AI to do something useful but usefulness quickly falls down a cliff once you go past "greenfield implementation" or "basically example code" or "the thing done a lot so AI have a lot of reference to put from" it quickly gets into kinda sorta but not really working state.

It can still be used effectively on smaller parts of the codebase (I used it a lot basically to generate some boilerplate to run the test even if I had to rewrite a bunch of actual tests) but as whole very, very overrated by the AI peddlers.

And it probably stems from the fact that for the clueless ones it looks like amazing productivity boost because they go from "not even knowing framework" to "somewhat working app"

People already say here that they don’t even look the code anymore. ”That is AIs job”. As long as there is a spec and tests pass, they are happy! I just can’t do that.

It's just the next rung on the enshittification ladder. So many steps in our "progress" to enlightenment as a society, as a technology community, is just abstracting away work with a "good enough" solution that is around an 80% solution

That's fine for the first iteration or two, because you think "oh man this is going to make me so productive, I'll be able to use this new productivity to wring 40% of progress out of that 20% gap"

But instead we just move on to the next thing, bring that 20% shittified gap along with us, and the next thing that gets built or paved over has a 20% gap, and eventually we're bankrupt from rolling over all that negative equity

The counter argument for this is the comparison for traditional compilers. AI is "the new compiler", just for natural language. The optimization happens over time! But I am not so sure about that.

Except that the most glaring difference is that compilers are deterministic, while LLMs aren't.

Given the same input, compilers will always return the same output, while for LLMs. They won't, given the same input, they will return different output.

Because most people don't work on public projects and can't share the code publicly?

What's more interesting is the lack of examples of non-trivial projects that are provably vibe-coded and that claim to be of high-quality.

I think many of us are looking for: "I vibe-coded [this] with minimal corrections/manual coding on a livestream [here] and I believe it to be high-quality code"

If the code is in fact good quality then the livestream would serve as educational material for using LLMs/agents productively and I guarantee that it would change many minds. Stop telling people how great it all is, show them. I don't want to be a naysayer, I want to be impressed.

I'm considering attempting to vibe code translate one of my XNA games to javascript and recording the process and using all of the latest tools and strategies like agents and .md files and multiple LLMs etc

That's been pretty much exactly my experience too.

For what it's worth, multiple times in my career, I've worked at shops that once thought they could do it quick and cheap and it would be good enough, and then had to hire someone 'picky' like me to sort out the inevitable money-losing mess.

From what I've seen even Opus 4.5 spit, the 'picky' are going to remain in demand for a little while longer still. Will that last? No clue. We'll see.

You can be picky with Opus, just yell at it to refactor a few times. To reduce refactor cycles, give it correct and enough context before you start along with expected code style, etc. These things aren't one shot magic machines.

Doesn't match my experience, that figure is closer to about 20-40% to me, though a lot of those changes I want are possible by just further prompting OR turning to a different model, or adding some automated checks that promptly fail and the AI can do a few more loops of fixes.

> Other people are just less picky than I am, or they have a less thorough review culture that lets subpar code slide more often.

This is also likely, or you are just doing stuff that is worse represented in the training data, or working on novel things where the output isn't as good. But I'm leaning towards people just being picky about what they view as "good code" (or underspecifying how the AI is supposed to output it) at least roughly since Sonnet 4, since with some people I work with it's just endless and oftentimes meaningless discussions and bikeshedding when in code review.

You can always be like: "This here pattern in these 20 files is Good Code™, use the same collection of approaches and code style when working on this refactoring/new feature."

> You can always be like: "This here pattern in these 20 files is Good Code™, use the same collection of approaches and code style when working on this refactoring/new feature."

…and then add that to your CLAUDE.md, and never worry about having to say it again manually.

Exactly! Unless you use something that doesn’t read CLAUDE.md, then you’d still just tell the model to read the file as a part of its work.

What helped me a bunch was having prebuild scripts (can be Bash, can be Python, can be whatever) for each of the architectural or style conventions I want to enforce. Tools like ESLint are also nice but focused a bit more on the code than architecture/structure.

Problems start when a colleague might just remove some of those due to personal preference without discussion but then you have other problems - in my experience, with proper controls in place AI will cause less issues and friction than people (ofc depending on culture fit).

I think I know what they mean, I share a similar experience. It has changed, 3.5 couldn't even attempt to solve non-trivial tasks so it was a 100% failure, now it's 70%.

I get the best results when using code to demonstrate my intention to an LLM, rather than try and explain it. It doesn't have to be working code.

I think that mentally estimating the problem space helps. These things are probabilistic models, and if there are a million solutions the chance of getting the right one is clearly unlikely.

On the subpar code, would the code work, albeit suboptimally?

I think part of the problem a lot of senior devs are having is that they see what they do as an artisanal craft. The rest of the world just sees the code as a means to an end.

I don't care how elegantly my toaster was crafted as long as it toasts the bread and doesn't break.

There is some truth to your point but you might want to consider that often seniors concerned with code quality aren't being pedantic about artisanal craft they are worried about the consequences of bad code...

- it becomes brittle and rigid (can't change it, can't add to it)

- it becomes buggy and impossible to fix one bug without creating another

- it becomes harder to tell what it's doing

- plus it can be inefficient / slow / insecure, etc.

The problem with your analogy is that toasters are quite simple. The better example would be your computer, and if you want your computer to just run your programs and not break, then these things matter.

Perhaps a better analogy is the smartphone or personal computer.

Think of all the awful cheapest android phones and Windows PCs and laptops that are slow, buggy, have not had a security update in however long and are thus insecure, become virtually unusable within a couple years. The majority of the people in the world live on such devices either because they don't know better or have no better option. The world continues to turn.

People are fine with imperfection in their products, we're all used to it in various aspects of our lives.

Code being buggy, brittle, hard to extend, inefficient, slow, insecure. None of those are actual deal breakers to the end user, or the owners of the companies, and that's all that really matters at the end of the day in determining whether or not the product will sell and continue to exist.

If we think of it in terms of evolution, the selection pressure of all the things you listed is actually very weak in determining whether or not the thing survives and proliferates.

The usefulness is a function of how quickly the consequences from poor coding arrive and how meaningful they are to the organization.

Like in finance if your AI trading bot makes a drastic mistake it's immediately realized and can be hugely consequential, so AI is less useful. Retail is somewhat in the middle, but for something like marketing or where the largest function is something with data or managerial the negatives aren't as quickly realized so there can be a lot of hype around AI and what it may be able to do.

Another poster commented how very useful AI was to the insurance industry, which makes total sense, because even then if something is terribly wrong it has only a minor chance of ever being an issue and it's very unlikely that it would have a consequence soon.

> The usefulness is a function of how quickly the consequences from poor coding arrive and how meaningful they are to the organization.

I would just add that these hypothetical senior devs we are talking about are real people with careers, accountability and responsibilities. So when their company says "we want the software to do X" those engineers may be responsible for making it happen and accountable if it takes too long or goes wrong.

So rather than thinking of them as being irrationally fixated on the artisanal aspect (which can happen) maybe consider in most cases they are just doing their best to take responsibility for what they think the company wants now and in the future.

There’s for sure legitimacy to the concern over the quality of output of LLMs and the maintainability of that code, not to mention the long term impact on next generation of devs coming in and losing their grasp on the fundamentals.

At the same time, the direction of software by and large seems to me to be going in the direction of fast fashion. Fast, cheap, replaceable, questionable quality.

Not all software can tolerate this, as I mentioned in another comment, flight control software, the software controlling your nuclear power plant, but the majority of the software in the world is far more trivial and its consumers (and producers) more tolerant of flaws.

I don’t think of seniors as purely irrationally fixated on the artisanal aspect, I also think they are rationally, subconsciously or not, fearful of the implications for their career as the bottom falls out of this industry.

I could be wrong though! Maybe high quality software will continue to be what the industry strives for and high paying jobs to fix the flawed vibe coded slop will proliferate, but I’m more pessimistic than to think that.

> I don't care how elegantly my toaster was crafted as long as it toasts the bread and doesn't break.

A consumer or junior engineer cares whether the toaster toasts the bread and doesn’t break.

Someone who cares about their craft also cares about:

- If I turn the toaster on and leave, can it burn my house down, or just set off the smoke alarm?

- Can it toast more than sliced uniform-thickness bread?

- What if I stick a fork in the toaster? What happens if I drop it in the bathtub while on? Have I made the risks of doing that clear in such a way that my company cannot be sued into oblivion when someone inevitably electrocutes themselves?

- Does it work sideways?

- When it fills up with crumbs after a few months of use, is it obvious (without knowing that this needs to be done or reading the manual) that this should be addressed, and how?

- When should the toaster be replaced? After a certain amount of time? When a certain misbehavior starts happening?

Those aren’t contrived questions in service to a tortured metaphor. They’re things that I would expect every company selling toasters to have dedicated extensive expertise to answering.

All those things you’re talking about may or may not matter some day, after years and a class action lawsuit that may or may not materialize or have any material impact on the bottom line of the company producing the toaster, by which time millions of units of subpar toasters that don’t work sideways will have sold.

The world is filled with junk. The majority of what fills the world is junk. There are parts of our society where junk isn’t well tolerated (jet engines, mri machines) but the majority of the world tolerates quite a lot of sloppiness in design and execution and the companies producing those products are happily profitable.

You really underestimate how much work goes into everything around you. You don't care because it just works: the stuff you use is by and large not crap, which makes the crappy stuff all the more noticable. Check out the housing code for your area: everything from the size of steps to the materials used for siding are in there. Or look at the FCC specifications for electrical devices that make sure you don't inadvertently jam radio frequencies in your local area, or the various codes which try very hard to stop you from burning your house down.

You're right that "there are parts of our society where junk isn't well tolerated", but the scope of those areas is far greater than you give credit for.

I'm long term traveling, mostly through the developing world, where something like 84% of humanity resides.

All around me, people's houses, the roads, the infrastructure, food cultivation and preparation, furniture, vehicles, it goes on and on, the tendency is towards loose approximation, loose standards. Things are constantly breaking, the quality is low, people are constantly being poisoned by the waste seeping into their water, air and soil, by the plastic they burn to cook their food, by the questionable chemicals in the completely unsafe industrial environments they work in to produce toxic products consumed by the masses.

There is no uniform size of steps. Yet the majority of humanity lives this way, and not just tolerates it but considers it a higher standard of living than we've had for the majority of human history.

I don't think people in the first world are a different species, so we will also adapt to whatever shitty environment we regress into as our standards fall. We'll realize that the majority of the areas we may consider sacrosanct are in fact quite negotiable in terms of quality when it comes down to our needs.

All this is to say that yeah, I think people will generally tolerate the quality of software going down just fine.

That's a sad way to think. I'd like to hope that humanity can improve itself, and that includes building products that are safer, more refined, more beautiful, more performant and more useful. I agree that there's a lot of crap out there, but I still want to believe and strive to make things that are excellent. I'm not ready to give up on that. And yes, I still get annoyed every time my crappy toaster doesn't work properly.

>I think part of the problem a lot of senior devs are having is that they see what they do as an artisanal craft. The rest of the world just sees the code as a means to an end.

Then you haven't been a senior dev long enough.

We want code that will be good enough because we will have to maintain it for years (or inherit maintaining from someone else), we want it to be clean enough that adding new features isn't a pain and architected well enough that it doesn't need major rewrite to do so.

Of course if code is throwaway that doesn't matter but if you're making long term product, making shit code now is taking on the debt you will have to pay off.

That is not to say "don't use AI for that", that is to say "actually go thru AI code and review whether it is done well enough". But many AI-first developers just ship first thing that compiles or passes tests, without looking.

> I don't care how elegantly my toaster was crafted as long as it toasts the bread and doesn't break.

...well if you want it to not break (and still be cheap) you have to put quite a bit of engineering into it.

Have you tried asking one of your peers who claims to get good results to run a test with you? Where you both try to create the same project, and share your results?

I and one or two others are _the_ AI use experts at my org, and I was by far the earliest adopter here. So I don't really have anyone else with significantly different experiences than me that I could ask.

I think this touches on the root of the issue. I am seeing a results over process winning. Code quality will reduce. Out of touch or apathetic project management who prioritize results, now are even more emboldened to have more tech debt riddled code

This is exact the impression that I got. Every question or task given to LLM returns pretty reasonable, but flawed result. For the coding, those are hard to spot but dangerous mistakes. They all look good and perfectly reasonable, but just wrong. Anthropic compared Claude Code to a "slot machine", and I fell that AI coding now is something close to gambling addiction. As small wins keep gambler to make more bets, so correct results from AI keep developers to use it: "I see it made correct solution, let's try again!"
At a startup CTO, I review most of the pull requests from team members, and team uses AI tools actively. The overall picture strongly confirms your second conclusion.

If someone gives you access to a slot machine which is weighted such that it pays out way more than you put into it, my advice is to start cranking that lever.

If it does indeed start costing more than it's paying out, step away.

To anybody who want to try, a concrete example, that I have tested in all available LLMs:

Make a prompt to get a common lisp application which makes a "hello triangle" in open gl, without using SDL or any framework, only OpenGL and GLFW bindings.

None of the replies even compiled. I kept asking at least 5 times, with error feedback, to see if AI can do it. It did't work. Never.

The best I got was from gemini, a code where I had to change about 10 lines, absolutely no trivial changes that need to be familiar with opengl and lisp. After doing the changes I asked back, what does it think of the changes, it replied I was wrong, with those changes it will never work.

If anybody can make a prompt that get me that, please let me known...

It sounds like you're using LLMs directly, instead of a coding agent. Agents are capable of testing their own code and using that to fix issues, which is what makes them so powerful.

Nice. The code I got running from gemini was much much cleaner, it did not work, but after the manual changes it did work. I will hive it a try with the next task: put text and generate primitives like rectangle, circle, polygon, etc…

"Please write me a program in Common LISP (SBCL is installed) which will render a simple "hello world" triangle in OpenGL. You should use only OpenGL and GLFW (using sbcl's FFI) for this, not any other existing 3D graphics framework."

That is using sb-alien and sb-sys, which basically is no common lisp anymore. That is basically sbcl. I didn’t get anything in that direction (my prompt said nothing about a CL implementation) but I would have rejected it. I just wanted to see glfw and opengl in the :use clause. I have to do something that has to work in Mac, Linux and Windows, with at least ECL, sbcl and ccl.

Yeah, I was just trying to keep to the letter of what you'd said - you asked for it just to use OpenGL/GLFW bindings, not other libraries, so I didn't want to install cl-opengl and cl-glfw, and told it just to use its own FFI.

Well at least ypu and the other commenters made something that worked, which I was unable to do. Seems the key is using a coding agent, not an LLM out of the box like I did.

Maybe if your coding style is already close to what an LLM like Claude outputs, you’ll never have these issues? At least it generally seems to be doing what I would do myself.

Most of the architectural failures come from it still not having the whole codebase in mind when changing stuff.

I actually think it's less about code style and more about the disjointed way end outcomes seem to be the culmination of a lot of prompt attempts over the course of a project/implementation.

The funny thing is reviewing stuff claude has made isn't actually unfamiliar to me in the slightest. It's something I'm intimately familiar with and have been intimately familiar with for many years, long before this AI stuff blew up...

..it's what code I've reviewed/maintained/rejected looks like when a consulting company was brought on board to build something. Such a company that leverages probably underpaid and overworked laborers both overseas and US based workers on visas. The delivered documentation/code is noisy+disjointed.

> The delivered documentation/code is noisy+disjointed.

Yeah, which is what you get if your memory consists of everything you’ve read in the past 20 minutes. Most of my Claude work involves pointing it at the right things.

> Every single time, I get something that works, yes, but then when I start self-reviewing the code, preparing to submit it to coworkers, I end up rewriting about 70% of the thing.

You might want to review how you approach these tools. Complaining that you need to rewrite 70% of the code screams of poor prompting, with too vague inputs, no constraints, and no feedback at all.

Using agents to help you write code is far from a one-shot task, but if throwing out 70% of what you create screams out that you are prompting the agent to create crap.

> 1) I'm not good at prompting, even though I am one of the earliest AI in coding adopters I know, and have been consistent for years. So I find this hard to accept.

I think you need to take a humble pill, review how you are putting together these prompts, figure out what you are doing wrong in prompts and processes, and work up from where you are at this point. If 70% of your output is crap, the problem is in your input.

I recommend you spend 20 minutes with your agent of choice prompting it to help you improve your prompts. Check instruction files, spec-driven approaches, context files, etc. Even a plain old README.md helps a lot. Prompt your agent to generate it for you. From there, instead of one-shot prompts try to break down a task into multiple sub steps with small deliverables. Always iterate on your instruction files. It you spend a few minutes on this, you will quickly halve your churn rate.

Maybe LLMs are like a next evolution of a rubber ducky: you can talk to it, and it's very helpful, just don't expect that IT will give you the final answer.

I have been doing the same since GPT-3. I remember a time, probably around 4o when it started to get useful for some things like small React projects but was useless for other things like firestore rules. I think that surface is still jagged, it's just that it's less obviously useless in areas that it's weaker.

When things really broke open for me was when I adopted windsurf with Opus 4, and then again with Opus 4.5. I think the way the IDE manages the context and breaks down tasks helps extend llm usefulness a lot, but I haven't tried cursor and haven't really tried to get good at Claude code.

All that said, I have a lot of experience writing in business contexts and I think when I really try I am a pretty good communicator. I find when I am sloppy with prompts I leave a lot more to chance and more often I don't get what I want, but when I'm clear and precise I get what I want. E.g. if it's using sloppy patterns and making bad architectural choices, I've found that I can avoid that by explaining more about what I want and why I want it, or just being explicit about those decisions.

Also, I'm working on smaller projects with less legacy code.

So in summary, it might be a combination of 1, 2 and the age/complexity of the project you're working on.

In my experience, using AI coding agents need highly specific success criteria, and an easy way to verify its output against that criteria.

My biggest successes have come when I take a TDD approach. First I identify a subset of my work into a module with an API that can be easily tested, then I collaborate with the agent on writing correct test-cases, and finally I tell it to implement the module such that the test cases pass without any lint or typing errors.

It forces me to spend much more time thinking about use cases, project architecture, and test coverage than about nitty-gritty implementation details. I can imagine that in a system that evolved over time without a clear testing strategy, AI would struggle mightily to be even barely useful.

Not saying this applies to your system, but I've definitely worked on systems in the past that fit the "big ball of mud" description pretty neatly, and I have zero clue how I'd have been able to make effective use of these AI tools.

4) It's easy to get too excited about the tech and ignore its failure modes when describing your experiences later

I use AI a lot. With your own control plane (as opposed to a generic Claude Code or whatever) you can fully automate a lot more things. It's still fundamentally incapable of doing tons of tasks though at any acceptable quality level, and I strongly suspect all of (2,3,4) are guiding the disconnect you're seeing.

Take the two things I've been working on this morning as an example.

One was a one-off query. I told it the databases it should consider, a few relevant files, roughly how that part of the business works, and asked it to come back when it finished. When it was done I had it patch up the output format. It two-shot (with a lot of helpful context) something that would have taken me an hour or more.

Another is more R&D-heavy. It pointed me to a new subroutine I needed (it couldn't implement it correctly though) and is otherwise largely useless. It's actively harmful to have it try to do any of the work.

It's possible that (1) matters more than you suspect too. AI has certain coding patterns it likes to use a lot which won't work in my codebase. Moreover, it can't one-shot the things I want. It can, however, follow a generic step-by-step guide for generating those better ideas, translating worse ideas into things that will be close enough to what I need, identifying where it messed up, and refactoring into something suitable, especially if you take care to keep context usage low and whatnot. A lot of people seem to be able to get away with CLAUDE.md or whatever, but I like having more granular control of what the thing is going to be doing.

It might be 1), being an early adopter doesn’t help much with AI. So much is changing constantly. If you put a good description of your architecture and coding guidelines in the right .md files and work on your prompts the output should be much better. In the other hand your project being legacy code probably also doesn’t help.

We find across our team different people are able to use these things at different levels. Unsurprisingly, more senior coders with both more experience in general and more experience in ai coding are able to do more with ai and get more ambitious things done more quickly.

A bummer is that we have a genai team (louie.ai) and a gpu/viz/graph analytics team (graphistry), and those who have spent the last 2-3 years doing genai daily have a higher uptake rate here than those who aren't. I wouldn't say team 1 is better than team 2 in general: these are tools, and different people have different engineering skill and ai coding skill, including different amounts of time doing both.

What was a revelation for me personally was taking 1-2mo early in claude code's release was to go full cold turkey on manual coding, similar to getting immersed in a foreign language. That forced eliminating a lot of bad habits wrt effective ai coding both personally and in state of our repo tooling. Since then, it's been steady work to accelerate and smooth that loop, eg, moving from vibe coding/engineering to now more eval-driven ai coding loops: https://media.ccc.de/v/39c3-breaking-bots-cheating-at-blue-t... . That takes a LOT of buildout.

I don't have any links but you can start with CLAUDE.md and/or AGENTS.md and put the basic instructions in there ( you can also google these filenames for examples and recommendations). I also put README.md's in every subfolder to describe which file does what, etc.

My experience with agents in larger / older codebases is that feedback loops are critical. They'll get it somewhere in the neighborhood of right on the first attempt; it's up to your prompt and tooling to guide them to improve it on correctness and quality. Basic checks: can the agent run the app, interact with it, and observe its state? If not, you probably won't get working code. Quality checks: by default, you'll get the same code quality as the code the agent reads while it's working; if your linters and prompts don't guide it towards your desired style, you won't get it.

To put that another way: one-shots attempts aren't where the win is in big codebases. Repeat iteration is, as long as your tooling steers it in the right direction.

I assume this is part of the problem (though I've avoided using LLMs mostly so can't comment with any true confidence here) but to a large extent this is blaming you for a suboptimal interface when the interface is the problem.

That some people seem to get much better results than others, and that the distinction does not map well to differences in ability elsewhere, suggests to me that the issue is people thinking slightly differently and the training data for the models somehow being biased to those who operate in certain ways.

> 2) Other people are just less picky than I am

That is almost certainly a much larger part of the problem. “Fuck it, it'll do, someone else can tidy it later if they are bothered enough” attitudes were rampant long before people started outsourcing work to LLMs.

I think you should try harder to find their limits. Be as picky as you want, but don't just take over after it gave you something you didn't like. Try again with a prompt that talks about the parts you think were bad the first time. I don't mean iterate with it, I mean start over with a brand new prompt. Try to figure out if there is a prompt that would have given you the result you wanted from the start.

It won't be worth it the first few times you try this, and you may not get it to where you want it. I think you might be pickier than others and you might be giving it harder problems, but I also bet you could get better results out of the box after you do this with a few problems.

Not even coding tasks. Just getting an LLM to help me put together a PromQL query to do something somewhat non-standard takes dozens of tries and copy/pasting back error messages.. and these aren't complex errors, trivial things like missing closing brackets and the like.

I know the usual clap back is "you're just missing this magical workflow" or "you need to prompt better" but.. do I really need to prompt "make sure your syntax is correct"? Shouldn't that be, ya know, a given for a prompt that starts with "Help me put together a PromQL query that..."?

If you find yourself having to copy and paste errors back and forward you need to upgrade to a coding agent harness like Claude Code so the LLM can try things out and then fix the errors on its own.

If you're not willing to do that you can also fix this by preparing a text file with a few examples of correctly formatted queries and pasting that in at the start of your session, or putting it in a skill markdown file.

With very good tooling (e.g., Google Antigravity, Claude Coding, Open AI’s codex, and several open platforms) and not caring about your monthly API and subscription costs, then very long running trial and error and also with tools for testing code changes, then some degree of real autonomy is possible.

But, do we want to work like this? I don’t.

I feel very good about using strong AI for research and learning new things (self improvement) and I also feel good about using strong AI as a ‘minor partner’ in coding.

Try learning to vibe code on something totally greenfield without looking at the code and see if it changes your mind. Ignore code quality, “does it work” and “am i happy with the app” are the only metrics.

Code quality is an issue you need to ignore with vibe coding - if code quality is important to your project or you then it’s not an issue. But if you abandon this concept and build things small enough or modular enough then speed gains await!

IMO codebases can be architected for LLMs to work better in them, but this is harder in brownfield apps.

If you start greenfield and ignore the code quality, how do you know you can maintain it long term?

Greenfield is fundamentally easier than maintaining existing software. Once software exists, users expect it to behave a certain way and they expect their data to remain usable in new versions.

The existing software now imposes all sorts of contraints that may not be explicit in the spec. Some of these constraints end up making some changes very hard. Bad assumptions in data modeling can make migrations a nightmare.

You can't just write entirely new software every time the requirements change.

1) Making the application small enough, and breaking it apart if needed (e.g. I've refactored my old 'big' app into 10 micro-apps).

2) Selecting an architecture that will work, looking after the data modelling and architecture yourself rather than delegating this to the LLM (it can implement it - but you need to design it).

3) Trusting that the LLM is capable enough to implement new requirements or fixes as required.

If requirements change so substantially that it's not possible, you can write new software as requirements change - as per point 1, you will have made your application modular enough that this isn't a significant concern.

> 2) Other people are just less picky than I am, or they have a less thorough review culture that lets subpar code slide more often.

Given how consistently terrible the code of Claude Code-d projects posted here have been, I think this is it.

I find LLMs pretty useful for coding, for multiple things(to write boilerplate, as an idiomatic design pattern search engine, as a rubber duck, helping me name things, explaining unclear error messages, etc.), but I find the grandiose claims a bit ridiculous.

It's not just about better prompting, but using better tools. Tools that will turn a bad prompt into a good prompt.

For example there is the plan mode for Cursor. Or just ask the AI: "make a plan to do this task", then you review the plan before asking it to implement. Configure the AI to ask you clarification questions instead of assuming things.

It's still evolving pretty quickly, so it's worth staying up to date with that.

I have not been as aggressive as GP in trying new AI tools. But the last few months I have been trying more and more and I'm just not seeing it.

One project I tried out recently I took a test-driven approach. I built out the test suite while asking the AI to do the actual implementation. This was one of my more successful attempts, and may have saved me 20-30% time overall - but I still had to throw out 80% of what it built because the agent just refused to implement the architecture I was describing.

It's at its most useful if I'm trying to bootstrap something new on a stack I barely know, OR if I decide I just don't care about the quality of the output.

I have tried different CLI tools, IDE tools. Overall I've had the best success with Claude Code but I'm open to trying new things.

Do you have any good resources you would recommend for getting LLM's to perform better, or staying up-to-date on the field in general?

I think you are not hardcore enough. I paste entire files or 2 3 files at once and ask to rewrite everything.

Then you rewiew it and in general have to ask to remove some stuff. And then it's (good enough). You have to accept to not nitpick some parts (like random functions being generated) as long as your test suite pass, otherwise of course you will end up rewritin everything

It also depends on your setting, some area (web vs AI vs robotics) can be more suited than other

It is pretty simple imo. AI (just like humans!) does best on well written, self contained code bases. Which is a very small niche, but also over represented in open source and subsequently by tech celebrities who tend not to work on “ugly code”.

I work on a giant legacy
code base at big tech, which is one piece of many distributed systems. LLM is helpful for localised, well defined work, but nowhere close to what the TFA describes.

If you follow antirez's post history, he was a skeptics until maybe a year ago. Why don't you look at his recent commits and judge for yourself. I suppose the majority of his most recent code is relevant for this discussion.

I don't think I'd be a good judge because I don't have the years of familiarity and expertise in his repos that I do at my job. A lot of the value of me specifically vs an LLM at my job is that I have the tribal knowledge and the LLM does not. We have gotten a lot better at documentation, but I don't think we can _ever_ truly eliminate that factor.

Not trying to back the AI hype, but most pre-AI auto generated code is garbage (like winform auto generated code or entity framework SQL in the .net world). But that’s fine, it’s not meant to be read by humans. If you want to change it you can regenerate it. It may be that AI just moves the line between what developers should care and look at vs the boring boiler plate code that has little value added.

But those code generators were deterministic (and indeed caused huge headaches if the generated code changed between versions). Seems like a totally different thing.

Instead of rewriting yourself have you tried telling the agent what it did wrong and do the rewrite with it? Then at the end of the session ask it to extract a set of rules that would have helped to get it right the first time. Save that in AGENTS.md. If you and your team do this a few times it can lead to only having to rewrite 5% of the code instead of 70%.

It says "of course you're right" and may or may not refactor/fix/rewrite the issue correctly. More often than not it doesn't or misses some detail.

So you tell it again, "of course you are right", and the cycle repeats.

And then the context window gets exhausted. Compaction loses most of the details and degrades quality. You start a new session, but the new session has to re-learn the entire world from scratch and may or may not fix the issue.

How much buggy / incorrect Java written by first year computer science University students is there on Stack Overflow (in SO post bodies)? Decades of it.

Ask the same question of Golang, or Rust, or Typescript.

I have a theory that the large dichotomy in how people experience AI coding has to do with the quality of the training corpus for each language online.

I'm not sure if I got in this weird LLM bubble where they give me bad advice to drive engagement, because I can't resist trying to correct them and tell them how absurdly wrong they are.

But it is astounding how terrible they are at debugging non-trivial assembly in my experience.

Anyone else have input here?

Am I in a weird bubble? Or is this just not their forte?

It's truly incredible how thoughtless they can be, so I think I'm in a bubble.

I've tried to use Claude Code with Sonnet 4.5 for implementing a new interpreter, and man is it bad with reference counting. Granted, I'm doing it in Zig, so there's not as much training, but Claude will suggest the most stupid changes. All it does is make the rare case of incorrect reference counting more rare, not fixing the underlying problem. It kept heaping on more and more hacks, until I decided enough is enough and rolled up my sleeves. I still can't tell if it makes me faster, or if I'm faster.

Even when refactoring, it would change all my comments, which is really annoying, as I put a lot of thought into my comments. Plus, the time it took to do each refactoring step was about how long it would take me, and when I do it I get the additional benefit of feeling when I'm repeating code too often.

So, I'm not using it for now, except for isolating bugs. It's addicting having it work on it for me, but I end up feeling disconnected and then something inevitably goes wrong.

Oh cool! I'd love to hear more. I'm implementing an existing language, Tcl, but I'm working on making it safe to share values between threads, since a project I contribute to[1] uses Tcl for all the scripting, but they have about a 30% overhead with serialization/deserialization between threads, and it doesn't allow for sharing large values without significant overheads. I'm also doing some experiments with heap representation to reduce data indirection, so it's been fun getting to learn how to implement malloc and other low-level primitives I usually take for granted.

Thank you for providing data which can actually be used to collate! I strongly suspect that experience is a huge determinant of what utility is seen from LLMs.

It seems that theres more people writing and finishing projets, but not many have reached the point where they have to maintain their code / deal with the tech debt.

Genuine question, doesn't this apply to coding style than actual results? Same applies to writing style. LLMs manage to write great stories but they don't suit my writing style. When generating code it doesn't always suit my coding style but the code it generates functions fine.

> Every single time, I get something that works, yes, but then when I start self-reviewing the code, preparing to submit it to coworkers, I end up rewriting about 70% of the thing.

Have another model review the code, and use that review as automatic feedback?

CodeRabbit in particular is gold here. I don't know what they do but it is far better at reviewing than any AI model I've seen. From the deep kinds of things it finds, I highly suspect they have a lot of agents routing code to extremely specialized subagents that can find subtle concurrency bugs, misuse of some deep APIs etc. I often have to do the architecture l/bug picture/how this fits into project vision review myself, but for finding actual bugs in code, or things that would be self evident from reading one file, it is extremely good.

I've been using a `/feedback ...` command with claude code where I give it either positive or negative feedback about some action it just did, and it'll look through the session to make some educated guesses about why it did some thing - notably, checking for "there was guidance for this, but I didn't follow it", or "there was no guidance for this".

the outcome is usually a new or tweaked skill file.

it doesn't always fix the problem, but it's definitely been making some great improvements.

Codex is a sufficiently good reviewer I now let it review my hand-coded work too. It's a really, really good reviewer. I think I make this point often enough now that I suspect OpenAI should be paying me. Claude and Gemini will happily sign off work that just doesn't work, OpenAI is a beast at code-review.

It sounds harsh but you're most likely using it wrong.

1) Have an AGENTS.md that describes not just the project structure, but also the product and business (what does it do, who is it for, etc). People expect LLMs to read a snippet of code and be as good as an employee who has implicit understanding of the whole business. You must give it all that information. Tell it to use good practices (DRY, KISS, etc). Add patterns it should use or avoid as you go.

2) It must have source access to anything it interacts with. Use Monorepo, Workspaces, etc.

3) Most important of all, everything must be setup so the agent can iterate, test and validate it's changes. It will make mistakes all the time, just like a human does (even basic syntax errors), but it will iterate and end up on a good solution. It's incorrect to assume it will make perfect code blindly without building, linting, testing, and iterating on it. No human would either. The LLM should be able to determine if a task was completed successfully or not.

4) It is not expected to always one shot perfect code. If you value quality, you will glance at it, and sometimes ahve to reply to make it this other way, extract this, refactor that. Having said that, you shouldn't need to write a single line of code (I haven't for months).

Using LLMs correctly allow you to complete tasks in minutes that would take hours, days, or even weeks, with higher quality and less errors.

Use Opus 4.5 with other LLMs as a fallback when Opus is being dumb.

> Most important of all, everything must be setup so the agent can iterate, test and validate it's changes.

This was the biggest unlock for me. When I received a bug report I have the LLM tell me where it thinks the source of the bug is located, write a test that triggers the bug/fails, design a fix, finally implement the fix and repeat. I'm routinely surprised how good it is at doing this, and the speed with which it works. So even if I have to manually tweak a few things, I've moved much faster than without the LLM.

Situation A) Model writes a new endpoint and that's it

Situation B) Model writes a new endpoint, runs lint and build, adds e2e tests with sample data and runs them.

Did situation B mathematically prove the code is correct? No. But the odds the code is correct increases enormously. You see all the time how the Agent finds errors at any of those steps and fixes them, that otherwise would have slipped by.

AI-generated implementation with AI-generated tests left me with some of the worst code I've witnessed in my life. Many of the passing tests it generated were tautologies (i.e. they would never fail even if behavior was incorrect).

When the tests failed the agent tended to change the (previously correct) test making it pass but functionally incorrect, or it "wisely" concluded that both the implementation and the test are correct but that there are external factors making the test fail (there weren't).

That's the curse of the expert. You see many of the shortcomings, that someone less experienced might not even think about, when they go to social media and blurt out that AI is now able to fully replace them.

Like with anything else the people best positioned to enjoy output are the people least well positioned to criticize it. This is true of AI just as eating at restaurants or enjoying movie dramas.

This is also my experience with enterprise Java. LLMs have done much better with slightly less convoluted code bases in Go. Its currently clearly better at Go and Typescript than Java in my view

Wow only 70%. I so far have had to drop and rewrite from scratch every time. Mind, I work in C/embedded spaces, and current LLMs are just horrible at any code in that space.

AI is a house painter, wall to wall, with missed spots and drips. Good coders are artists. That said, artists have been known to use assistants on backgrounds. Perhaps the end case is a similar coder/AI collaborative effort?

LLMs tend to rise to the level of the complexity of the codebase. They are probabilistic pattern matching machines, after all. It's rare to have a 15 year old repo without significant complexity; is it possible that the reason LLMs have trouble with complex codebases is that the codebases are complex?

IMO it has nothing to do with LLMs. They just mirror the patterns they see - don't get upset when you don't like your own reflection! Software complexity is still bad. LLMs just shove it back in our face.

Implications: AI is always going to feel more effective on brand new codebases without any legacy weight. And less effective on "real" apps where the details matter.

The bias is strongly evident - you rarely hear anyone talking about how they vibe coded a coherent changeset to an existing repo.

> I don't understand the stance that AI currently is able to automate away non-trivial coding tasks

It's just the Dunning-Kruger effect. People who think AI is the bee's knees are precisely the dudes who are least qualified to judge its effectiveness.

I think it comes down to what you mean by sub par code. If you're talking a mess of bubblesorts and other algorithmic problems, that's probably a prompting issue. If you're talking "I just don't like the style of the code, it looks inelegant" that's not really a prompting issue, models will veer towards common patterns in a way that's hard to avoid with prompts.

Think about it like compiler output. Literally nobody cares if that is well formatted. They just care that they can get fairly performant code without having to write assembly. People still dip to assembly (very very infrequently now) for really fine performance optimizations, but people used to write large programs in it (miserably).

There's a huge amount you're missing by boiling down their complaint to "bubble sorts or inelegant code". The architecture of the new code, how it fits into the existing system, whether it makes use of existing utility code (IMO this is a huge downside; LLMs seem to love to rewrite a little helper function 100x over), etc.

These are all important when you consider the long-term viability of a change. If you're working in a greenfield project where requirements are constantly changing and you plan on throwing this away in 3 months, maybe it works out fine. But not everyone is doing that, and I'd estimate most professional SWEs are not doing that, even!

There's certainly coupled, obtuse, contorted code styles that the LLM will be unable to twister itself into (which is different from the coupled, obtuse code it generates itself). Don't pretend this is good code though, own that you're up to your neck in shit.

LLMs are pretty good at modifying well factored code. If you have a functional modular monolith, getting agents to add new functions and compose them into higher order functionality works pretty darn well.

It's a combination of being bad at prompting and different expectations from the tool.
You expect it to be one shot, and then rewrite things that don't match up to what you want.

Instead I recommend that you use LLMs to fix the problems that they introduced as well, and over time you'll get better at figuring out the parts that the LLM will get confused by. My hunch is that you'll find your descriptions of what to implement were more vague than you thought, and as you iterate, you'll learn to be a lot more specific. Basically, you'll find that your taste was more subjective than you thought and you'll rid yourself of the expectation that the LLM magically understands your taste.

> But what was the fire inside you, when you coded till night to see your project working? It was building.

I feel like this is not the same for everyone. For some people, the "fire" is literally about "I control a computer", for others "I'm solving a problem for others", and yet for others "I made something that made others smile/cry/feel emotions" and so on.

I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer, and for them, I understand LLMs remove the fun part. For me, I initially got into programming because I wanted to ruin other people's websites, then I figured out I needed to know how to build websites first, then I found it more fun to create and share what I've done with others, and they tell me what they think of it. That's my "fire". But I've met so many people who doesn't care an iota about sharing what they built with others, it matters nothing to them.

I guess the conclusion is, not all programmers program for the same reason, for some of us, LLMs helps a lot, and makes things even more fun. For others, LLMs remove the core part of what makes programming fun for them. Hence we get this constant back and forth of "Can't believe others can work like this!" vs "I can't believe others aren't working like this!", but both sides seems to completely miss the other side.

You’re right of course. For me there’s no flow state possible with LLM “coding”. That makes it feel miserable instead of joyous. Sitting around waiting while it spits out tokens that I then have to carefully look over and tweak feels like very hard work. Compared to entering flow and churning out those tokens myself, which feels effortless once I get going.

I'm the same way. LLMs are still somewhat useful as a way to start a greenfield project, or as a very hyper-custom google search to have it explain something to me exactly how I'd like it explained, or generate examples hyper-tuned for the problem at hand, but that's hardly as transformative or revolutionary as everyone is making Claude Code out to be. I loathe the tone these things take with me and hate how much extra bullshit I didn't ask for they always add to the output.

When I do have it one-shot a complete problem, I never copy paste from it. I type it all out myself. I didn't pay hundreds of dollars for a mechanical keyboard, tuned to make every keypress a joy, to push code around with a fucking mouse.

I’m a “LLM believer” in a sense, and not someone who derives joy from actually typing out the tokens in my code, but I also agree with you about the hype surrounding Claude Code and “agentic” systems in general. I have found the three positive use cases you mentioned to be transformative to my workflow on its own. I’m grateful that they exist even if they never get better than they are today.

Having worked with a greenfield project that has significant amount of LLM output in it, I’m not sure if I agree. There’s all sorts of weird patterns, insufficient permission checking, weird tests that don’t actually test things, etc. It’s like building a house on sand.

I’ve used Claude to create copies of my tests, except instead of testing X feature, it tests Y feature. That has worked reasonably well, except that it has still copied tests from somewhere else too. But the general vibe I get is that it’s better at copying shit than creating it from scratch.

That's why I never have it generate any significant amount of code for those. I get juuuuuust enough to start understanding the structure and procedure and which parts of the API docs I should even be looking at for the problem at hand, and start from there on my own. I need a lay of the land, not an entire architecture. I build that part.

This is where we as software engineers need to be on the ball - just because an LLM wrote it doesn't mean it's right, doesn't mean we can let go of all the checks and balances and best practices we've developed over decades.

Set up tooling like tests and linters and the like. Set rules. Mandate code reviews. I've been using LLMs to write tests and frequently catch it writing tests that don't actually have any valuable assertions. It only takes a minute to fix these.

> Set up tooling like tests and linters and the like. Set rules. Mandate code reviews. I've been using LLMs to write tests and frequently catch it writing tests that don't actually have any valuable assertions. It only takes a minute to fix these.

You can do all that, but it still remains a case of "I'm only interested in the final result".

When I read LLM generated systems (not single functions), it looks very ... alien to me. Even juniors don't put together systems that have this uncanny valley feel to it.

I suppose the best way to describe it would be to say that everything lacks coherency, and if you are one of these logical-mind people who likes things that make sense, it's not fun wading through a field of Chesterton's Fences as your f/time job.

I noticed this too. Everything lacks consistency, wrapped in headings that are designed to make me feel good. It’s uncomfortable reading one thing that seems so right followed by code that feels wrong but my usual instincts about why help less because of how half right it is.

(But still, LLMs have helped me investigate and write code that is beyond me)

> (But still, LLMs have helped me investigate and write code that is beyond me)

They haven't done that yet[1], but they have sped up things via rubber-ducking, and for documentation (OpenSSL is documentation is very complete, very thorough, but also completely opaque).

------------------------------------

[1] I have a project in the far future where they will help me do that, though. It all depends on whether I can get additional financial security so I can dedicate some time to a new project :-(

> and hate how much extra bullshit I didn't ask for they always add to the output.

I can recommend for that problem to make the "jumps" smaller, e.g. "Add a react component for the profile section, just put a placeholder for now" instead of "add a user profile".

With coding LLMs there's a bit of a hidden "zoom" functionality by doing that, which can help calibrating the speed/involvment/thinking you and the LLM does.

Three things I can suggest to try, having struggled with something similiar:

1. Look at it as a completely different discipline, dont consider it leverage for coding - it's it's own thing.

2. Try using it on something you just want to exist, not something you want to build or are interested in understanding.

3. Make the "jumps" smaller. Don't oneshot the project. Do the thinking yourself, and treat it as a junior programmer: "Let's now add react components for the profile section and mount them. Dont wire them up yet" instead of "Build the profile section". This also helps finding the right speed so that you can keep up with what's happening in the codebase

> Try using it on something you just want to exist, not something you want to build or are interested in understanding.

I don't get any enjoyment from "building something without understanding" — what would I learn from such a thing? How could I trust it to be secure or to not fall over when i enter a weird character? How can I trust something I do not understand or have not read the foundations of? Furthermore, why would I consider myself to have built it?

When I enter a building, I know that an engineer with a degree, or even a team of them, have meticulously built this building taking into account the material stresses of the ground, the fault lines, the stresses of the materials of construction, the wear amounts, etc.

When I make a program, I do the same thing. Either I make something for understanding, OR I make something robust to be used. I want to trust the software I'm using to not contain weird bugs that are difficult to find, as best as I can ensure that. I want to ensure that the code is clean, because code is communication, and communication is an art form — so my code should be clean, readable, and communicative about the concepts that I use to build the thing. LLMs do not assure me of any of this, and the actively hamstring the communication aspect.

Finally, as someone surrounded by artists, who has made art herself, the "doing of it" has been drilled into me as the "making". I don't get the enjoyment of making something, because I wouldn't have made it! You can commission a painting from an artist, but it is hubris to point at a painting you bought or commissioned and go "I made that". But somehow it is acceptable to do this for LLMs. That is a baffling mindset to me!

>I don't get any enjoyment from "building something without understanding" — what would I learn from such a thing? How could I trust it to be secure or to not fall over when i enter a weird character? How can I trust something I do not understand or have not read the foundations of? Furthermore, why would I consider myself to have built it?

All of these questions are irrelevant if the objective is 'get this thing working'.

You seem to read a lot into what I wrote, so let me phrase it differently.

These are ways I'd suggest to approach working with LLMs if you enjoy building software, and are trying to find out how it can fit into your workflow.

If this isnt you, these suggestions probably wont work.

> I don't get any enjoyment from "building something without understanding".

That's not what I said. It's about your primary goal. Are you trying to learn technology xyz, and found a project so you can apply it vs you want a solution to your problem, and nothing exists, so you're building it.

What's really important is that wether you understand in the end what the LLM has written or not is 100% your decision.

You can be fully hands off, or you can be involved in every step.

> You can commission a painting from an artist, but it is hubris to point at a painting you bought or commissioned and go "I made that". But somehow it is acceptable to do this for LLMs. That is a baffling mindset to me!

The majority of the work on a lot of famous masterpieces of art was done by apprentices. Under the instruction of a master, but still. No different than someone coming up with a composition, and having AI do a first pass, then going in with photoshop and manually painting over the inadequate parts. Yet people will knob gobble renaissance artists and talk about lynching AI artists.

I've heard this analogy regurgitated multiple times now, and I wish people would not.

It's true that many master artists had workshops with apprenticeships. Because they were a trade.

By the time you were helping to paint portraits, you'd spent maybe a decade learning techniques and skill and doing the unimportant parts and working your way up from there.

It wasn't a half-assed, slop some paint around and let the master come fix it later. The people doing things like portrait work or copies of works were highly skilled and experienced.

Typing "an army of Garfields storming the beach at Normandy" into a website is not the same.

Anti-AI art folks don't care if you photobashed bits of AI composition and then totally painted over it in your own hand, the fact that AI was involved makes it dirty, evil, nasty, sinful and bad. Full stop. Anti-AI writing agents don't care if every word in a manuscript was human written, if you asked AI a question while writing it suddenly you're darth fucking vader.

The correct comparison for some jackass who just prompts something, then runs around calling it art is to a pre-schooler that scribbles blobs of indistinct color on a page, then calls it art. Compare apples to apples.

That's not what a strawman is lol. Me saying the analogy sucks is just criticism.

If you feel judged about using AI, then your choices are (1) don't use it or (2) don't tell people you use it or (3) stop caring what other people think.

Have the courage of your own convictions and own your own actions.

Lately I've been interesting in biosignals, biofeedback and biosynchronization.

I've been really frustrated with the state of Heart Rate Variability (HRV) research and HRV apps, particularly those that claim to be "biofeedback" but are really just guided breathing exercises by people who seem to have the lights on and nobody home. [1]

I could have spent a lot of time reading the docs to understand the Web Bluetooth API and facing up to the stress that getting anything with Bluetooth working with a PC is super hit and miss so estimating the time I'd expect a high risk of spending hours rebooting my computer and otherwise futzing around to debug connection problems.

Although it's supposedly really easy to do this with the Web Bluetooth API I amazingly couldn't find any examples which made all the more apprehensive that there was some reason it doesn't work. [2]

As it was Junie coded me a simple webapp that pulled R-R intervals from my Polar H10 heart rate monitor in 20 minutes and it worked the first time. And in a few days, I've already got an HRV demo app that is superior to the commercial ones in numerous ways... And I understand how it works 100%.

I wouldn't call it vibe coding because I had my feet on the ground the whole time.

[1] for instance I am used to doing meditation practices with my eyes closed and not holding a 'freakin phone in my hand. why they expect me to look at a phone to pace my breathing when it could talk to be or beep at me is beyond me. for that matter why they try to estimate respiration by looking at my face when they could get if off the accelerometer if i put in on my chest when i am lying down is also beyond me.

[2] let's see, people don't think anything is meaningful if it doesn't involve an app, nobody's gotten a grant to do biofeedback research since 1979 so the last grad student to take a class on the subject is retiring right about now...

>When I enter a building, I know that an engineer with a degree, or even a team of them, have meticulously built this building taking into account the material stresses of the ground, the fault lines, the stresses of the materials of construction, the wear amounts, etc.

You can bet that "AI" is coming for this too. The lawsuits that will result when buildings crumble and kill people because an LLM "hallucinated" will be tragic, but maybe we'll learn from it. But we probably won't.

Have you heard of the Horizon IT Post Office Scandal[0]?

> Between 1999 and 2015, more than 900 subpostmasters were wrongfully convicted of theft, fraud and false accounting based on faulty Horizon data, with about 700 of these prosecutions carried out by the Post Office. Other subpostmasters were prosecuted but not convicted, forced to cover illusory shortfalls caused by Horizon with their own money, or had their contracts terminated.
>
> Although many subpostmasters had reported problems with the new software, and Fujitsu was aware that Horizon contained software bugs as early as 1999, the Post Office insisted that Horizon was robust and failed to disclose knowledge of the faults in the system during criminal and civil cases.

(content warning for the article about that for suicide)

Now think of places where LLMs are being deployed:

- accountancy[1][2]

- management systems similar to Horizon IT

- medical workers using it to pass their coursework (A friend of mine is doing a nursing degree in the USA and they are encouraged to use Gemini, and she's already seen someone on the same course use it to complete their medical ethics homework...)

- Ordinary people checking drug interactions[3], learning about pickling (and almost getting botulism), talking to LLMs and getting poisoned by bromide[4]

I build a lot of custom tools, things with like a couple of users. I get a lot of personal satisfaction writing that code.

I think comments on YouTube like "anyone still here in $CURRENT_YEAR" are low effort noise, I don't care about learning how to write a web extension (web work is my day job) so I got Claude to write one for me. I don't care who wrote it, I just wanted it to exist.

I’ve wanted a good markdown editor with automatic synchronization. I used to used inkdrop. Which I stopped using when the developer/owner raised the price to $120/year.

In a couple hours with Claude code, I built a replacement that does everything I want, exactly the way I want. Plus, it integrates native AI chat to create/manage/refine notes and ideas, and it plugs into a knowledge RAG system that I also built using Claude code.

What more could I ask for? This is a tool I wanted for a long time but never wanted to spend the dozens of hours dealing with the various pieces of tech I simply don’t care about long-term.

The incredible thing (to me) is that this isn’t even remotely a new thing: it’s reviewing pull requests vs writing your own code. We all know how different that feels!

To me, using an LLMs is more like having a team of ghostwriters writing your novel. Sure, you "built" your novel but it feels entirely different to writing it yourself.

Wouldn't it be like having a team of software developers writing your code? The analogy doesn't need to be even as far as a different line of work. And for some this (writing to managing) is a natural career progression.

And if you write novels mostly because you enjoy watching them sell, as opposed to sharing ideas with people, you don't care.

To scientists, the purpose of science is to learn more about the world; to certain others, it's about making a number of dollars go up. Mathematicians famously enjoy creating math, and would have no use for a "create more math" button. Musicians enjoy creating music, which is very different from listening to it.

We're all drawn to different vocations, and it's perverse to accept that "maximize shareholder value" is the highest.

I have both; for embedded and backend I prefer entering code; once in the flow, I produce results faster and feel more confident everything is correct. for frontend (except games), i find everything annoying and a waste of time manually, as do all my colleagues. LLMs really made this excellent for our team and myself. I like doing UX, but I like drawing it with a pen and paper and then do experiments with controls/components until it works. This is now all super fast (I usually can just take photo of my drawings and claude makes it work) and we get excellent end results that clients love.

> For me there’s no flow state possible with LLM “coding”.

I would argue that it's the same question as whether it's possible to get into a flow state when being the "navigator" in a pair-programming session. I feel you and agree that it's not quite the same flow state as typing the code yourself, but when a session with a human programmer or Claude Code is going well for me, I am definitely in something quite close to flow myself, and I can spend hours in the back and forth. But as others in this thread said, it's about the size of the tasks you give it.

I can say I feel that flow state sometimes when it all works but I certainly don't when it doesn't work.

The other day I was making changes to some CSS that I partially understood.

Without an LLM I would looked at the 50+ CSS spec documents and the 97% wrong answers on Stack Overflow and all the splogs and would have bumbled around and tried a lot of things and gotten it to work in the end and not really understood why and experienced a lot of stress.

As it was I had a conversation with Junie about "I observe ... why does it work this way?", "Should I do A or do B?", "What if I did C?" and came to understand the situation 100% and wrote a few lines of code by hand that did the right thing. After that I could have switched it to Code mode and said "Make it so!" but it was easy when I understood it. And the experience was not stressful at all.

I could imagine a world where LLM coding was fun. It would sound like "imagine a game, like Galaxians but using tractor trailers, and as a first person shooter." And it pumps out a draft and you say, "No, let's try it again with an army of bagpipers."

In other words, getting to be the "ideas guy", but without sounding like a dipstick who can't do anything.

I don't think we're anywhere near that point yet. Instead we're at the same point where we are with self-driving: not doing anything but on constant alert.

I'll be honest, the bagpiper 3D models were way better than I expected! That game's a bit too hard though, you have to run sideways pretty quickly to avoid being destroyed by incoming fire.

That is deeply impressive. The code is quite readable. I appreciate that it even got the "joke" with bagpipers. I know it's just recycling other people's jokes, but it's still quite the feat.

I have never used one of these. I'm going to have to try it.

(As a FW-curious noob I wondered if Gemini understand, Why do foxes struggle, compared to wolves of all gender)

>Yes, many fox species, particularly the red fox, exhibit more neotenous (juvenile-like) traits compared to wolves, such as shorter muzzles, larger eyes relative to head size, and different skull development, reflecting a divergence in evolutionary paths within the canid family, with foxes often retaining softer, more generalized features compared to the larger, more specialized wolf. While wolves are highly SOCIAL pack animals with traits adapted for cooperative hunting, foxes are generally SOLITARY, and this difference in lifestyle and morphology highlights their distinct evolutionary strategies, with foxes leaning towards juvenile-like features in their adult forms.

Wolves work together to bring down large prey and with their large digestive tracts they "wolf it down".

Foxes have a small digestive tract (makes them lightweight so they can jump and pounce on prey) and can't even eat a whole rabbit so they eat a bit and bury the rest under a layer of dirt (to hide it from other animals) and leaf litter (to hide it from birds.) A fox will lose an occasional cache to another fox but will occasionally find a cache from another fox so it evens out. Foxes in a given territory usually have some family relationship so it works from a sociobiological level, it's their form of "social hunting".

For me this week it's been about practicing autonomic control, I've been building biofeedback systems and getting to the bottom of heart rate variability and working towards a biosynchronization demo. Also working to start an anime theme song cover band (Absolute Territory) where I am clearly the "kitsune" (AT-00) but more of a band manager than a mascot. I've got the all-purpose guitarist (AT-01) but I'm still casting AT-02, AT-03 and such.

... and boy do I have a technique now to find out people and places that are identity driven and those who are not.

One interesting discovery is that people are really dour about it at places that hire a lot of enby's [1] but overall enby people who are by themselves or working at places where 10% or fewer people are enby really dig somebody who represents "oceanic reservoir of calm" [2] with a kidult presentation of self.

So these 10% enby presumably tend to be "silent enbys"?

Hmm. Gotta investigate whether HN (the HQ) has (or aims for) the magic ~10% enby. (I had suspected as such but you again voiced what was in my subconscious!!)

>
There's a reason why bagpipes are banned under the Geneva convention!

I know this is not Reddit, but when I see such a comment, I can't resist posting a video of "the internet's favorite song" on an electrical violin and bagpipes:

> Through the Fire and Flames (Official Video) - Mia x Ally

Yes, but I didn't bother here (not part of the original prompt).

You're welcome to drop the HTML into a coding agent and tell it to do that. In my experience you usually have to decide how you want that to work - I've had them build me on-screen D-Pad controls before but I've also tried things like getting touch-to-swipe plus an on-screen fire button.

There are multiple self driving car companies that are fully autonomous and operating in several cities in the US and China. Waymo has been operating for many years.

There are full self driving systems that have been in operation with human driver oversight from multiple companies.

And the capabilities of the LLMs in regards to your specific examples were demonstrated below.

The inability of the public to perceive or accept the actual state of technology due to bias or cognitive issues is holding back society.

It's a lot of mistrust and fear, too - a computer could never be as good at driving as a person!

And yet, over the years many things have just been accepted. Satnav for example, I grew up with my mom having the map in her lap, or my dad writing down directions. Later on we had a route planner on diskettes (I think) and a printout of the route. And my dad now has had a satnav in his car for near enough two decades. I'm sure they like everyone else ran into the quirks of satnav, but I don't think there was nearly as much "fear" and doubt for satnav as there is for self-driving cars and nowadays LLMs / coding agents. Or I'm misremembering it and have rose-tinted glasses, I also remember the brouhaha of people driving into canals because the satnav told them to turn left.

You're not alone. I definitely feel like this is / will be a major adaptation required for software engineers going forward. I don't have any solutions to offer you - but I will say that the state that's enabled by fast feedback loops wasn't always the case. For most of my career build times were much, much longer than they are today, as an example. We had to work around that to maintain flow, and we'll have to work around this, now.

You can definitely keep tweaking. It's also helpful just to ask it about what your possible concerns are and it will tell you and explain what it did.

I spent a good chunk of 2025 long time being super super careful & specific, using mostly very very cheap DeepSeek and just leading it by the leash at every moment and studying the output. It still felt like a huge win. But with more recent models, I have trust that they are doing ok, and I'm better at asking some questions once the code is written to hone my understanding. And mostly I just trust it now! I don't have to look carefully and tweak to exacting standards, because I've seen it do a ton of good work & am careful in what I ask.

There's other tactics that help. Rather than stare carefully at the code, making sure you and the AI are both running the program frequently, have a rig to test what's under development (ideally I'm an integration test type of way, which it can help set-up!). And then having what good programmers have long had, good observability tools at their back. Be that great logging or ideally sweet tracing. We have such better tools to see the high level behavior of systems now. AI with some prompts to go there can be extremely good about helping enhance that view.

It is going to feel different. But there's a lot you can do to get much better loops.

I feel the same way often but I find it to be very similar to coding. Whether coding or prompting when I’m doing rote, boring work I find it tedious. When I am solving a hard problem or designing something interesting I am engaged.

My app is fairly mature with well established patterns, etc. When I’m adding “just CRUD” as part of a feature it’s very tedious to prompt agents, reviewing code, rinse & repeat. Were I actually writing the code by hand I would probably be less productive and just as bored/unsatisfied.

I spent a decent amount of time today designing a very robust bulk upload API (compliance fintech, lots of considerations to be had) for customers who can’t do a batch job. When it was finished I was very pleased with the result and had performance tests and everything.

I gotta say, the "sitting around waiting" comment hits - I have the same with current-day merge request based development, a lot of time is fragmented because I'm waiting for the CI to finish. I've got seven open merge requests at the moment, some of which have been open since before the holidays. It's a lot of fragmented waiting, fixing, prodding people to review code, and shitposting on HN to pass the time. It's uh. Not healthy.

But this is my reality in my current line of work, a lot of relatively simple work but a lot of processes and checks to conform to rules (that I set myself lol) and not break existing functionality.

I've hit flow state setting it up to fly. When it flys is when the human gets out of the loop so the AI can look at the thing itself and figure out why centering the div isn't working to center the div, or why the kernel isn't booting. Like, getting to a point, pre-AI, where git bisect running in a loop is the flow state. Now, with ai, setting that up is the flow.

I feel differently! My background isn't programming, so I frequently feel inhibited by coding. I've used it for over a decade but always as a secondary tool. Its fun for me to have a line of reasoning, and be able to toy with and analyze a series of questions faster than I used to be able to.

Ditto. Coding isn't what i specifically do, but it's something i will choose to do when it's the most efficient solution to a problem. I have no problem describing what i need a program to do and how it should do so in a way that could be understandable even to a small child or clever golden retriever, but i'm not so great at the part where you pull out that syntactic sugar and get to turning people words into computer words. LLMs tend to do a pretty good job at translating languages regardless of whether i'm talking to a person or using a code editor, but i don't want them deciding what i wanted to say for me.

Yes, if your goal is to build a duck, and to understand what goes into building a duck. A lot of people derive joy from learning how to do something, not merely seeing the end result.

Do you want to make one beautiful intricate table that will last ages. Or do you need a table ASAP because you have guests coming and your end-table can barely fit a pint and a bag of chips?

It's perfectly OK to want to craft something beautiful and understand every single line of code deeply. But it also takes more time than just solving the problem with sufficient quality.

Coding with an LLM seems like it’s often more editing in service of less writing.

I get this is a very simplistic way of looking at it and when done right it can produce solutions, even novel solutions, that maybe you wouldn’t have on your own. Or maybe it speeds up a part of the writing that is otherwise slow and painful. But I don’t know, as somebody who doesn’t really code every time I hear people talk about it that’s what it sounds like to me.

Yes this is exactly what I feel. I disconnect enough that if it’s really taking its time I will pull up Reddit and now that single prompt cost me half an hour.

Well are you the super developer than never run into issues, challenges? For me and I think most developers, coding is like a continuous stream of problems you need to solve. For me a LLM is very useful, because I can now develop much faster. Don't have to think which sorting algoritm should be used or which trigonometric function I need for a specific case. My LLM buddy solves most of those issues.

When you don't know the answer to a question you ask an LLM, do you verify it or do you trust it?

Like, if it tells you merge sort is better on that particular problem, do you trust it or do you go through an analysis to confirm it really is?

I have a hard time trusting what I don't understand. And even more so if I realize later I've been fooled. Note that it's the same with human though.
I think I only trust technical decision I don't understand when I deem the risk of being wrong low enough. Overwise I'll invest in learning and understanding enough to trust the answer.

For all these "open questions" you might have it is better to ask the LLM write a benchmark and actually see the numbers. Why rush, spend 10 minutes, you will have a decision backed by some real feedback from code execution.

But this is just a small part from a much grander testing activity that needs to wrap the LLM code. I think my main job moved to 1. architecting and 2. ensuring the tests are well done.

What you don't test is not reliable yet, looking at code is not testing, it's "vibe-testing" and should be an antipattern, no LGTM for AI code. We should rely on our intuition alone because it is not strict enough, and it makes everything slow - we should not "walk the motorcycle".

Ok. I also have the intuition that more tests and formal specifications can help there.

So far, my biggest issue is, when the code produced is incorrect, with a subtle bug, then I just feel I have wasted time to prompt for something I should have written because now I have to understand it deeply to debug it.

If the test infrastructure is sound, then maybe there is a gain after all even if the code is wrong.

Often those kind of performance things just don't matter.

Like right now I am working on algorithms for computing heart rate variability and only looking at a 2 minute window with maybe 300 data points at most so whether it is N or N log N or N^2 is beside the point.

When I know I computing the right thing for my application and know I've coded it up correctly and I am feeling some pain about performance that's another story.

> I have a hard time trusting what I don't understand

Who doesn't? But we have to trust them anyway, otherwise everyone should get a PhD on everything.

Also for people who "has a hard time trusting", they might just give up when encountering things they don't understand. With AI at least there is a path for them to keep digging deeper and actually verify things to whatever level of satisfaction they want.

My issue is, LLM fooled me more than a couple of times with stupid but difficult to notice bugs.
At that point, I have hard time to trust them (but keep trying with some stuff).

If I asked someone for something and found out several time that the individual is failing, then I'll just stop working with them.

Edit: and to avoid with just anthropomorphizing LLM too much, the moment I notice a tool I use bug to point to losing data for example, I reconsider real hard before I use it again or not.

IME I don't learn by reading or watching, only by wrestling with a problem.
ATM, I will only do it if the problem does not feel worth learning about (like jenkinsfile, gradle scripting).

But yes, the bench result will tell something true.

See, I’m with you, but in my day to day work I almost never could almost never get into a flow state while coding, because very little of my work involves creating things or solving real problems; it typically involves just trying to mentally untangle huge rat nests, Jenna-ing bug fixes and the occasional feature in, and then spending a bunch of time testing to make sure I didn’t break anything, no flow involved. I’ve been grudgingly using Cursor heavily for the past few weeks and it’s been helping make all of this significantly more bearable.

LLMs aren’t replacing the joy of coding for me, but they do seem to be helping me deal with the misery of being a professional coder.

> I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer...

Reminds me of this excerpt from Richard Hamming's book:

> Finally, a more complete, and more useful, Symbolic Assembly Program (SAP) was devised—after more years than you are apt to believe during which most programmers continued their heroic absolute binary programming. At the time SAP first appeared I would guess about 1% of the older programmers were interested in it—using SAP was “sissy stuff”, and a real programmer would not stoop to wasting machine capacity to do the assembly. Yes! Programmers wanted no part of it, though when pressed they had to admit their old methods used more machine time in locating and fixing up errors than the SAP program ever used. One of the main complaints was when using a symbolic system you do not know where anything was in storage—though in the early days we supplied a mapping of symbolic to actual storage, and believe it or not they later lovingly pored over such sheets rather than realize they did not need to know that information if they stuck to operating within the system—no! When correcting errors they preferred to do it in absolute binary addresses.

I think this is beside the point, because the crucial change with LLMs is that you don’t use a formal language anymore to specify what you want, and get a deterministic output from that. You can’t reason with precision anymore about how what you specify maps to the result. That is the modal shift that removes the “fun” for a substantial portion of the developer workforce.

its not not about fun. when I'm going through the actual process of writing a function, I think about design issues. about how things are named, about how the errors from this function flow up. about how scheduling is happening. about how memory is managed. I compare the code to my ideal, and this is the time where I realize that my ideal is flawed or incomplete.

I think alot of us dont get everything specced out up front, we see how things fit, and adjust accordingly. most of the really good ideas I've had were not formulated in the abstract, but realizations had in the process of spelling things out.

I have a process, and it works for me. Different people certainly have other ones, and other goals. But maybe stop telling me that instead of interacting with the compiler directly its absolutely necessary that instead I describe what I want to a well meaning idiot, and patiently correct them, even though they are going to forget everything I just said in a moment.

> ... stop telling me that instead of interacting with the compiler directly its absolutely necessary that instead I describe what I want to a well meaning idiot, and patiently correct them, even though they are going to forget everything I just said in a moment.

This perfectly describes the main problem I have with the coding agents. We are told we should move from explicit control and writing instructions for the machine to pulling the slot lever over and over and "persuading the machine" hoping for the right result.

I'd be interested in learning more about your workflow. I've certainly used plaintext files (and similar such things) to aid in project planning, but I've never used paper beyond taking a few notes here and there.

Not who you’re replying to, but I do this as well. I carry a pocket notebook and write paragraphs describing what I want to write. Sometimes I list out the fields of a data structure. Then I revise. By the time I actually write the code, it’s more like a recitation. This is so much easier than trying to think hard about program structure while logged in to my work computer with all the messaging and email.

> because the crucial change with LLMs is that you don’t use a formal language anymore to specify what you want, and get a deterministic output from that

You don't just code, you also test, and your safety is just as good as your test coverage and depth. Think hard about how to express your code to make it more testable. That is the single way we have now to get back some safety.

But I argue the manual inspection of code and thinking it through in your head is still not strict coding, it is vibe-testing as well, only code backed by tests is not vibe-based. If needed use TLA+ (generated by LLM) to test, or go as deep as necessary to test.

I don't know what book you're talking about, but it seems that you intend to compare the switch to an AI-based workflow to using a higher-level language. I don't think that's valid at all. Nobody using Python for any ordinary purpose feels compelled to examine the resulting bytecode, for example, but a responsible programmer needs to keep tabs on what Claude comes up with, configure a dev environment that organizes the changes into a separate branch (as if Claude were a separate human member of a team) etc. Communication in natural language is fundamentally different from writing code; if it weren't, we'd be in a world with far more abundant documentation. (After all, that should be easier to write than a prompt, since you already have seen the system that the text will describe.)

> Nobody using Python for any ordinary purpose feels compelled to examine the resulting bytecode, for example,

The first people using higher level languages did feel compelled to. That's what the quote from the book is saying. The first HLL users felt compelled to check the output just like the first LLM users.

But there is no reason to suppose that responsible SWEs would ever be able to stop doing so for an LLM, given the reliance on nondeterminism and a fundamentally imprecise communication mechanism.

That's the point. It's not the same kind of shift at all.

Assembly was a "high level" language when it was new -- it was far more abstract than entering in raw bytes. C was considered high level later on too, even though these days it is seen as "low level" -- everything is relative to what else is out there.

The same pattern held through the early days of "high level" languages that were compiled to assembly, and then the early days of higher level languages that were interpreted.

Read the famous "Story of Mel" [1] about Mel Kaye, who refused to use optimizing assemblers in the late 1950s because "you never know where they are going to put things". Even in the 1980s you used to find people like that.

I don't think that does count against the narrative? The narrative is just that each time we've moved up the abstraction chain in generating code, there have been people who have been skeptical of the new level of abstraction. I would say that it's usually the case that highly skilled operators at the previous level remain more effective than the new adopters of the next level. But what ends up mattering more in the long run is that the higher level of abstraction enables a lot more people to get started and reach a basic level of capability. This is exactly what's happening now! Lots of experienced programmers are not embracing these tools, or are, but are still more effective just writing code. But way more people can get into "vibe coding" with some basic level of success, and that opens up new possibilities.

The narrative is that non-LLM adopters will be left behind, lose their jobs, are Luddites, yadda yadda yadda because they are not moving up the abstraction layers by adopting LLMs to improve their output. There is no point in the timeframe of the story at which Mel would have benefitted from a move to a higher abstraction level by adopting the optimizing compiler because its output will always be drastically inferior to his own using his native expertise.

That's not the narrative in this thread. That's a broader narrative than the one in this thread.

And yes, as I said, the point is not that Mel would benefit, it's that each time a new higher level of abstraction comes onto the scene, it is accessible to more people than the previous level. This was the pattern with machine code to symbolic assembly, it was the pattern with assembly to compiled languages, with higher level languages, and now with "prompting".

The comment I originally replied to implied that this current new abstraction layer is totally different than all the previous ones, and all I said is that I don't think so, I think the comparison is indeed apt. Part of that pattern is that a lot of new people can adopt this new layer of abstraction, even while many people who already know how to program are likely to remain more effective without it.

> you intend to compare the switch to an AI-based workflow to using a higher-level language.

That was the comparison made. AI is an eerily similar shift.

> I don't think that's valid at all.

I dont think you made the case by cherry picking what it can't do. This is exactly the same situation, as the time SAP appeared. There weren't symbols for every situation binary programmers were using at the time. This doesn't change the obvious and practical improvement that abstractions provided. Granted, I'm not happy about it, but I can't deny it either.

Contra your other replies, I think this is exactly the point.

I had an inkling that the feeling existed back then, but I had no idea it was documented so explicitly. Is this quote from The Art of Doing Science and Engineering?

In my feed 'AI hype' outnumbers 'anti-AI hype' 5-1. And anti-hype moderates like antirez and simonw are rare. To be a radical in ai is to believe that ai tools offer a modest but growing net positive utility to a modest but growing subset of hackers and professionals

In theory “not having to churn out yet another crud app” doesn’t require AI, any ol code generator will do. AI is a really expensive way (in terms of gpus/tpus) to generate boilerplate, but as long as that cost is massively subsidized by investors, you may as well use it.

The problem I see is not so much in how you generate the code. It is about how to maintain the code. If you check in the AI generated code unchanged then do you start changing that code by hand later? Do you trust that in the future AI can fix bugs in your code. Or do you clean up the AI generated code first?

I've never worked in web development, where it seems to me the majority of LLM coding assistants are deployed.

I work on safety critical and life sustaining software and hardware. That's the perspective I have on the world. One question that comes up is "why does it take so long to design and build these systems?" For me, the answer is: that's how long it takes humans to reach a sufficient level of understanding of what they're doing. That's when we ship: when we can provide objective evidence that the systems we've built are safe and effective. These systems we build, which are complex, have to interact with the real world, which is messy and far more complicated.

Writing more code means that's more complexity for humans (note the plurality) to understand. Hiring more people means that's more people who need to understand how the systems work. Want to pull in the schedule? That means humans have to understand in less time. Want to use Agile or this coding tool or that editor or this framework? Fine, these tools might make certain tasks a little easier, but none of that is going to remove the requirement that humans need to understand complex systems before they will work in the real world.

So then we come to LLMs. It's another episode of "finally, we can get these pesky engineers and their time wasting out of the loop". Maybe one day. But we are far from that today. What matters today is still how well do human engineers understand what they're doing. Are you using LLMs to help engineers better understand what they are building? Good. If that's the case you'll probably build more robust systems, and you _might_ even ship faster.

Are you trying to use LLMs to fool yourself into thinking this still isn't the game of humans needing to understand what's going on? "Let's offload some of the understanding of how these systems work onto the AI so we can save time and money". Then I think we're in trouble.

> Are you trying to use LLMs to fool yourself into thinking this still isn't the game of humans needing to understand what's going on?

This is a key question. If you look at all the anti-AI stuff around software engineering, the pervading sentiment is “this will never be a senior engineer”. Setting aside the possibility of future models actually bridging this gap (this would be AGI), let’s accept this as true.

You don’t need an LLM to be a senior engineer to be an effective tool, though. If an LLM can turn your design into concrete code more quickly than you could, that gives you more time to reason over the design, the potential side effects, etc. If you use the LLM well, it allows you to give more time to the things the LLM can’t do well.

Fully agree. In my own usage of AI (which I came to a bit late but have tried to fully embrace so I know what it can and can't do) I've noticed a very unusual side effect: I spend way more of my time documenting and reviewing designs than I used to, and that has been a big positive. I've always been very (maybe too) thoughtful about design and architecture, but I usually focused on high-level design and then would get to some coding as a way of evaluating/testing my designs. I could then throw away v0 using lessons learned and start a v1 on a solid track. Now however, I find myself able to get a lot further in nailing down the design to the point I don't have to build and throw away v0. The prototype is often highly salvageable with the help of the LLM doing the refactoring/iterating that used to make "starting over" a more optimal path. That in turn allows me to maintain the context and velocity of the design much better since there aren't days, or weeks, or even months between the "lessons learned" that then have to go back and revise the design.

The caveat here though, is if I didn't have the decades of experience writing/designing software by hand, I don't think I'd have the skills needed to reap the above benefit.

" They make it easier to explore ideas, to set things up, to translate intent into code across many specialized languages. But the real capability—our ability to respond to change—comes not from how fast we can produce code, but from how deeply we understand the system we are shaping. Tools keep getting smarter. The nature of learning loop stays the same."

Learning happens when your ideas break, when code fails, unexpected things happen. And in order to have that in a coding agent you need to provide a sensitive skin, which is made of tests, they provide pain feedback to the agent. Inside a good test harness the agent can't break things, it moves in a safe space with greater efficiency than before. So it was the environment providing us with understanding all alone, and we should make an environment where AI can understand what are the effects of its actions.

maybe. I think we're really just starting this, and I suspect that trying to fuse neural networks with symbolic logic is a really interesting direction to try to explore.

that's kind of not what we're talking about. a pretty large fraction of the community thinks programming is stone cold over because we can talk to an LLM
and have it spit out some code that eventually compiles.

personally I think there will be a huge shift in the way things are done. it just won't look like Claude.

I don't know why you're being downvoted, I think you're right.

I think LLMs need different coding languages, ones that emphasise correctness and formal methods. I think we'll develop specific languages for using LLMs with that work better for this task.

Of course, training an LLM to use it then becomes a chicken/egg problem, but I don't think that's insurmountable.

I don't think "understanding" should be the criteria, you can't commit your eyes in the PR. What you can commit is a test that enforces that understanding programatically. And we can do many many more tests now than before. You just need to ensure testing is deep and well designed.

I suspect that we are going to have a wave of gurus who show up soon to teach us how to code with LLMs. There’s so much doom and gloom in these sorts of threads about the death of quality code that someone is going to make money telling people how to avoid that problem.

The scenario you describe is a legitimate concern if you’re checking in AI generated code with minimal oversight. In fact I’d say it’s inevitable if you don’t maintain strict quality control. But that’s always the case, which is why code review is a thing. Likewise you can use LLMs without just checking in garbage.

The way I’ve used LLMs for coding so far is to give instructions and then iterate on the result (manually or with further instructions) until it meets my quality standards. It’s definitely slower than just checking in the first working thing the LLM churns out, but it’s sill been faster than doing it myself, I understand it exactly as well because I have to in order to give instructions (design) and iterate.

My favorite definition of “legacy code” is “code that is not tested” because no matter who writes code, it turns into a minefield quickly if it doesn’t have tests.

How do you know that it's actually faster than if you'd just written it yourself? I think the review and iteration part _is_ the work, and the fact that you started from something generated by an LLM doesn't actually speed things up. The research that I've seen also generally backs this idea up -- LLMs _feel_ very fast because code is being generated quickly, but they haven't actually done any of the work.

Because I’ve been a software engineer for over 20 years. If I look at a feature and feel like it will take me a day and an LLM churns it out in a hour including the iterating, I’m confident that using the LLM was meaningfully faster. Especially since engineers (including me) are notoriously bad at accurate estimation and things usually take at least twice as long as they estimate.

I have tested throwing several features at an LLM lately and I have no doubt that I’m significantly faster when using an LLM. My experience matches what Antirez describes. This doesn’t make me 10x faster, mostly because so much of my job is not coding. But in term of raw coding, I can believe it’s close to 10x.

> I know exactly what the result should be, the LLM is just typing it for me.

This is the mental model people should be working with. The LLM is there to tighten the loop from thought to code. You doing need to test it like an engineer. You just need to use it to make you more efficient.

It so happens that you *can^ give an LLM half-baked thoughts and it will sometimes still do a good job because the right thing is so straightforward. But in general the more vague and unclear your own thoughts, the lower quality the results, necessitating more iterations to refine.

> My favorite definition of “legacy code” is “code that is not tested” because no matter who writes code, it turns into a minefield quickly if it doesn’t have tests.

Unfortunately, "tests" don't do it, they have to be "good tests". I know, because I work on a codebase that has a lot of tests and some modules have good tests and some might as well not have tests because the tests just tell you that you changed something.

> My favorite definition of “legacy code” is “code that is not tested” because no matter who writes code, it turns into a minefield quickly if it doesn’t have tests.

On the contrary, legacy code has, by definition, been battle tested in production. I would amend the definition slightly to:

“Legacy code is code that is difficult to change.”

Lacking tests is one common reason why this could be, but not the only possible reason.

It’s from Working Effectively with Legacy Code. I don’t recall the exact definition but it’s something to that effect. Legacy = lack of automated tests.

The biggest barrier to changing code is usually insufficient automated testing. People are terrified of changing code when they can’t verify the results before breaking production.

More glibly legacy code is “any code I don’t want to deal with”. I’ve seen code written 1 year prior officially declared “legacy” because new coding standards were being put in place and no one wanted to update the old code to match.

I think it was Cory Doctorow who compared AI-generated code to asbestos.
Back in its day,
asbestos was in everything,
because of how useful it seemed.
Fast forward decades and now asbestos abatement is a hugely expensive and time-consuming requirement for any remodeling or teardown project.
Lead paint has some of the same history.

I see where you're coming from, and I agree with the implication that this is more of an issue for inexperienced devs. Having said that, I'd push back a bit on the "legacy" characterization.

For me, if I check in LLM-generated code, it means I've signed off on the final revision and feel comfortable maintaining it to a similar degree as though it were fully hand-written. I may not know every character as intimately as that of code I'd finished writing by hand a day ago, but it shouldn't be any more "legacy" to me than code I wrote by hand a year ago.

It's a bit of a meme that AI code is somehow an incomprehensible black box, but if that is ever the case, it's a failure of the user, not the tool. At the end of the day, a human needs to take responsibility for any code that ends up in a product. You can't just ship something that people will depend on not to harm them without any human ever having had the slightest idea of what it does under the hood.

Some of those words appear in my comment, but not in the way you're implying I used them.

My argument was that 1) LLM output isn't inherently "legacy" unless vibe coded, and 2) one should not vibe code software that others depend on to remain stable and secure. Your response about "abandonware" is a non sequitur.

I presume that through some process one can exorcise the legacy/vibe-codiness away. Perhaps code review of every line? (This would imply that the bottleneck to LLM output is human code review.) Or would having the LLM demonstrate correctness via generated tests be sufficient?

Just to clarify, you're inferring several things that I didn't say:

* I was agreeing with you that all vibe code is effectively legacy, but obviously not all legacy code is vibe code. Part of my point is also that not all LLM code is vibe code.

* I didn't comment on the dependability of legacy code, but I don't believe that strict vibe code should ever be depended on in principle.

As far as non-vibe coding with LLMs, I'd definitely suggest some level of human review and participation in the overall structure/organization. Even if the developer hasn't pored through it line by line, they should have signed off on the tech stack/dependencies/architecture and have some idea of what the file layout and internal modules/interfaces look like. If a major bug is ever discovered, the developer should know enough to confidently code review the fix or implement it by hand if necessary.

Take responsibility by leaving a good documentation of your code and a beefy set of tests, future agents and humans will have a point to bootstrap from, not just plain code.

Depends on what you do. When I'm using LLMs to generate code for projects I need to maintain (basically, everything non-throw-away-once-used), I treat it as any other code I'd write, tightly controlled with a focus on simplicity and well-thought out abstractions, and automated testing that verify what needs to be working. Nothing gets "merged" into the code without extensive review, and me understanding the full scope of the change.

So with that, I can change the code by hand afterwards or continue with LLMs, it makes no difference, because it's essentially the same process as if I had someone follow the ideas I describe, and then later they come back with a PR. I think probably this comes naturally to senior programmers and those who had a taste of management and similar positions, but if you haven't reviewed other's code before, I'm not sure how well this process can actually work.

At least for me, I manage to produce code I can maintain, and seemingly others to, and they don't devolve into hairballs/spaghetti. But again, requires reviewing absolutely every line and constantly edit/improve.

We recently got a PR from somebody adding a new feature and the person said he doesn't know $LANG but used AI.

The problem is, that code would require a massive amount of cleanup. I took a brief look and some code was in the wrong place. There were coding style issues, etc.

In my experience, the easy part is getting something that works for 99%. The hard part is getting the architecture right, all of the interfaces and making sure there are no corner cases that get the wrong results.

I'm sure AI can easily get to the 99%, but does it help with the rest?

> I'm sure AI can easily get to the 99%, but does it help with the rest?

Yes the AI can help with 100% is it. But the operator of the AI needs to be able to articulate this to the AI .

I've been in this position, where I had no choice but to use AI to write code to fix bugs in another party's codebase, then PR the changes back to the codebase owners. In this case it was vendor software that we rely on which the vendor hadn't fixed critical bugs in yet. And exactly as you described, my PR ultimately got rejected because even though it fixed the bugs in the immediate sense, it presented other issues due to not integrating with the external frameworks the vendor used for their dev processes. At which point it was just easier for the vendor to fix the software their way instead of accept my PR. But the point is that I could have made the PR correct in the first place, if I as the AI operator had the knowledge needed to articulate these more detailed and nuanced requirements to the AI. Since I didn't have this information then the AI generated code that worked but didn't meet the vendors spec. This type of situation is incredibly easy to fall into and is a good example of why you still need a human at the wheel on projects to set the guidance but you don't necessarily need the human to be writing every line of code.

I don't like the situation much but this is the reality of it. We're basically just code reviewers for AI now

Yeah, so what I'm mostly doing, and advocate for others to do, is basically the pure opposite of that.

Focus on architecture, interfaces, corner-cases, edge-cases and tradeoffs first, and then the details within that won't matter so much anymore. The design/architecture is the hard part, so focus on that first and foremost, and review + throw away bad ideas mercilessly.

Yes it does... but only in the hands of an expert who knows what they are doing.

I'd treat PRs like that as proof of concepts that the thing that can be done, but I'd be surprised if they often produced code that should be directly landed.

In the hands of an expert… right. So is it not incredibly irresponsible to release these tools into the wild, and expose it those who are not experts? They will actually become incredibly worse off. Ironically this does not ‘democratise’ intelligence at all - the gap widens between experts and the rest.

I sometimes wonder what would have happened if OpenAI had built GPT3 and then GPT-4 and NOT released them to the world, on the basis that they were too dangerous for regular people to use.

That nearly happened - it's why OpenAI didn't release open weight models past GPT2, and it's why Google didn't release anything useful built on Transformers despite having invented the architecture.

If we lived in the world today, LLMs would be available only to a small, elite and impossibly well funded class of people. Google and OpenAI would solely get to decide who could explore this new world with them.

With all due respect I don’t care about an acceleration in writing code - I’m more interested in incremental positive economic impact. To date I haven’t seen anything convince me that this technology will yield this.

Producing more code doesn’t overcome the lack of imagination, creativity and so on to figure out what projects resources should be invested in. This has always been an issue that will compound at firms like Google who have an expansive graveyard of projects laid to rest.

In fact, in a perverse way, all this ‘intelligence’ can exist. At the same time humans can get worse in their ability to make judgments in investment decisions.

You mean the net benefit in widespread access to LLMs?

I get the impression there's no answer here that would satisfy you, but personally I'm excited about regular people being able to automate tedious things in their lives without having to spend 6+
months learning to program first.

And being able to enrich their lives with access to as much world knowledge as possible via a system that can translate that knowledge into whatever language and terminology makes the most sense to them.

The average person already automates a lot of things in their day to day lives. They spend far less time doing the dishes, laundry, and cleaning because parts of those tasks have been mechanized and automated. I think LLMs probably automate the wrong thing for the average person (i.e., I still have to load the laundry machine and fold the laundry after) but automation has saved the average person a lot of time

For example, my friend doesn’t know programming but his job involves some tedious spreadsheet operations. He was able to use an LLM to generate a Python script to automate part of this work. Saving about 30 min/day. He didn’t review the code at all, but he did review the output to the spreadsheet and that’s all that matters.

His workplace has no one with programming skills, this is automation that would never have happened. Of course it’s not exactly replacing a human or anything. I suppose he could have hired someone to write the script but he never really thought to do that.

A work colleague had a tedious operation involving manually joining a bunch of video segments together in a predictable pattern. Took them a full working day.

They used "just" ChatGPT on the web to write an automation. Now the same process takes ~5 minutes of work. Select the correct video segments, click one button to run script.

The actual processing still takes time, but they don't need to stand there watching it progress so they can start the second job.

And this was a 100% non-tecnical marketing person with no programming skills past Excel formulas.

My favorite anecdotal story here is that a couple of years ago I was attending a training session at a fire station and the fire chief happened to mention that he had spent the past two days manually migrating contact details from one CRM to another.

I do not want the chief of a fire station losing two days of work to something that could be scripted!

I don't want my doctor to vibe script some conversion only to realize weeks or months later it made a subtle error in my prescription.
I want both of them to have enough fund to hire someone to do it properly.
But wanting is not enough unfortunately...

Humans make subtle errors all the time too though. AI results still need to be checked over for anything important, but it's on a vector toward being much more reliable than a human for any kind of repetitive task.

Currently, if you ask an LLM to do something small and self-contained like solve leetcode problems or implement specific algorithms, they will have a much lower rate of mistakes, in terms of implementing the actual code, than an experienced human engineer. The things it does badly are more about architecture, organization, style, and taste.

But with a software bug, the error becomes rapidly widespread and systematic, whereas human error are often not. Doing wrong with a couple of prescription because the doc worked for 12+ hrs is different from systematically doing wrong on a significant number of prescriptions until someone double check the results.

I agree with the excel thing.
Not with thinking it can't happen with vibecoded python.

I think handling sensitive data should be done by professional.
A lawyer handles contracts, a doctor handles health issue and a programmer handles data manipulation through programs.
This doesn't remove risk of errors completely, but it reduces it significantly.

In my home, it's me who's impacted if I screw up a fix in my plumbing, but I won't try to do it at work or in my child's school.

I don't care if my doctor vibe codes an app to manipulate their holidays pictures, I care if they do it to manipulate my health or personal data.

Of course issues CAN happen with Python, but at least with Python we have tools to check for the issues.

Bunch of your personal data is most likely going through some Excel made by a now-retired office worker somewhere 15 years ago. Nobody understands how the sheet works, but it works so they keep using it :) A replacement system (a massive SaaS application) has been "coming soon" for 8 years and cost millions, but it still doesn't work as well as the Excel sheet.

You can apply the same logic to all technologies, including programming languages, HTTP, cryptography, cameras, etc. Who should decide what's a responsible use?

I'm curious about the economic aspects of this. If only experts can use such tools effectively, how big will the total market be and does that warrant the investments?

For companies, if these tools make experts even more special, then experts may get more power certainly when it comes to salary.

So the productively benefits of AI have to be pretty high to overcome this. Does AI make an expert twice as productive?

Coding style can be deterministically checked for, and should be checked, automatically during linting. And no PR should get a single human pair of eyes, except for the author, looking at it until all CI checks have passed.

Many many other stylistic choices and code complexity can be automatically checked, why aren't you doing it?

> We recently got a PR from somebody adding a new feature and the person said he doesn't know $LANG but used AI.

"Oh, and check it out: I'm a bloody genius now! Estás usando este software de traducción in forma incorrecta. Por favor, consulta el manual. I don't even know what I just said, but I can find out!"

I think we will find out that certain languages, frameworks and libraries are easier for AI to get all the way correct. We may even have to design new languages, frameworks and libraries to realize the full promise of AI. But as the ecosystem around AI evolves I think these issues will be solved.

The solution for low performers is very close oversight. If you imagine an LLM as a very junior engineer who needs an inordinate amount of hand holding (but who can also read and write about 1000x faster than you and who gets paid approximately nothing), you can get a lot of useful work out of it.

A lot of the criticisms of AI coding seem to come from people who think that the only way to use AI is to treat it as a peer. “Code this up and commit to main” is probably a workable model for throwaway projects. It’s not workable for long term projects, at least not currently.

A Junior programmer is a total waste of time if they don't learn. I don't help Juniors because it is an effective use of my time, but because there is hope that they'll learn and become Seniors. It is a long term investment. LLMs are not.

It’s a metaphor. With enough oversight, a qualified engineer can get good results out of an underperforming (or extremely junior) engineer. With a junior engineer, you give the oversight to help them grow. With an underperforming engineer you hope they grow quickly or you eventually terminate their employment because it’s a poor time trade off.

The trade off with an LLM is different. It’s not actually a junior or underperforming engineer. It’s far faster at churning out code than even the best engineers. It can read code far faster. It writes tests more consistently than most engineers (in my experience). It is surprisingly good at catching edge cases. With a junior engineer, you drag down your own performance to improve theirs and you’re often trading off short term benefits vs long term. With an LLM, your net performance goes up because it’s augmenting you with its own strengths.

As an engineer, it will never reach senior level (though future models might). But as a tool, it can enable you to do more.

> It writes tests more consistently than most engineers (in my experience)

I'm going to nit on this specifically. I firmly believe anyone that genuinely believes this either never writes tests that actually matter, or doesn't review the tests that an LLM throws out there. I've seen so many cases of people saying 'look at all these valid tests our LLM of choice wrote' only for half of them to do nothing and half of them misleading as to what it actually tests.

It’s like anything else, you’ve got to check the results and potentially push it to fix stuff.

I recently had AI code up a feature that was essentially text manipulation. There were existing tests to show it how to write effective tests and it did a great job of covering the new functionality. My feedback to the AI was mostly around some inaccurate comments it made in the code but the coverage was solid. Would have actually been faster for me to fix but I’m experimenting with how much I can make the AI do.

On the other hand I had AI code up another feature in a different code base and it produced a bunch of tests with little actual validation. It basically invoked the new functionality with a good spectrum of arguments but then just validated that the code didn’t throw. And in one case it tested something that diverged slightly from how the code would actually be invoked. In that case I told it how to validate what the functionality was actually doing and how to make the one test more representative. In the end it was good coverage with a small amount of work.

For people who don’t usually test or care bunch about testing, yeah, they probably let the AI create garbage tests.

I don't see anything here that corroborates your claim that it outputs more consistent test code than most engineers. In fact your second case would indicate otherwise.

And this also goes back to my first point about writing tests that matters. Coverage can matter, but coverage is not codifying business logic in your test suite. I've seen many engineers focus only on coverage only for their code to blow up in production because they didn't bother to test the actual real world scenarios it would be used in, which requires deep understanding of the full system.

I still feel like in most of these discussions the criticism of LLMs is that they are poor replacements for great engineers. Yeah. They are. LLMs are great tools for great engineers. They won’t replace good engineers and they won’t make shitty engineers good.

You can’t ask an LLM to autonomously write complex test suites. You have to guide it. But when AI creates a solid test suite with 20 minutes of prodding instead of 4 hours of hand coding, that’s a win. It doesn’t need to do everything alone to be useful.

> writing tests that matters

Yeah. So make sure it writes them. My experience so far is that it writes a decent set of tests with little prompting, honestly exceeding what I see a lot of engineers put together (lots of engineers suck at writing tests). With additional prompting it can make them great.

That seems like the kind of feature where the LLM would already have the domain knowledge needed to write reasonable tests, though. Similar to how it can vibe code a surprisingly complicated website or video game without much help, but probably not create a single component of a complex distributed system that will fit into an existing architecture, with exactly the correct behaviour based on some obscure domain knowledge that pretty much exists only in your company.

> probably not create a single component of a complex distributed system that will fit into an existing architecture, with exactly the correct behaviour based on some obscure domain knowledge that pretty much exists only in your company.

An LLM is not a principal engineer. It is a tool. If you try to use it to autonomously create complex systems, you are going to have a bad time. All of the respectable people hyping AI for coding are pretty clear that they have to direct it to get good results in custom domains or complex projects.

A principal engineer would also fail if you asked them to develop a component for your proprietary system with no information, but a principal engineer would be able to so their own deep discovery and design if they have the time and resources to do so. An AI needs you to do some of that.

I also find it hard to agree with that part. Perhaps it depends on what type of software you write, but in my experience finding good test cases is one of those things that often requires a deep level of domain knowledge. I haven’t had much luck making LLMs write interesting, non-trivial tests.

This has been my experience as well. So far, whenever I’ve been initially satisfied with the one shotted tests, when I had to go back to them I realized they needed to be reworked.

I guess everyone dealing with legacy software sees code as a cost factor. Being able to delete code is harder, but often more important than writing code.

Owning code requires you to maintain it. Finding out what parts of the code actual implement features and what parts are not needed anymore (or were never needed in the first place) is really hard. Since most of the time the requirements have never been documented and the authors have left or cannot remember. But not understanding what the code does removed all possibility to improve or modify it. This is how software dies.

Churning out code fast is a huge future liability. Management wants solutions fast and doesn't understand these long term costs. It is the same with all code generators: Short term gains, but long term maintainability issues.

Do you not write code? Is your code base frozen, or do you write code for new features and bug fixes?

The fact that AI can churn out code 1000x faster does not mean you should have it churn out 1000x more code. You might have a list of 20 critical features and it have time to implement 10. AI could let you get all 20 but shouldn’t mean you check in code for 1000 features you don’t even need.

I write code. On a good day perhaps 800-1000 "hand written" lines.

I have never actually thought about how much typing time this actually is. Perhaps an hour? In that case 7/8th of my day are filled with other stuff. Like analysis, planning, gathering requirements, talking to people.

So even if an AI removed almost all the time I spend typing away: This is only a 10% improvement in speed. Even if you ignore that I still have to review the code, understand everything and correct possible problems.

A bigger speedup is only possible if you decide not to understand everything the AI does and just trust it to do the right thing.

Maybe you code so fast that the thought-to-code transition is not a bottleneck for you. In which case, awesome for you. I suspect this makes you a significant outlier since respected and productive engineers like Antirez seem to find benefits.

Sure if you just leave all the code there. But if it's churning out iterations, incrementally improving stuff, it seems ok? That's pretty much what we do as humans, at least IME.

I feel like this is a forest for the trees kind of thing.

It is implied that the code being created is for “capabilities”. If your AI is churning out needless code, then sure, that’s a bad thing. Why would you be asking the AI for code you don’t need, though? You should be asking it for critical features, bug fixes, the things you would be coding up regardless.

You can use a hammer to break your own toes or you can use it to put a roof on your house. Using a tool poorly reflects on the craftsman, not the tool.

Just like LLMs are a total waste of time if you never update the system/developer prompts with additional information as you learn what's important to communicate vs not.

That is a completely different level. I expect a Junior Developer to be able to completely replace me long term and to be able decide when existing rules are outdated and when they should be replaced. Challenge my decisions without me asking for it. Being able to adapt what they have learned to new types of projects or new programming languages. Being Senior is setting the rules.

An LLM only follows rules/prompts. They can never become Senior.

I think you're making a mistake if your reviews are just that you trust that your co-workers never make a mistake. I make mistakes. My co-workers make mistakes. Everybody makes mistakes, that's why we have code reviews.

Yes. Firstly AI forgets why it wrote certain code and with humans at least you can ask them when reviewing. Secondly current gen AI(at least Claude) kind of wants to finish the thing instead of thinking of bigger picture. Human programmers code little differently that they hate a single line fix in random file to fix something else in different part of the code.

I think the second is part of RL training to optimize for self contained task like swe bench.

So you live in a world where code history must only be maintained orally? Have you ever thought to ask AI to write documentation on what and why and not just write the code. Asking it to document as well as code works well when the AI needs to go back and change either.

I don't see how asking AI to write some description of why it wrote this or that code would actually result in an explanation of why it wrote that code? It's not like it's thinking about it in that way, it's just generating both things. I guess they'd be in the same context so it might be somewhat correct.

If you ask it to document why it did something, then when it goes back later to update the code it has the why in its context. Otherwise, the AI just sees some code later and has no idea why it was written or what it does without reverse engineering it at the moment.

I'm not sure you understood the GP comment. LLMs don't know and can't tell you why they write certain things. You can't fix that by editing your prompt so it writes it on a comment instead of telling you. It will not put the "why" in the comment, and therefore the "why" won't be in the future LLM's context, because there is no way to make it output the "why".

It can output something that looks like the "why" and that's probably good enough in a large percentage of cases.

LLMs know why they are writing things in the moment, and they can justify decisions. Asking it to write those things down when it writes code works, or even asking them to design the code first and then generate/update code from the design also works. But yes, if things aren’t written down, “the LLM don’t know and can’t tell.” Don’t do that.

I'm going to second seanmcdirmid here, a quick trick is to have Claude write a "remaining.md" if you know you have to do something that will end the session.

Example from this morning, I have to recreate the EFI disk of one of my dev vm's, it means killing the session and rebooting the vm. I had Claude write itself a remaining.md to complement the overall build_guide.vm I'm using so I can pick up where I left off. It's surprisingly effective.

No, humans probably have tens of millions of token in memory of memory per PR. It includes not only what's in the code, but what all they searched, what all they tested and in which way, which order they worked on, the edge cases they faced etc. Claude just can't document all these, else it will run out of its working context pretty soon.

Ya, LLMs are not human level, they have smaller focus windows, but you can "remember" things with documentation, just like humans usually resort to when you realize that their tens of millions of token in memory per PR isn't reliable either.

The nice thing about LLMs, however, is that they don't grumble about writing extra documentation and tests like humans do. You just tell them to write lots of docs and they do it, they don't just do the fun coding part. I can empathize why human programmers feel threatened.

> It can output something that looks like the "why"

This feels like a distinction without difference. This is an extension of the common refrain that LLMs cannot “think”.

Rather than get overly philosophical, I would ask what the difference is in practical terms. If an LLM can write out a “why” and it is sufficient explanation for a human or a future LLM, how is that not a “why“?

If you're planning on throwing the code away, fine, but if you're not, eventually you're going to have to revisit it.

Say I'm chasing down some critical bug or a security issue. I run into something that looks overly complicated or unnecessary. Is it something a human did for a reason or did the LLM just randomly plop something in there?

I don't want a made up plausible answer, I need to know if this was a deliberate choice, forex "this is to work around an bug in XY library" or "this is here to guard against [security issue]" or if it's there because some dude on Stackoverflow wrote sample code in 2008.

If your concern is philosophical, and you are defining LLMs as not having a “why”, then of course they cannot write down “why” because it doesn’t exist. This is the philosophical discussion I am trying to avoid because I don’t think it’s fruitful.

If your concern is practical and you are worried that the “why” an LLM might produce is arbitrary, then my experience so far says this isn’t a problem. What I’m seeing LLMs record in commit messages and summaries of work is very much the concrete reasons they did things. I’ve yet to see a “why” that seemed like nonsense or arbitrary.

If you have engineers checking in overly complex blobs of code with no “why”, that’s a problem whether they use AI or not. AI tools do not replace engineers and I would not with in any code base where engineers were checking in vibe coded features without understanding them and vetting the results properly.

I don't care what text the LLM generates. If you wanna read robotext, knock yourself out. It's useless for what I'm talking about, which is "something is broken and I'm trying to figure out what"

In that context, I'm trying to do two things:

1. Fix the problem
2. Don't break anything else

If there's something weird in the code, I need to know if it's necessary. "Will I break something I don't know about if I change this" is something I can ask a person. Or a whole chain of people if I need to.

I can't ask the LLM, because "yes $BIG_CLIENT needs that behavior for stupid reasons" is not gonna be a part of its prompt or training data, and I need that information to fix it properly and not cause any regressions.

It may sound contrived but that sort of thing happens allllll the time.

> If there's something weird in the code, I need to know if it's necessary.

What does this have to do with LLMs?

I agree this sort of thing happens all the time. Today. With code written by humans. If you’re lucky you can go ask the human author, but in my experience if they didn’t bother to comment they usually can’t remember either. And very often the author has moved on anyway.

The fix for this is to write why this weird code is necessary in a comment or at least a commit message or PR summary. This is also the fix for LLM code. In the moment, when in the context for why this weird code was needed, record it.

You also should shame any engineer who checks in code they don’t understand, regardless of whether it came from an LLM or not. That’s just poor engineering and low standards.

Yeah. I know. The point is there is no Chesterson's Fence when it comes to LLMs. I can't even start from the assumption that this code is here for a reason.

And yes, of course people should understand the code. People should do a lot of things in theory. In practice, every codebase has bits that are duct taped together with a bunch of #FIXME comments lol. You deal with what you got.

The problem is that your starting point seems to be that LLMs can check in garbage to your code base with no human oversight.

If your engineering culture is such that an engineer could prompt an LLM to produce a bunch of code that contains a bunch of weird nonsense, and they can check that weird nonsense in with no comments and no will say “what the hell are you doing?”, then the LLM is not the problem. Your engineering culture is. There is no reason anyone should be checking in some obtuse code that solves BIG_CORP_PROBLEM without a comment to that effect, regardless of whether they used AI to generate the code or not.

Are you just arguing that LLM’s should not be allowed to check in code without human oversight? Because yeah, I one hundred percent agree and I think most people in favor of AI use for coding would also agree.

Yeah, and I'm explaining that the gap between theory and practice is greater in practice than it is in theory, and why LLMs make it worse.

It's easy to just say "just make the code better", but in reality I'm dealing with something that's an amalgam of the work of several hundred people, all the way back to the founders and whatever questionable choices they made lol.

The map is the territory here. Code is the result of our business processes and decisions and history.

You're treating this as a philosophical question like a LLM can't have actual reasons because it's not conscious. That's not the problem. No, the problem is mechanical. The processing path that would be needed to output actual reasons just doesn't exist.

LLMs only have one data path and that path basically computes what a human is most likely to write next. There's no way to make them not do this. If you ask it for a cake recipe it outputs what it thinks a human would say when asked for a fake recipe. If you ask it for a reason it called for 3 eggs, it outputs what it thinks a human would say when asked why they called for 3 eggs. It doesn't go backwards to the last checkpoint and do a variational analysis to see what factors actually caused it to write down 3 eggs. It just writes down some things that sound like reasons you'd use 3 eggs.

If you want to know the actual reasons it wrote 3 eggs, you can do that, but you need to write some special research software that metaphorically sticks the AI's brain full of electrodes. You can't do it by just asking the model because the model doesn't have access to that data.

Humans do the same thing by the way. We're terrible at knowing why we do things. Researchers stuck electrodes in our brains and discovered a signal that consistently appears about half a second before we're consciously aware we want to do something!

But this is exactly why it is philosophical. We’re having a discussion about why an LLM cannot really ever explain “why”. And then we turn around and say, but actually humans have the exact same problem. So it’s not an LLM problem at all. It’s a philosophical problem about whether it’s possible to identify a real “why”. In general it is not possible to distinguish between a “real why” and a post hoc rationalization so the distinction is meaningless for practical purposes.

It's absolutely not meaningless if you work on code that matters. It matters a lot.

I don't care about philosophical "knowing", I wanna make sure I'm not gonna cause an incident by ripping out or changing something or get paged because $BIG_CLIENT is furious that we broke their processes.

If I show you two "why" comments in a codebase, can you tell which one was written by an LLM and which was not?

Just like humans leave comments like this

// don't try to optimise this, it can't be done
// If you try, increment this number: 42

You can do the same for LLMs

// This is here because <reason> it cannot be optimised using <method>

It works, I've done it. (In the surface that code looks you can use a specific type of caching to speed it up, but it actually fails because of reasons - LLMs kept trying, I added a comment that stopped them).

Of course I can't tell the difference. That's not the point. And yes, humans can leave stupid comments too.

The difference is I can ping humans on Slack and get clarification.

I don't want reasons because I think comments are neat. If I'm tracking this sort of thing down, something is broken and I'm trying to fix it without breaking anything else.

It only takes screwing this up a couple times before you learn what a Chesterson's Fence is lol.

You are framing this as an AI problem, but from what I’m hearing, this is just an engineering culture problem.

You should not bet on the ability to ping humans on Slack long-term. Not because AI is going to replace human engineers, but because humans have fallible memories and leave jobs. To the extent that your processes require the ability to regularly ask other engineers “why the hell did you do this“, your processes are holding you back.

If anything, AI potentially makes this easier. Because it’s really easy to prompt the AI to record why the hell things are done the way they are, whether recording its own “thoughts” or recording the “why” it was given by an engineer.

It's not an engineering culture problem lol, I promise. I have over a decade in this career and I've worked at places with fantastic and rigorous processes and at places with awful ones. The better places slacked each other a lot.

I don't understand what's so hard to understand about "I need to understand the actual ramifications of my changes before I make them and no generated robotext is gonna tell me that"

StackOverflow is a tool. You could use it to look for a solution to a bug you're investigating. You could use it to learn new techniques. You could use it to guide you through tradeoffs in different options. You can also use it to copy/paste code you don't understand and break your production service. That's not a problem with StackOverflow.

> "I need to understand the actual ramifications of my changes before I make them and no generated robotext is gonna tell me that"

Who's checking in this robotext?

* Is it some rogue AI agent? Who gave it unfettered access to your codebase, and why?

* Is it you, using an LLM to try to fix a bug? Yeah, don't check it in if you don't understand what you got back or why.

* Is it your peers, checking in code they don't understand? Then you do have a culture problem.

An LLM gives you code. It doesn't free you of the responsibility to understand the code you check in. If the only way you can use an LLM is to blindly accept what it gives you, then yeah, I guess don't use an LLM. But then you also probably shouldn't use StackOverflow. Or anything else that might give you code you'd be tempted to check in blindly.

It does actually work incredibly well. It's even remarkably good at looking through existing stuff (written by AI or not) and reasoning about why it is the way it is. I agree it's not "thinking" in the same way a human might, but it gets to a more plausible explanation than many humans can a lot more often than I ever would have thought.

> So you live in a world where code history must only be maintained orally?

There are many companies and scenarios where this is completely legitimate.

For example, a startup that's iterating quickly with a small, skilled dev team. A bunch of documentation is a liability, it'll be stale before anyone ever reads it.

Just grabbing someone and collaborating with them on what they wrote is much more effective in that situation.

> For example, a startup that's iterating quickly with a small, skilled dev team. A bunch of documentation is a liability, it'll be stale before anyone ever reads it.

This is a huge advantage for AI though, they don't complain about writing docs, and will actively keep the docs in sync if you pipeline your requests to do something like "I want to change the code to do X, update the design docs, and then update the code". Human beings would just grumble a lot, an AI doesn't complain...it just does the work.

> Just grabbing someone and collaborating with them on what they wrote is much more effective in that situation.

Again, it just sounds to me that you are arguing why AIs are superior, not in how they are inferior.

Documentation isn't there to have and admire, you write it for a purpose.

There are like eight bajillion systems out there that can generate low-level javadoc-ish docs. Those are trivial.

The other types of internal developer documentation are "how do I set this up", "why was this code written" and "why is this code the way it is" and usually those are much more efficiently conveyed person to person. At least until you get to be a big company.

For a small team, I would 100% agree those kinds of documentation are usually a liability. The problem is "I can't trust that the documentation is accurate or complete" and with AI, I still can't trust that it wrote accurate or complete documentation, or that anyone checked what it generated. So it's kind of worse than useless?

The LLM writes it with the purpose you gave it, to remember why it did things when it goes to change things later. The difference between humans and AI is that humans skip the document step because they think they can just remember everything, AI doesn’t have that luxury.

Just say the model uses the files to seed token state. Anthropomorphizing the thing is silly.

And no, you don't skip the documentation because you "think you can just remember everything". It's a tradeoff.

Documentation is not free to maintain (no, not even the AI version) and bad or inaccurate documentation is worse than none, because it wastes everyone's time.

You build a mental map of how the code is structured and where to find what you need, and you build a mental model of how the system works. Understanding, not memorization.

When prod goes down you really don't wanna be faffing about going "hey Alexa, what's a database index".

Have you never had a situation where a question arose a year (or several) later that wasn’t addressed in the original documentation?

In particular IME the LLM generates a lot of documentation that explains what and not a lot of the why (or at least if it does it’s not reflecting underlying business decisions that prompted the change).

You can ask it to generate the why, even if it the agent isn’t doing that by default. At least you can ask it to encode how it is mapping your request to code, and to make sure that the original request is documented, so you can record why it did something at least, even if it can’t have insight into why you made the request in the first place. The same applies to successive changes.

I seriously don't remember why I wrote certain code two months ago. I have to read my code that I wrote two months ago to understand what I was doing and why. I don't remember every single line of code that I wrote and why. I guess I'm a stateless developer that way.

1. a detailed spec, result of your discussions with the agent about work, when it gets it you ask the agent to formalize it into docs

2. an extensive suite of tests to cover every angle; the tests are generated, but your have to ensure their quality, coverage and depth

I think, to make a metaphor, that specs are like the skeleton of the agent, tests are like the skin, while the agent itself is the muscle and cerebellum, and you are the PFC. Skeleton provides structure and decides how the joints fit, tests provide pain and feedback. The muscle is made more efficient between the two.

In short the new coding loop looks like: "spec -> code -> test, rinse and repeat"

Are you just generating code with the LLM? Ya, you are screwed. Are you generating documentation and tests and everything else to help to code live? Your options for maintenance go up. Now just replace “generate” with “maintain” and you are basically asking AI to make changes to a description at the top that then percolate to multiple artifacts being updated, only one happening to be the code itself, and the code updates multiple time as the AI checks tests and stuff.

I wish there were good guides on how to get the best out of LLMs. All of these tips about adding documentation etc seem very useful but I’ve never seen good guides on how to do this effectively or sustainably.

It is still the early days; everyone has their process, and a lot of the process is still ad hoc. It is an exciting time to be in the field though, before turn key solutions come we all get to be explorers.

Fair, but it would be interesting to see how people are implementing this “write the docs you need to do a better job” logic and putting it into use. I’m playing with this but would love to see someone’s success story. “I did X and now the code is better/its more token efficient/reviewers understand the changes/whatever.”

I just let the LLM write the docs it will read and I don't pay attention to them very much unless I need to debug a problem that it can't solve on its own. I just tell it what areas to focus on, it writes stuff that gets checked in but not really read by humans, it updates the docs when it things change before it changes the code, but can also review all the design stuff to when making code changes.

Sometimes I run into a problem that the LLM can't really handle yet, but I just break the problem up into more docs, tests, and code. So...that usually works, but I admit I'm move more slowly on those problems, and I'm not asking the LLM how to break the problem up yet (although I think we will get there soon).

Do you prompt for anything specific to record or does your prompt just contain something general like “read .aidump if present for potentially useful context and update or create .aidump with any useful information”?

Mostly the latter! You can ask it to look at things conditionally (like, if the test fails, look at this doc before deciding what to do next), but usually I just load it all up at the start before asking it to make change. The LLM is good enough about picking out what it needs. The one problem is that if you have a change you are propagating through the workflow, you need to highlight that change to the LLM or it might not notice it.

I'm working on workflow processing to make this easier ATM (because I can't help my coworkers do what I'm doing, and what I'm doing is so ad hoc), which is why I'm talking about it so much. So the idea is that you request a change at the top, and the LLM updates everything to accommodate the change, keeping track of what changed in each artifact. When it goes to generate code...it has a change for the artifacts that input into code (which are just read in along with a prompt saying "generate the code!"). You just don't ask the LLM to change the code directly (because if you do that, none of the docs get updated for the change, and things can go bad after that...).

When things go wrong, I add extra context if I can spot the problem ("focus on X, X looks wrong because...") and that just merges with the other docs as the context. Sometimes if I can't figure out why a test is failing, I ask it to create a simpler version of the test and see if that fails (if it does, it will be easier to eye the problem). Manual intervention is still necessary (and ugh, sometimes the LLM is just having a bad day and I need to /clear and try again).

I need to play with this more. I’ve had AI generate a bunch of small summaries that it could theoretically use to optimize future work. I haven’t asked it specifically to just dump info as it’s doing other work yet.

The files I had it generate were interesting but I’m not convinced looking at them that they contain the real info the AI needs to be more efficient. I should look into what kind of context analysis agents are passing back because that seems like what I want to save for later.

You can’t just ask AI to dump, you need to vaguely describe what design elements you think are important, like for SQL, you might want to plan our your CTEs first, then come up with a strategy for implementing each one, before getting to the SQL file itself (and of course tests, but that is a separate line of artifacts, you don’t want the AI to look at the tests when updating code, because you want to avoid letting AI code to the test). You can also look at where the AI having trouble doing something, or not doing it very well, and ask it to write documentation that will help it do that more successfully.

I can’t imagine asking AI to change some code without having a description of what the code does. You could maybe reverse engineer that, but that would basically be generating the documents after the fact. Likewise changing code without tests, where failing tests are actionable signals for the AI to make sure it doesn’t break things on update. Some people here think you can just ask it to write code without any other artifacts, thats nuts (maybe agentic will develop in the direction where AI writes persistent artifacts on its own without being told to do so, actually I’m sure that will happen eventually).

> You can’t just ask AI to dump, you need to vaguely describe what design elements you think are important

Right. And that’s what I’ve tried to do but I am not confident it’s captured the most critical info in an efficient way.

> I can’t imagine asking AI to change some code without having a description of what the code does. You could maybe reverse engineer that, but that would basically be generating the documents after the fact.

This is exactly how I’ve been using AI so far. I tell it to deeply analyze the code before starting and it burns huge amounts of tokens relearning the same things it learned last time. I want to get some docs in place to minimize this. That’s why I’m interested in what a subagent would respond with because that’s what it’s operating with usually. Or maybe the compressed context might be an interesting reference.

You can save the analysis and those are your docs. But your workflow has to maintain them in sync with the code.

I have no idea about token cost working for a FAANG, it’s a blind spot for me. One of these days I’m going to try to get QWen coder going for some personal projects on my M3 Max (I can run 30b or even 80b heavily quantized), and see if I can get something going that’s thrifty with the resources provided by a local LLM.

I’m not actually paying for tokens. Just trying to be a good citizen. And also trying to figure out how to set everyone in my organization up to do the same.

Interestingly while playing with Claude Code I just learned that /init actually does analyze and record findings.

Would it not be a new paradigm, where the generated code from AI is segregated and treated like a binary blob? You don't change it (beyond perhaps some cosmetic, or superficial changes that the AI missed). You keep the prompt(s), and maintain that instead. And for new changes you want added, the prompts are either modified, or appended to.

indeed - https://www.dbreunig.com/2026/01/08/a-software-library-with-... appears to be exactly that - the idea that the only leverage you have for fixing bugs is updating prompts (and, to be fair, test cases, which you should be doing for every bug anyway) is kind of upsetting as someone who thinks software can actually work :-)

There is a related issue of ownership. When human programmers make errors that cost revenue or worse, there is (in theory) a clear chain of accountability. Who do you blame if errors generated by LLMs end up in mission critical software?

> Who do you blame if errors generated by LLMs end up in mission critical software?

I don't think many companies/codebases allow LLMs to autonomously edit code and deploy it, there is still a human in the loop that "prompt > generates > reviews > commits", so it really isn't hard to find someone to blame for those errors, if you happen to work in that kind of blame-filled environment.

Same goes with contractors I suppose, if you end up outsourcing work to a contractor, they do a shitty job but that got shipped anyways, who do you blame? Replace "contractor" with "LLM" and I think the answer remains the same.

I have AI agents write, perform code review, improve and iterate upon the code. I trust that an agent with capabilities to write working code can also improve it. I use Claude skills for this and keep improving the skills based on both AI and human code reviews for the same type of code.

> programmer who actually do like the actual typing

It's not about the typing, it's about the understanding.

LLM coding is like reading a math textbook without trying to solve any of the problems. You get an overview, you get a sense of what it's about and most importantly you get a false sense of understanding.

But if you try to actually solve the problems, you engage completely different parts of your brain. It's about the self-improvement.

> LLM coding is like reading a math textbook without trying to solve any of the problems.

Most math textbooks provide the solutions too. So you could choose to just read those and move on and you’d have achieved much less. The same is true with coding. Just because LLMs are available doesn’t mean you have to use them for all coding, especially when the goal is to learn foundational knowledge. I still believe there’s a need for humans to learn much of the same foundational knowledge as before LLMs otherwise we’ll end up with a world of technology that is totally inscrutable. Those who choose to just vibe code everything will make themselves irrelevant quickly.

I haven't used AI yet but I definitely would love a tool that could do the drudgery for me for designs that I already understand. For instance, if I want to store my own structures in an RDBMS, I want to lay the groundwork and say "Hey Jeeves, give me the C++ syntax to commit this structure to a MySQL table using commit/rollback". I believe once I know what I want, futzing over the exact syntax for how to do it is a waste of time. I heard c++ isn't well supported but eventually I'll give it a try.

The ones I used for the first couple of years of my math PhD had solutions. That's a sufficient level of "advanced" to be applicable in this analogy. It doesn't really matter though - the point still stands that _if_ solutions are available you don't have to use them and doing so will hurt your learning of foundational knowledge.

> It's not about the typing, it's about the understanding.

Well, it's both, for different people, seemingly :)

I also like the understanding and solving something difficult, that rewards a really strong part of my brain. But I don't always like to spend 5 hours in doing so, especially when I'm doing that because of some other problem I want to solve. Then I just want it solved ideally.

But then other days I engage in problems that are hard because they are hard, and because I want to spend 5 hours thinking about, designing the perfect solution for it and so on.

Different moments call for different methods, and particularly people seem to widely favor different methods too, which makes sense.

> LLM coding is like reading a math textbook without trying to solve any of the problems. You get an overview, you get a sense of what it's about and most importantly you get a false sense of understanding.

Can be, but… well, the analogy can go wrong both ways.

This is what Brilliant.org and Duolingo sell themselves on: solve problems to learn.

Before I moved to Berlin in 2018, I had turned the whole Duolingo German tree gold more than once, when I arrived I was essentially tourist-level.

Brilliant.org, I did as much as I could before the questions got too hard (latter half of group theory, relativity, vector calculus, that kind of thing); I've looked at it again since then, and get the impressions the new questions they added were the same kind of thing that ultimately turned me off Duolingo, easier questions that teach little, padding out a progressions system that can only be worked through fast enough to learn anything if you pay a lot.

Code… even before LLMs, I've seen and I've worked with confident people with a false sense of understanding about the code they wrote. (Unfortunately for me, one of my weaknesses is the politics of navigating such people).

Yeah, there's a big difference between edutainment like Brilliant and Duolingo and actually studying a topic.

I'm not trying to be snobbish here, it's completely fine to enjoy those sorts of products (I consume a lot of pop science, which I put in the same category) but you gotta actually get your hands dirty and do the work.

It's also fine to not want to do that -- I love to doodle and have a reasonable eye for drawing, but to get really good at it, I'd have to practice a lot and develop better technique and skills and make a lot of shitty art and ehhhh. I don't want it badly enough.

Lately I've been writing DSLs with the help of these LLM assistants. It is definitely not vibe coding as I'm paying a lot of attention to the overall architecture. But most importantly my focus is on the expressiveness and usefulness of the DSLs themselves. I am indeed solving problems and I am very engaged but it is a very different focus. "How can the LSP help orient the developer?" "Do we want to encourage a functional-looking pipeline in this context"? "How should the step debugger operate under these conditions"? etc.

We've been hearing this a lot, but I don't really get it. A lot of code, most probably, isn't even close to being as challenging as a maths textbook.

It obviously depends a lot on what exactly you're building, but in many projects programming entails a lot of low intellectual effort, repetitive work.

It's the same things over and over with slight variations and little intellectual challenge once you've learnt the basic concepts.

Many projects do have a kernel of non-obvious innovation, some have a lot of it, and by all means, do think deeply about these parts. That's your job.

But if an LLM can do the clerical work for you? What's not to celebrate about that?

To make it concrete with an example: the other day I had Claude make a TUI for a data processing library I made. It's a bunch of rather tedious boilerplate.

I really have no intellectual interest in TUI coding and I would consider doing that myself a terrible use of my time considering all the other things I could be doing.

The alternative wasn't to have a much better TUI, but to not have any.

> It obviously depends a lot on what exactly you're building, but in many projects programming entails a lot of low intellectual effort, repetitive work.

I think I can reasonably describe myself as one of the people telling you the thing you don't really get.

And from my perspective: we hate those projects and only do them if/because they pay well.

> the other day I had Claude make a TUI for a data processing library I made. It's a bunch of rather tedious boilerplate. I really have no intellectual interest in TUI coding...

From my perspective, the core concepts in a TUI event loop are cool, and making one only involves boilerplate insofar as the support libraries you use expect it. And when I encounter that, I naturally add "design a better API for this" to my project list.

Historically, a large part of avoiding the tedium has been making a clearer separation between the expressive code-like things and the repetitive data-like things, to the point where the data-like things can be purely automated or outsourced. AI feels weird because it blurs the line of what can or cannot be automated, at the expense of determinism.

I've also been hearing variations of your comment a lot too and correct me if I am wrong but I think they always implicitly assume that LLMs are more useful for the low-intellectual stuff than solving the high-intellectual core of the problem.

The thing is:

1) A lot of the low-intellectual stuff is not necessarily repetitive, it involved some business logic which is a culmination of knowing the process behind what the uses needs. When you write a prompt, the model makes assumptions which are not necessarily correct for the particular situation. Writing the code yourself forced you to notice the decision points and make more informed choices.

I understand your TUI example and it's better than having none now, but as a result anybody who wants to write "a much better TUI" now faces a higher barrier to entry since a) it's harder to justify an incremental improvement which takes a lot of work b) users will already have processes around the current system c) anybody who wrote a similar library with a better TUI is now competing with you and quality is a much smaller factor than hype/awareness/advertisement.

We'll basically have more but lower quality SW and I am not sure that's an improvement long term.

2) A lot of the high-intellectual stuff ironically can be solved by LLMs because a similar problem is already in the training data, maybe in another language, maybe with slight differences which can be pattern matched by the LLM. It's laundering other people's work and you don't even get to focus on the interesting parts.

> but I think they always implicitly assume that LLMs are more useful for the low-intellectual stuff than solving the high-intellectual core of the problem.

Yes, this follows from the point the GP was making.

The LLM can produce code for complex problems, but that doesn't save you as much time, because in those cases typing it out isn't the bottleneck, understanding it in detail is.

And so in the future if you want to add a feature, either the LLM can do it correctly or the feature doesn’t get added? How long will that work as the TUI code base grows?

At that point you change your attitude to the project and start treating it like something you care about, take control of the architecture, rewrite bits that don't make sense, etc.

Plus the size of project that an LLM can help maintain keeps growing. I actually think that size may no longer have any realistic limits at all now: the tricks Claude Code uses today with grep and sub-agents mean there's no longer a realistic upper limit to how much code it can help manage, even with Opus's relatively small (by today's standards) 200,000 token limit.

The problem I'm anticipating isn't so much "the codebase grows beyond the agent-system's comprehension" so much as "the agent-system doesn't care about good architecture" (at least unless it's explicitly directed to). So the codebase grows beyond the codebase's natural size when things are redundantly rewritten and stuffed into inappropriate places, or ill-fitting architectural patterns are aped.

> I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer, and for them, I understand LLMs remove the fun part.

One of the reasons why I learned vim was because I enjoy staying in the keyboard; I'm a fast typer and part of the fun is typing out the code I'm thinking.

I can see how some folks only really like seeing the final product rather than the process of building it but I'm just not cut for that — I hate entrepreneurship for the same reason, I enjoy the building part more than the end.

And it's the part that's killing me with all this hype.

Conversely I have very little interest in the process of programming by itself, all the magic is about the end result and the business value for me (which fortunately has served me quite well professionally). As young as I remember I was fascinated with the GUI DBMS (4th Dimension/FileMaker/MS Access/…) my dad used to improve his small business. I only got into programming only to not be limited by graphical tools. So LLMs for me are just a nice addition in my toolbox, like a power tool is to a manual one. It doesn’t philosophically changes anything.

It resonates. But as I see it, that kind of ritual I rather devote myself to at home. At work, the more efficient and rapidly we can get stuff dobe, the better.

Drawing and painting is a ritual to me as well. No one pays me for it and I am happy about that.

You might be surprised to find out how much of your motivation to do any of it at all was tied to your enjoyment, and that’s much more difficult to overcome than people realize.

> Now the fun is gone, maybe I can do more important work.

This is a very sad, bleak, and utilitarian view of "work." It is also simply not how humans operate. Even if you only care about the product, humans that enjoy and take pride in what they're doing almost invariably produce better products that their customers like more.

My problem was the exact opposite. I wanted to deliver but the dislike of the actual programming / typing code prevented me from doing so. AI has solved this for me.

> For me, I initially got into programming because I wanted to ruin other people's websites, then I figured out I needed to know how to build websites first, then I found it more fun to create and share what I've done with others, and they tell me what they think of it.

Talk about a good thing coming from bad intentions! Congratulations on shaking that demon.

1. Those who see their codebase as a sculpture, a work of art, a source of pride
2. Those who focus on outcomes.

They are not contradictory goals, but I'm finding that if your emphasis is 1, you general dislike LLMs, and if your emphasis is 2, you love them, or at least tolerate them.

I don't disagree with you - LLMs are not at odds with quality code if you use them correctly. But many people who take excessive pride in their code don't even bother to look and see what can be done with them. Though, in the last couple months, I have seen several of the (1) types around me finally try them.

I think it’s true that people get enjoyment from different things. Also, I wonder if people have fixed ideas about how coding agents can be used? For example, if you care about what the code looks like and want to work on readability, test coverage, and other “code health” tasks with a coding agent, you can do that. It’s up to you whether you ask it to do cleanup tasks or implement new features.

Maybe there are people who are about literally typing the code, but I get satisfaction from making the codebase nice and neat, and now I have power tools. I am just working on small personal projects, but so far, Claude Opus 4.5 can do any refactoring I can describe.

> I think there is a section of programmer who actually do like the actual typing of letters

Do people actually spend a significant time typing? After I moved beyond the novice stage it’s been an inconsequential amount of time. What it still serves is a thorough review of every single line in a way that is essentially equivalent to what a good PR review looks like.

> I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer, and for them, I understand LLMs remove the fun part.

I know you didn't mean to, but I think that description is a mischaracterization. I'd wager most of us "I control the computer" people who enjoy crafting software don't really care for the actual imputation of symbols. That is just the mechanism by which we move code from our heads to the computer. What LLMs destroy – at least for me – is the creation of code in my head and its (more-or-less) faithful replication inside the computer. I don't particularly enjoy the physical act of moving my fingers across a piece of plastic, but I do enjoy the result executing my program on my computer.

If an LLM is placed in the middle, two things happen: first, I'm expressing the _idea_ of my program not to a computer, but to an LLM; and second, the LLM expresses its "interpretation" of that idea to the computer. Both parts destroy joy for me. That's of course not important to anyone but myself and likeminded people, and I don't expect the world to care. But I do also believe that both parts come with a whole host of dangers that make the end result less trustworthy and less maintainable over time.

I'm definitely warming to the role of LLMs as critics though. I also see value in having them write tests – the worst a bad or unmaintainable test will provide is a false error.

> I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer.

I don't think this is really it for many people (maybe any); after all, you can do all of that when writing a text message rather than a piece of code.

But it inches closer to what I think is the "right answer" for this type of software developer. There are aspects of software development that are very much like other forms of writing (e.g., prose or poetry).

Like other writing, writing code can constitute self-expression in an inherently satisfying way, and it can also offer the satisfaction of finding "the perfect phrase". LLMs more or less eliminate both sources of pleasure, either by eliminating the act of writing itself (that is, choosing and refining the words) or through their bland, generic, tasteless style.

There are other ways that LLMs can disconnect the people using them from what is joyful about writing code, not least of all because LLMs can be used in a lot of different ways. (Using them as search tools or otherwise consulting them rather than having them commit code to simply be either accepted/rejected "solves" the specific problems I just mentioned, for instance.)

There is something magical about speaking motion into existence, which is part of what has made programming feel special to me, ever since I was a kid. In a way, prompting an LLM to generate working code preserves that and I can imagine how, for some, it even seems to magnify the magic. But there is also a sense of essential mastery involved in the wonderful way code brings ideas to life. That mastery involves not just "understanding" things in the cursory way involved in visually scanning someone else's code and thinking "looks good to me", but intimately knowing how the words and abstractions and effects all "line up" and relate to each other (and hopefully also with the project's requirements). That feeling of mastery is itself one of the joys of writing code.

Without that mastery, you also lose one of the second-order joys of writing code that many here have already mentioned in these comments: flow. Delegation means fumbling in a way that working in your own context just doesn't. :-\

I think all programmers are like LEGO builders. But different programmers will see each brick as a different kind of abstraction. A hacker kind of programmer may see each line of code as a brick. An architect kind of programmer may see different services as a brick. An entrepreneur kind of programmer may see entire applications as a brick. These aren't mutually exclusive, of course. But we all just like to build things, the abstractions we use to build them just differ.

This is exactly the way I see it. You can always get better performance at lower levels of abstraction, but there are trade-offs. Sometimes the trade-offs are worth it (like building bigger things), and sometimes they aren't (it's a buggy mess).

Indeed. My response was: actually, no, if I think about it I really don't think it was "building" at all. I would have started fewer things, and seen them through more consistently, if it were about "building". I think it has far more to do with personal expression.

("Solving a problem for others" also resonates, but I think I implement that more by tutoring and mentoring.)

> … not all programmers program for the same reason, for some of us, LLMs helps a lot, and makes things even more fun. For others, LLMs remove the core part of what makes programming fun for them. Hence we get this constant back and forth of "Can't believe others can work like this!" vs "I can't believe others aren't working like this!", but both sides seems to completely miss the other side.

Unfortunately the job market does not demand both types of programmer equally: Those who drive LLMs to deliver more/better/faster/cheaper are in far greater demand right now. (My observation is that a decade of ZIRP-driven easy hiring paused the natural business cycle of trying to do more with fewer employees, and we’ve been seeing an outsized correction for the past few years, accelerated by LLM uptake.)

> Unfortunately the job market does not demand both types of programmer equally: Those who drive LLMs to deliver more/better/faster/cheaper are in far greater demand right now.

I doubt that the LLM drivers deliver something better; quite the opposite. But I guess managers will only realize this when it's too late: and of course they won't take any responsibility for this.

> I doubt that the LLM drivers deliver something better…

That is your definition of “better”. If we’re going to trade our expertise for coin, we must ask ourselves if the cost of “better” is worth it to the buyer. Can they see the difference? Do they care?

(for the HN readers: a related concept is "information asymmetry in markets").

George Akerlof (the author of this paper), Michael Spence and Joseph Stiglitz got a Nobel Memorial Prize in Economic Sciences in 2001 for their analyses of markets with asymmetric information.

Good points. I'm a 'solve the problem' person, so rarely get into language wars, editor wars, etc... I just don't care as long as the problem is solved in a way that meets the needs of the user.

I've worked with all the types, and no type is wrong. For example, I can certainly appreciate the PL researcher type who wants to make everything functional, etc... I won't fight against it as long as it doesn't get in the way of solving the problem. I've also found that my style works well with the other styles because I have way of always asking "so does this solve the problem??" which is sometimes forgotten by the code is beautiful people, etc...

I’m better at code than prose, so coding via an agent is frustrating. Rather than multiple attempts to achieve the desired results, I’d rather just write in once, with the precision and nuance that I want. I’d be interested to try a “dueling pianos” style approach where I can cooperate with an agent indirectly through the code, rather than a lower fidelity option.

Who’s saying you can’t enjoy the typing of letters, numbers, and symbols into a computer? The issue is that this is getting to be a less economically valuable activity.

You wouldn’t say, “It’s not that they hate electricity it’s just that they love harpooning whales and dying in the icy North Atlantic.”

You can love it all you want but people won’t pay you to do it like they used to in the good old days.

> For others, LLMs remove the core part of what makes programming fun for them.

Anecdotally, I’ve had a few coworkers go from putting themselves firmly in this category to saying “this is the most fun I’ve ever had in my career” in the last two months. The recent improvement in models and coding agents (Claude Code with Opus 4.5 in our case) is changing a lot of minds.

Yeah, I'd put myself in this camp. My trust is slowly going up, and coupled with improved guardrails (more tests, static analysis, refactoring to make reviewing easier), that increasing trust is giving me more and more speed at going from thought ("hmm, I should change how this feature works to be like X") to deployment into the hands of my customers.

For me it's not the typing that is satisfying, but rather building the program in my head first, and internally validating. Describing to an LLM how to output the program I built in my head just isn't possible without taking more time than it does to write the code myself.

Almost more importantly is: the people who pay you to build software, don’t care if you type or enjoy it, they pay you for an output of working software

Literally nothing is stopping people from writing assembly in their free time for fun

But the number of people who are getting paid to write assembly is probably less than 1000

yep theres all types of people. i get hung up on the structure and shape of a source file, like its a piece of art. if it looks ugly, even if it works, i dont like it. ive seen some llm code that i like the shape of but i wouldnt like to use it verbatim since i didnt create it.

LLMs do empower you (and by "you" I mean the reader or any other person from now on) to actually complete projects you need in the very limited free time and have available. Manually coding the same could take months (I'm speaking from experience developing a personal project for about 3 hours every Friday and there's still much to be done). In a professional context, you're being paid to ship and AI can help you grow an idea to an MVP and then to a full implementation in record-breaking time. At the end of the day, you're satisfied because you built something useful and helped your company. You probably also used your problem solving skills.

Programming is also a hobby though. The whole process matters too. I'm one of the people who feels incredible joy when achieving a goal, knowing that I completed every step in the process with my own knowledge and skills. I know that I went from an idea to a complete design based on everything I know and probably learned a few new things too. I typed the variable names, I worked hard on the project for a long time and I'm finally seeing the fruits of my effort. I proudly share it with other people who may need the same and can attest its high quality (or low quality if it was a stupid script I hastily threw together, but anyway sharing is caring —the point is that I actually know what I've written).

The experience of writing that same code with an LLM will leave you feeling a bit empty. You're happy with the result: it does everything you wanted and you can easily extend it when you feel like it. But you didn't write the code, someone else did. You just reviewed an intern's work and gave feedback. Sometimes that's indeed what you want. You may need a tool for your job or your daily life, but you aren't too interested in the internals. AI is truly great for that.

I can't reach a better conclusion than the parent comment, everyone is unique and enjoys coding in a different way. You should always find a chance to code the way you want, it'll help maintain your self-esteem and make your life interesting. Don't be afraid of new technologies where they can help you though.

For me its the feeling of true understanding and discovery. Not just of how the computer works, but how whatever problem domain I'm making software for works. It's model building and simulation of the world. To the degree I can use the LLM to teach me to solve the problem better than I could before I like it, to the degree it takes over and obscures the understanding from me, I despise it. I don't love computers because of how fast I can create shareholder value, that's for sure.

> I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer, and for them, I understand LLMs remove the fun part.

I've "vibe coded" a ton of stuff and so I'm pretty bullish on LLMs, but I don't see a world where "coding by hand" isn't still required for at least some subset of software. I don't know what that subset will be, but I'm convinced it will exist, and so there will be ample opportunities for programmers who like that sort of thing.

---

Why am I convinced hand-coding won't go away? Well, technically I lied, I have no idea what the future holds. However, it seems to me that an AI which could code literally anything under the sun would almost by definition be that mythical AGI. It would need to have an almost perfect understanding of human language and the larger world.

An AI like that wouldn't just be great at coding, it would be great at everything! It would be the end of the economy, and scarcity. In which case, you could still program by hand all you wanted because you wouldn't need to work for a living, so do whatever brings you joy.

So even without making predictions about what the limitations of AI will ultimately be, it seems to me you'll be able to keep programming by hand regardless.

That can trim costs but not drive it to zero. If you assume that the computer is going to do all the work, won't your salary erode, making it harder for you to afford scarce things?

> For some people, the "fire" is literally about "I control a computer", for others "I'm solving a problem for others", and yet for others "I made something that made others smile/cry/feel emotions" and so on.

For the latter two, that's a minimum-wage job when LLMs produce your software, if that.

This article is not about whether programming is fun, elegant, creative, or personally fulfilling.

It is about business value.

Programming exists, at scale, because it produces economic value. That value translates into revenue, leverage, competitive advantage, and ultimately money. For decades, a large portion of that value could only be produced by human labor. Now, increasingly, it cannot be assumed that this will remain true.

Because programming is a direct generator of business value, it has also become the backbone of many people’s livelihoods. Mortgages, families, social status, and long term security are tied to it. When a skill reliably converts into income, it stops being just a skill. It becomes a profession. And professions tend to become identities.

People do not merely say “I write code.” They say “I am a software engineer,” in the same way someone says “I am a pilot” or “I am a police officer.” The identity is not accidental. Programming is culturally associated with intelligence, problem solving, and exclusivity. It has historically rewarded those who mastered it with both money and prestige. That combination makes identity attachment not just likely but inevitable.

Once identity is involved, objectivity collapses.

The core of the anti AI movement is not technical skepticism. It is not concern about correctness, safety, or limitations. Those arguments are surface rationalizations. The real driver is identity threat.

LLMs are not merely automating tasks. They are encroaching on the very thing many people have used to define their worth. A machine that can write code, reason about systems, and generate solutions challenges the implicit belief that “this thing makes me special, irreplaceable, and valuable.” That is an existential threat, not a technical one.

When identity is threatened, people do not reason. They defend. They minimize. They selectively focus on flaws. They move goalposts. They cling to outdated benchmarks and demand perfection where none was previously required. This is not unique to programmers. It is a universal human response to displacement.

The loudest opponents of AI are not the weakest programmers. They are often the ones most deeply invested in the idea of being a programmer. The ones whose self concept, status, and narrative of personal merit are tightly coupled to the belief that what they do cannot be replicated by a machine.

That is why the discourse feels so dishonest. It is not actually about whether LLMs are good at programming today. It is about resisting a trend line that points toward a future where the economic value of programming is increasingly detached from human identity.

This is not a moral failing. It is a psychological one. But pretending it is something else only delays adaptation.

AI is not attacking programming. It is attacking the assumption that a lucrative skill entitles its holder to permanence. The resistance is not to the technology itself, but to the loss of a story people tell themselves about who they are and why they matter.

That is the real conflict. HN is littered with people facing this conflict.

This is because they have entrenched themselves in a comfortable position that they don’t want to give up.

Most won’t admit this to be the actual reason.
Think about it: you are a normal hands on self thought software developer. You grew up tinkering with Linux and a bit of hardware. You realise there’s good money to be made in a software career. You do it for 20-30 years; mostly the same stuff over and over again. Some Linux, c#, networking. Your life and hobby revolves around these technologies. And most importantly you have a comfortable and stable income that entrenches your class and status. Anything that can disrupt this state is obviously not desireable. Never mind that disrupting others careers is why you have a career in the first place.

> disrupting others careers is why you have a career in the first place.

Not every software project has or did this. In fact I would argue many new businesses exist that didn't exist before software and computing and people are doing things they didn't beforehand. Especially around discovery of information - solving the "I don't know what I don't know" problem also expanded markets and demand to people who now know.

Whereas the current AI wave seems to be more about efficiency/industrialization/democratizing of existing use cases rather than novel things to date. I would be more excited if I saw more "product orientated" AI use cases other than destroying jobs. While I'm hoping that the "vibing" of software will mean that SWE's are needed to productionise it I'm not confident that AI won't be able to do that soon too nor any other knowledge profession.

I wouldn't be surprised with AI if there's mass unemployment but we still don't cure cancer for example in 20 years.

> Not every software project has or did this. In fact I would argue many new businesses exist that didn't exist before software and computing and people are doing things they didn't beforehand.

That's exactly what I am hoping to see happen with AI.

All I can say to that is "I hope so too"; but logic is telling me otherwise at this point. Because the alternative, as evidenced by this thread, isn't all that good. The fear/dread in people since the holidays has been sad to see - its overwhelmed everything else in tech now.

Sure; I absolutely agree and more to the point SWE's and their ideologies compared to other professions have meant they are the first on the chopping block. But what do you tell those people; that they no longer matter? Do they still matter? How will they matter? They are no different than practitioners of any other craft - humans in general derive value partly from the value they can give to their fellow man.

If the local unskilled job matters more than a SWE now these people have gone from being worth something to society to being less of worth than someone unskilled with a job. At that point following from your logic I can assume their long term value is one of an unemployed person which to some people is negative. That isn't just an identity crash; its a crash potentially on their whole lives and livelihood. Even smart people can be in situations where it is hard to pivot (as you say mortgages, families, lives, etc).

I'm sure many of the SWE's here (myself included) are asking the same questions; and the answers are too pessimistic to admit public ally and even privately. Myself the joy of coding is taken away with AI in general, in that there is no joy doing something that a machine will be able to do better soon for me at least.

I agree with you that the implications are bleak. For many people they are not abstract or philosophical. They are about income, stability, and the ability to keep a life intact. In that sense the fear is completely rational.

What stands out to me is that there seems to be a threshold where reality itself becomes too pessimistic to consciously accept.

At that point people do not argue with conclusions. They argue with perception.

You can watch the systems work. You can see code being written, bugs being fixed, entire workflows compressed. You can see the improvement curve. None of this is hidden. And yet people will look straight at it and insist it does not count, that it is fake, that it is toy output, that it will never matter in the real world. Not because the evidence is weak, but because the implications are unbearable.

That is the part that feels almost surreal. It is not ignorance. It is not lack of intelligence. It is the mind refusing to integrate a fact because the downstream consequences are too negative to live with. The pessimism is not in the claim. It is in the reality itself.

Humans do this all the time. When an update threatens identity, livelihood, or future security, self deception becomes a survival mechanism. We selectively ignore what we see. We raise the bar retroactively. We convince ourselves that obvious trend lines somehow stop right before they reach us. This is not accidental. It is protective.

What makes it unsettling is seeing it happen while the evidence is actively running in front of us. You are holding reality in one hand and watching people try to look away without admitting they are looking away. They are not saying “this is scary and I do not know how to cope.” They are saying “this is not real,” because that is easier.

So yes, the questions you raise are the real ones. Do people still matter. How will they matter. What happens when economic value shifts faster than lives can adapt. Those questions are heavy, and I do not think anyone has clean answers yet.

But pretending the shift is not happening does not make the answers kinder. It just postpones the reckoning.

The disturbing thing is not that reality is pessimistic. It is that at some point reality becomes so pessimistic that people start editing their own perception of it. They unsee what is happening in order to preserve who they think they are.

That is the collision we are watching. And it is far stranger than a technical debate about code quality.

Isn't this common on HN? People with subjective opinions voice their subjective opinions confidently. People who disagree calmly state they disagree and also state why.

The question is more about why my post triggered you... why would my simple opinion trigger you? Does disagreement trigger you? If I said something that is obviously untrue that you disagreed with, for example: "The world is flat." Would this trigger you? I don't think it would. So why was my post different?

Maybe this is more of a question you should ask yourself.

I have been reading over your comments for nearly two hours now. I find your writing suspiciously sterile, and I have trouble understanding how any one person could produce so much of it in such a small space of time.

The content is overwhelmingly compelling, and I think that any thinking person would have difficulty disagreeing with you.

I begin to wonder, cynically, that someone might enjoy presenting such a devastating, radically destabilizing picture.

Do you know how this thing works, that you can confidently claim, or even dare plant the seed in someone else's mind, that replacement is inevitable?

Excellent comment (even "mini essay"). I'm unsure if you've written it with AI-assistance, but even if that's the case, I'll tolerate it.

I have two things to add.

> This is not a moral failing. It is a psychological one.

(1) I disagree: it's not a failing at all. Resisting displacement, resisting that your identity, existence, meaning found in work, be taken away from you, is not a failing.

Such resistance might be futile, yes; but that doesn't make it a failing. If said resistance won, then nobody would call it a failing.

The new technology might just win, and not adapting to that reality, refusing that reality, could perhaps be called a failing. But it's also a choice.

For example, if software engineering becomes a role to review AI slop all day, then it simply devolves, for me, into just another job that may be lucrative but has zero interest for me.

(2) You emphasize identity. I propose a different angle: meaning, and intrinsic motivation. You mention:

> economic value of programming is increasingly detached from human identity

I want to rephrase it: what has been meaningful to me thus far remains meaningful, but it no longer allows me to make ends meet, because my tribe no longer appreciates when I act out said activity that is so meaningful to me.

THAT is the real tragedy. Not the loss of identity -- which you seem to derive from the combination of money and prestige (BTW, I don't fully dismiss that idea). Those are extrinsic motivations. It's the sudden unsustainability of a core, defining activity that remains meaningful.

The whole point of all these AI-apologist articles is that "it has happened in the past, time and again; humanity has always adapted, and we're now better off for it". Never mind those generations that got walked over and fell victim to the revolution of the day.

In other words, the AI-apologists say, "don't worry, you'll either starve (which is fine, it has happened time and agani), or just lose a large chunk of meaning in your life".

I think where we actually converge is on the phenomenon itself rather than on any moral judgment about it.

What I was trying to point at is how strange it is to watch this happen in real time. You can see something unfolding directly in front of you. You can observe systems improving, replacing workflows, changing incentives. None of it is abstract. And yet the implications of what is happening are so negative for some people that the mind simply refuses to integrate them. It is not that the facts are unknown. It is that the outcome is psychologically intolerable.

At that point something unusual happens. People do not argue with conclusions, they argue with perception. They insist the thing they are watching is not really happening, or that it does not count, or that it will somehow stop before it matters. It is not a failure of intelligence or ethics. It is a human coping mechanism when reality threatens meaning, livelihood, or future stability.

Meaning and intrinsic motivation absolutely matter here. The tragedy is not that meaningful work suddenly becomes meaningless. It is that it can remain meaningful while becoming economically unsustainable. That combination is brutal. But denying the shift does not preserve meaning. It only delays the moment where a person has to decide how to respond.

What I find unsettling is not the fear or the resistance. It is watching people stand next to you, looking at the same evidence, and then effectively unsee it because accepting it would force a reckoning they are not ready for.

>I'm unsure if you've written it with AI-assistance, but even if that's the case, I'll tolerate it.

Even if it was, the world is changing. You already need to tolerate AI in code, it's inevitable AI will be part of writing.

> the outcome is psychologically intolerable [...] People do not argue with conclusions, they argue with perception [...] accepting it would force a reckoning they are not ready for

"denying the shift does not preserve sustainability"

as "meaning" need not be preserved by anything. The idea here is that meaning -- stemming from the profession being supplanted -- is axiomatic.

And with that correction applied, I agree -- to an extent anyway. I hope that, even if (or "when") the mainstream gets swayed by AI, pockets / niches of "hand-crafting" remain sustainable. We've seen this with other professions that used to be mainstream but have been automated away at large scale.

People who love thinking in false dichotomies like this one have absolutely no idea how much harder it is to “get paid for doing commercial/trendy art”.

It’s so easy to be a starving artist; and in the world of commercial art it’s bloody dog-eat-dog jungle, not made for faint-hearted sissies.

I've given this quite some thought and came to the conclusion that there is actually no choice, and all parties fall into the first category. It's just that some people intrinsically like working on commercial themes, or happen to be trendy.

Of course there are some artists who sit comfortably in the grey area between the two oppositions, and for these a little nudging towards either might influence things. But for most artists, their ideas or techniques are simply not relevant to a larger audience.

> and all parties fall into the first category [...] Of course there are some artists who sit comfortably in the grey area between the two oppositions

I'm not sure what your background is, but there are definitly artists out there drawing, painting and creating art they have absolutely zero care for, or even actively is against or don't like, but they do it anyways because it's easier to actually get paid doing those things, than others.

Take a look in the current internet art community and ask how many artists are actively liking the situation of most of their art commissions being "furry lewd art", vs how many commissions they get for that specific niche, as just one example.

History has lots of other examples, where artists typically have a day-job of "Art I do but do not care for" and then like the programmer, hack on what they actually care about outside of "work".

Agreed, but I'd say these would be artists in the "grey area". They are capable of drawing furry art, for example, and have the choice to monetize that, even though they might have become bored with it.

I was mostly considering contemporary artists that you see in museums, and not illustrators. Most of these have moved on to different media, and typically don't draw or paint. They would therefore also not be able to draw commission pieces. And most of the time their work does not sell well.

(Source: am professionally trained artist, tried to sell work, met quite a few artists, thought about this a lot. That's not to say that I may still be completely wrong though, so I liked reading your comment!)

Edit: and of course things get way more complicated and nuanced when you consider gallerists pushing existing artists to become trendy, and artists who are only "discovered" after their deaths, etc. etc.)

Yeah, but I guess wider. It's like the discussion would turn into "Don't use oil colors, then you don't get to do the fun process of mixing water and color together to get it just perfect" while maybe some artists don't think that's the fun process, and all the other categories, all mixed together, and everyone think their reason of doing it is the reason most people do it.

With LLMs, if you did the first in the past, then no matter what license you chose, your work is now in the second category, except you don't get a dime.

> do like the actual typing of letters, numbers and special characters into a computer

and from the first line of the article:

> I love writing software, line by line.

I've said it before and I'll say it again: I don't write programs "line by line" and typing isn't programming. I work out code in the abstract away from the keyboard before typing it out, and it's not the typing part that is the bottleneck.

Last time I commented this on HN, I said something like "if an AI could pluck these abstract ideas from my head and turn them into code, eliminating the typing part, I'd be an enthusiastic adopter", to which someone predictably said something like "but that's exactly what it does!". It absolutely is not, though.

When I "program" away from the keyboard I form something like a mental image of the code, not of the text but of the abstract structure. I struggle to conjure actual visual imagery in my head (I "have aphantasia" as it's fashionable to say lately), which I suspect is because much of my visual cortex processes these abstract "images" of linguistic and logical structures instead.

The mental "image" I form isn't some vague, underspecified thing. It corresponds directly to the exact code I will write, and the abstractions I use to compartmentalise and navigate it in my mind are the same ones that are used in the code. I typically evaluate and compare many alternative possible "images" of different approaches in my head, thinking through how they will behave at runtime, in what ways they might fail, how they will look to a person new to the codebase, how the code will evolve as people make likely future changes, how I could explain them to a colleague, etc. I "look" at this mental model of the code from many different angles and I've learned only to actually start writing it down when I get the particular feeling you get when it "looks" right from all of those angles, which is a deeply satisfying feeling that I actively seek out in my life independently of being paid for it.

Then I type it out, which doesn't usually take very long.

When I get to the point of "typing" my code "line by line", I don't want something that I can give a natural language description to. I have a mental image of the exact piece of logic I want, down to the details. Any departure from that is a departure from the thing that I've scrutinised from many angles and rejected many alternatives to. I want the exact piece of code that is in my head. The only way I can get that is to type it out, and that's fine.

What AI provides, and it is wildly impressive, is the ability to specify what's needed in natural language and have some code generated that corresponds to it. I've used it and it really is very, very good, but it isn't what I need because it can't take that fully-specified image from my head and translate it to the exact corresponding code. Instead I have to convert that image to vague natural language, have some code generated and then carefully review it to find and fix (or have the AI fix) the many ways it inevitably departs from what I wanted. That's strictly worse than just typing out the code, and the typing doesn't even take that long anyway.

I hope this helps to understand why, for me and people like me, AI coding doesn't take away the "line-by-line part" or the "typing". We can't slot it into our development process at the typing stage. To use it the way you are using it we would instead have to allow it to replace the part that happens (or can happen) away from the keyboard: the mental processing of the code. And many of us don't want to do that, for a wide variety of reasons that would take a whole other lengthy comment to get into.

That’s because you’re a subset of software engineers who know what they’re doing and cares about rigour and so on.

There’s many who’s thinking is not so deep nor sharp as yours - LLM’s are welcomed by them but come at a tremendous cost to their cognition and the firms future well-being of its code base. Because this cost is implicit and not explicit it doesn’t occur to them.

Companies don't care about you or any other developer. You shouldn't care about them or their future well-being.

> Because this cost is implicit and not explicit it doesn’t occur to them.

Your arrogance and naiveté blinds you to the fact it is does occur to them, but because they have a better understanding of the world and their position in it, they don't care. That's a rational and reasonable position.

>they have a better understanding of the world and their position in it.

Try not to use better/worse when advocating so vociferously.
As described by the parent they are short-term pragmatic, that is all. This discussion can open up into a huge worldview where different groups have strengths and weaknesses based on this axis of pragmatic/idealistic.

"Companies" are not a monolith, both laterally between other companies, and what they are composed of as well. I'd wager the larger management groups can be pragmatic, where the (longer lasting) R&D manager will probably be the most idealistic of the firm, mainly because of seeing the trends of punching the gas without looking at long-term consequences.

No, they just have a different job than I do and they (and you, I suspect) don't understand the difference.

Software engineers are not paid to write code, we're paid to solve problems. Writing code is a byproduct.

Like, my job is "make sure our customers accounts are secure". Sometimes that involves writing code, sometimes it involves drafting policy, sometimes it involves presentations or hashing out ideas. It's on me to figure it out.

> Like, my job is "make sure our customers accounts are secure".

This is naiveté. Secure customer accounts and the work to implement them is tolerated by the business only while it is necessary to increase profits. Your job is not to secure customer accounts, but to spend the least amount of money to produce a level of account security that will not affect the bottom line. If insecure accounts were tolerated or became profitable, that would be the immediate goal and your job description would pivot on a dime.

Failure to understand this means you don't understand your role, employer, or industry.

> Your job is not to secure customer accounts, but to spend the least amount of money to produce a level of account security that will not affect the bottom line

I completely agree with every line of this statement. That is literally the job.

Of course I balance time/cost against risk. That's what engineers do. You don't
make every house into a concrete bunker because it's "safer", that's expensive and unnecessary. You also don't engineer buildings for hurricanes in California. You do secure against earthquakes, because that's a likely risk.

Engineers are paid for our judgement, not our LOC. Like I said.

> I've used it and it really is very, very good, but it isn't what I need because it can't take that fully-specified image from my head and translate it to the exact corresponding code. Instead I have to convert that image to vague natural language, have some code generated and then carefully review it to find and fix (or have the AI fix) the many ways it inevitably departs from what I wanted.

I agree with this. The hard part of software development happens when you're formulating the idea in your head, planning the data structures and algorithms, deciding what abstractions to use, deciding what interfaces look like--the actual intellectual work. Once that is done, there is the unpleasant, slow, error-prone part: translating that big bundle of ideas into code while outputting it via your fingers. While LLMs might make this part a little faster, you're still doing a slow, potentially-lossy translation into English first. And if you care about things other than "does it work," you still have a lot of work to do post-LLM to clean things up and make it beautiful.

I think it still remains to be seen whether idea -> natural language -> code is actually going to be faster or better than idea -> code. For unskilled programmers it probably already is. For experts? The jury may still be out.

> I work out code in the abstract away from the keyboard before typing it out, and it's not the typing part that is the bottleneck.

Funny thing. I tend to agree, but I think it wouldn't look that way to an outside observer. When I'm typing in code, it's typically at a pretty low fraction of my general typing speed — because I'm constantly micro-interrupting myself to doubt the away-from-keyboard work, and refine it in context (when I was "working in the abstract", I didn't exactly envision all the variable names, for example).

I'm like you. I get on famously with Claude Code with Opus 4.5 2025.11 update.

Give it a first pass from a spec. Since you know how it should be shaped you can give an initial steer, but focus on features first, and build with testability.

Then refactor, with examples in prompts, until it lines up. You already have the tests, the AI can ensure it doesn't break anything.

> focus on features first, and build with testability.

This is just telling me to do this:

> To use it the way you are using it we would instead have to allow it to replace the part that happens (or can happen) away from the keyboard: the mental processing of the code.

I feel like some of these proponents act like a poet has the goal to produce an anthology of poems and should be happy to act as publisher and editor, sifting through the outputs of some LLM stanza generator.

The entire idea using natural language for composite or atomic command units is deeply unsettling to me. I see language as an unreliable abstraction even with human partners that I know well. It takes a lot of work to communicate anything nuanced, even with vast amounts of shared context. That's the last thing I want to add between me and the machine.

What you wrote futher up resonates a lot for me, right down to the aphantasia bit. I also lack an internal monologue. Perhaps because of these, I never want to "talk" to a device as a command input. Regardless of whether it is my compiler, smartphone, navigation system, alarm clock, toaster, or light switch, issuing such commands is never going to be what I want. It means engaging an extra cognitive task to convert my cognition back into words. I'd much rather have a more machine-oriented control interface where I can be aware of a design's abstraction and directly influence its parameters and operations. I crave the determinism that lets me anticipate the composition of things and nearly "feel" transitive properties of a system. Natural language doesn't work that way.

Note, I'm not against textual interfaces. I actually prefer the shell prompt to the GUI for many recurring control tasks. But typing works for me and speaking would not. I need editing to construct and proof-read commands which may not come out of my mind and hands with the linearity it assumes in the command buffer. I prefer symbolic input languages where I can more directly map my intent into the unambiguous, structured semantics of the chosen tool. I also want conventional programming syntax, with unambiguous control flow and computed expressions for composing command flows. I do not want vagaries of natural language interfering here.

> I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer...

This sounds like an alien trying and failing to describe why people like creating things. No, the typing of characters in a keyboard has no special meaning, neither does dragging a brush across a canvas or pulling thread through fabric. It's the primitive desire to create something by your own hands. Have people using AI magically lost all understanding of creativity or creation, everything has to be utilitarian and business?

My entire point is that people are different. For some people (read through the other comments), it's quite literally about typing of characters, or dragging a brush across the canvas. Sure, that might not be the point for you, but my entire point of my comment is that just because it's "obviously because of X" for you, that doesn't mean it's like that for others.

Sometimes I like to make music because I have an idea of the final results, and I wanna hear it like that. Other times, I make music because I like the feeling of turning a knob, and striking keys at just the right moment, and it gives me a feeling of satisfaction. For others, they want to share an emotion via music. Does this mean someone of us are "making music for the wrong reasons"? I'd claim no.

No, they're right. Your description is what you get from outsiders who don't understand what they're seeing.

In a creative process, when you really know your tools, you start being able to go from thought to result without really having to think about the tools. The most common example when it comes to computers would be touch-typing - when your muscle memory gets so good you don't think about the keyboard at all anymore, your hands "know" what to do to get your thoughts down. But for those of us with enough experience in the programming languages and editor/IDE we use, the same thing can happen - going from thought to code is nearly effortless, as is reading code, because we don't need to think about the layers in between anymore.

But this only works when those tools are reliable, when we know they'll do exactly what we expect. AI tooling isn't reliable: It introduces two lossy translation layers (thought -> English and English -> code) and a bunch of waiting in the middle that breaks any flow. With faster computers maybe we can eliminate the waiting, but the reliability just isn't there.

This applies to music, painting, all sorts of creative things. Sure there's prep time beforehand with physical creation like painting, but when someone really gets into the flow it's the same: they're not having to think about the tools so much as getting their thoughts into the end result. The tools "disappear".

> Other times, I make music because I like the feeling of turning a knob, and striking keys at just the right moment, and it gives me a feeling of satisfaction.

But I'll bet you're not thinking about "I like turning this knob" at the moment you're doing it, I'll bet you're thinking "Increase the foo" (and if you're like me it's probably more liking knowing that fact without forming the words) and the knob's immediate visceral feedback is where the satisfaction comes from because you're increasing the foo without having to think about how to do it - in part because of how reliable it is.

I bet you also sometimes like to make music because the final result emerges from your intimate involvement with striking keys, no? That's the suggestion.

Let me get this right. You're telling me that in your personal experience, you don't abstract away low level actions like pressing keys of your instrument or typing on the keyboard? You're genuinely telling me you derive equal pleasure from music as the feel of the keys?

Nah bro, most of us learn touch typing and musical instrument finger exercises etc when starting out, it's usually abstracted away once we get competent.

AI takes away the joy of creation, not the low level actions. That's like abstracted twice over..

Do you enjoy the process of creating a solution more than the actual solution?

This is the main difference why people argue against LLMs in programming.

I'm in the "I want to solve a problem" end of the spectrum. Many others are in the "I want the code to be elegant, maintanable and beautifully crafted - oh, yeah, the problem might be solved too" end.

I don't think these characterizations in either direction are very helpful; I understand they're coming from a place with someone trying to make sense of why their ingrained notion of what creativity means and what the "right" way to generate software projects is is not shared by other people.

I use CC for both business and personal projects. In both cases: I want to achieve something cool. If I do it by hand, it is slow, I will need to learn something new which takes too much time and often time the thing(s) I need to learn is not interesting to me (at the time). Additionally, I am slow and perpetually unhappy with the abstractions and design choices I make despite trying very hard to think through them. With CC: it can handle parts of the project I don't want to deal with, it can help me learn the things I want to learn, it can execute quickly so I can try more things and fail fast.

What's lamentable is the conclusion of "if you use AI it is not truly creative" ("have people using AI lost all understanding of creativity or creation?" is a bit condescending).

In other threads the sensitive dynamic from the AI-skeptic crowds is more or less that AI enthusiasts "threaten or bully" people who are not enthusiastic that they will get "punished" or fall behind. Yet at the same time, AI-skeptics seem to routinely make passive aggressive implications that they are the ones truly Creating Art and are the true Craftsman; as if this venture is some elitist art form that should be gate kept by all of you True Programmers (TM).

I find these takes (1) condescending, (2) wrong and also belying a lack of imagination about what others may find genuinely enjoyable and inspiring, (3) just as much of a straw man as their gripes against others "bullying" them into using AI.

> I think there is a section of programmer who actually do like the actual typing of letters, numbers and special characters into a computer

but luckily for us, we can still do that, and it's just as fun as it ever was. LLMs don't take anything away from the fun of actually writing code, unless you choose to let them.

if anything the LLMs make it more fun, because the boring bits can now be farmed out while you work on the fun bits. no, i don't really want to make another CRUD UI, but if the project i'm working on needs one i can just let claude code do that for me while i go back to working on the stuff that's actually interesting.

I think the downside is the developers who love the action of coding managed to accomplish several things at once - they got to code, and create things, and get paid lots for doing it.

AI coding makes creating things far more efficient (as long as you use AI), and will likely mean you don't get paid much (unless you use AI).

You can still code for the fun of it, but you don't get the ancillary benefits.

Thanks for reading / commenting this post. Initially it seemed like I received a bunch of very negative comments, now I read most of the thread, and there are very good points, articulated with sensibility. Thank you.

I wanted to provide some more context that is not part of the blog post. Since somebody may believe I don't enjoy / love the act of writing code.

1. I care a lot about programming, I love creating something from scratch, line by line. But: at this point, I want to do programming in a way that makes me special, compared to machines. When the LLM hits a limit, and I write a function in a way it can't compete, that is good.

2. If I write a very small program that is like a small piece of poetry, this is good human expression. I'll keep doing this as well.

3. But, if I need to develop a feature, and I have a clear design idea, and I can do it in 2 hours instead of 2 weeks, how to justify to myself that, just for what I love, I will use a lot more time? That would be too much of ego-centric POV, I believe.

4. For me too this is painful, as a transition, but I need to adapt. Fortunately I also enjoyed a lot the design / ideas process, so I can focus on that. And write code myself when needed.

5. The reason why I wrote this piece is because I believe there are still a lot of people that are unprepared for the fact we are going to be kinda of obsolete in what defined us, as a profession: the ability to write code. A complicated ability requiring a number of skills at the same time, language skills, algorithms, problem decomposition. Since this is painful, and I believe we are headed in a certain direction, I want to tell the other folks in programming to accept reality. It will be easier, this way.

I think there is a danger in the enthusiasm for AI inside of these excellent points, namely that the skills that make a good programmer are not inherent, they are learned.

The comparison would be a guy who is an excellent journeyman electrician. This guy has visual-spatial skills that makes bending and installing conduit a kind of art. He has a deep and intuitive understanding of how circuits are balanced in a panel, so he does not overload a phase. But he was not born with them. These are acquired over many years of labor and tutelage.

If AI removes these barriers--and I think it will, as AI-enhanced programmers will out-perform and out-compete those who are not in today's employment market--then the programmer will learn different skills that may or may not be in keeping with language skills, algorithms, problem decomposition, etc. They may in fact be orthogonal to these skills.

The effect of this may be an improvement, of course. It's hard to say for sure as I left my crystal ball in my other jacket. But it will certainly be different. And those who are predisposed for programming in the old-school way may not find the field as attractive because it is no longer the same sort of engineering, something like the difference between the person that designs a Lego set and the person that assembles a Lego set. It could, in fact, mean that the very best programmers become a kind of elite, able to solve most problems with just a handful of those elite programmers. I'm sure that's the dream of Google and Microsoft. However this will centralize the industry in a way not seen since perhaps IBM, only with a much smaller chance of outside disruption.

Maybe solving more trivial problems with AI will left novice programmer to do more depth problems and will make them better faster, because they will spend time solving problems that matter.

That is possible, for sure. But think of it like a person learning the piano. You could practice your arpeggios on a Steinway, or you can buy a Casio with an arpeggiator button.

At a certain point, the professional piano player can make much better use of the arpeggiator button. But the novice piano player benefits greatly from all the slogging arpeggio practice. It's certainly possible that skipping all that grunt work will improve and/or advance music, but it's hardly a sure thing. That's the experiment we're running right now with AI programming. I suppose we'll see soon enough, and I hope I'm utterly wrong about the concerns I have.

That's really interesting, but i'm wondering if this is as rational as it looks.

> we are going to be kinda of obsolete in what defined us, as a profession: the ability to write code

Is it a fact, really? I don't think "writing code" is a defining factor, maybe it's a prerequisite, as being able to write words hardly defines "a novelist".

Anyway, prompt writing skills might become obsolete quite soon. So the main question might be to know which trend of technological evolution to pick and when, in order not to be considered obsolete. A crystal ball might still be more relevant than LLMs for that.

I call it "the ability to communicate intent [using a programming language]" and suddenly building with AI looks at lot more like the natural extension of what we used to do writing code by ourselves.

I don't think our profession was writing code to begin with (and this may be a bit uuhh. rewriting history?); what we do is take an idea, requirements, an end goal and make it reality. Often times that involves writing code, but that's only one aspect of the software developer's job.

Analogy time because comment sections love analogies. A carpenter can hammer nails, screw screws, make holes, saw wood to size. If they then use machines to make that work easier, do they stop being carpenters?

It's good if not essential to be able to write code. It's more important to know what to write and when. Best thing to do at this point is to stop attaching one's self-worth with the ability to write code. That's like a novelist (more analogies) who praises their ability to type at 100wpm. The 50 shades books proved you don't need to either touch type (the first book was mostly written on a blackberry apparently) or be good at writing to be successful, lol.

Agreed - as I see it, it's akin to the transitions from machine code -> assembly language -> C -> Javascript. As time went by, knowing the deep internals of the machine became less and less necessary, even though having that knowledge still gives an engineer a useful insight into their work and often makes them better at their job. The goal remains the same - make the computer do the thing; only the mechanism changes as the tools evolve.

"-> AI" is just the next step along that journey. Maybe it will end at "-> AGI" and then humans will engage in programming mostly for the craft and the pleasure of it, like other crafts that were automated away over the ages.

As a specific example of this, the U.S. 18F team had helped the Forest Service a decade ago with implementing a requirement to help people get a permit to cut down a Christmas tree.

Although there was a software component for the backend, the thing that the actual user ended up with was a printed-out form rather than a mobile app or QR code. This was a deliberate design decision (https://greacen.com/media/guides/2019/02/12/open-forest-laun...), not due to a limitation of software.

I still really, really, really struggle to see how humans are going to maintain and validate the programs written by LLMs if we no longer know (intimately) how to program. Any thoughts?

Very few people have the expertise to write efficient assembly code, yet everyone relies on compilers and assemblers to translate high-level code to byte-level machine code. I think same concept is true here.

Once coding agents become trivial, few people will know the detail of the programming language and make sure intent is correctly transformed to code, and the majority will focus on different objectives and take LLM programming for granted.

No, that's a completely different concept, because we have faultless machines which perfectly and deterministically translate high-level code into byte-level machine code. This is another case of (nearly) perfect abstraction.

On the other hand, the whole deal of the LLM is that it does so stochastically and unpredictably.

The unpredictable part isn't new - from a project manager's point of view, what's the difference between an LLM and a team of software engineers? Both, from that POV, are a black box. The "how" is not important to them, the details aren't important. What's important is that what they want is made a reality, and that customers can press on a button to add a product to their shopping cart (for example).

LLMs mean software developers let go of some control of how something is built, which makes one feel uneasy because a lot of the appeal of software development is control and predictability. But this is the same process that people go through as they go from coder to lead developer or architect or project manager - letting go of control. Some thrive in their new position, having a higher overview of the job, while some really can't handle it.

"But this is the same process that people go through as they go from coder to lead developer or architect or project manager - letting go of control."

In those circumstances, it's delegating control. And it's difficult to judge whether the authority you delegated is being misused if you lose touch with how to do the work itself. This comparison shouldn't be pushed too far, but it's not entirely unlike a compiler developer needing to retain the ability to understand machine code instructions.

As someone that started off with assembly issues for a large corporation - assembly code may sometimes contain very similiar issues that mroe high-level code those, the perfection of the abstraction is not guaranteed.

But yeah, there's currently a wide gap between that and a stochastic LLM.

We also have machines that can perfectly and deterministically check written code for correctness.

And the stohastic LLM can use those tools to check whether its work was sufficient, if not, it will try again - without human intervention. It will repeat this loop until the deterministic checks pass.

You can make analysers that check for deeply nested code, people calling methods in the wrong order and whatever you want to check. At work we've added multiple Roslyn analysers to our build pipeline to check for invalid/inefficient code, no human will be pinged by a PR until the tests pass. And an LLM can't claim "Job's Done" before the analysers say the code is OK.

And you don't need to make one yourself, there are tons you can just pick from:

> It's not like testing code is a new thing. Junit is almost 30 years old today.

Unit tests check whether code behaves in specific ways. They certainly are useful to weed out bugs and to ensure that changes don't have unintended side effects.

> And code correctness:

These are tools to check for syntactic correctness. That is, of course, not what I meant.

Algorithmic correctness? Unit tests are great for quickly poking holes in obviously algorithmically incorrect code, but far from good enough to ensure correctness. Passing unit tests is necessary, not sufficient.

Syntactic correctness is more or less a solved problem, as you say. Doesn't matter if the author is a human or an LLM.

It depends on the algorithm of course. If your code is trying to prove P=NP, of course you can't test for it.

But it's disingenuous to claim that even the majority of code written in the world is so difficult algorithmically that it can't be unit-tested to a sufficient degree.

Suppose you're right and the "majority of code" is fully specified by unit testing (I doubt it). The remaining body of code is vast, and the comments in this thread seem to overlook that.

> Very few people have the expertise to write efficient assembly code, yet everyone relies on compilers and assemblers to translate high-level code to byte-level machine code. I think same concept is true here.

That's a poor analogy which gets repeated in every discussion: compilers are deterministic, LLMs are not.

> That's a poor analogy which gets repeated in every discussion: compilers are deterministic, LLMs are not.

Compilers are not used directly, they are used by human software developers who are also not deterministic.

From the perspective of an organization with a business or service-based mission, they already know how to supervise non-deterministic LLMs because they already know how to supervise non-deterministic human developers.

Why does it matter if LLMs are not deterministic? Who cares?

There should be tests covering meaningful functionality, as long as the code passes the tests, ie. the externally observable behaviour is the same, I don't care. (Especially, if many tests can also be autogenerated with the LLM.)

>>> Very few people have the expertise to write efficient assembly code, yet everyone relies on compilers and assemblers to translate high-level code to byte-level machine code. I think same concept is true her

>> That's a poor analogy which gets repeated in every discussion: compilers are deterministic, LLMs are not.

> Why does it matter if LLMs are not deterministic? Who cares?

In the context of this analogy, it matters. If you're not using this analogy, then sure, only the result matters. But when the analogy being used is deterministic, then, yes, it matters.

You can't very well claim "We'll compare this non-deterministic process to this other deterministic process that we know works."

The difference is that if you write in C you can debug in C. You don't have to debug the assembly. You can write an english wish list for an LLM but you will still have to debug the generated code. To debug it you will need to understand it.

> how humans are going to maintain and validate the programs written by LLMs if we no longer know (intimately) how to program

Short answer: we wouldn’t be able to. Slightly-less short answer: unlikely to happen.

Most programmers today can’t explain the physics of computation. That’s fine. Someone else can. And if nobody can, someone else can work backwards to it.

> > how humans are going to maintain and validate the programs written by LLMs if we no longer know (intimately) how to program

> Short answer: we wouldn’t be able to.

That's a huge problem! A showstopper for many kinds of programs!

> Slightly-less short answer: unlikely to happen.

Could you elaborate?

> Most programmers today can’t explain the physics of computation. That’s fine. Someone else can. And if nobody can, someone else can work backwards to it.

That's not the same at all. We have properly abstracted away the physics of computation. A modern computer operates in a way where, if you use it the way you've been instructed to, the physics underlying the computations cannot affect the computation in any undocumented way. Only a very few (and crucically, known and understood!!) physical circumstances can make the physics influence the computations. A layperson does not need to know how those circumstances work, only roughly what their boundaries are.

This is wildly different from the "abstraction" to programming that LLMs provide.

> That's a huge problem! A showstopper for many kinds of programs!

We have automated validation and automated proofs.

Proof is necessary. Do you validate the theorem prover, or trust that it works? Do you prove the compiler is correctly compiling the program (when it matters, you should, given they do sometimes re-write things incorrectly) or trust the compiler?

> We have properly abstracted away the physics of computation. A modern computer operates in a way where, if you use it the way you've been instructed to, the physics underlying the computations cannot affect the computation in any undocumented way.

You trust the hardware the code is running on? You shouldn't.

Rowhammer comes to mind, but it's hardly the only case. US banned some Chinese chips for unspecified potential that this was going on.

For some people it's OK to run a few simple tests on the chip's output to make sure it doesn't have something like the Pentium FDIV bug, for others they remove the silicon wafer from the packaging and scan it with an electron microscope, verify not just each transistor is in the right place but also that the wires aren't close enough to have currents quantum tunnelling or act as an antenna that leaks out some part of a private key.

Some people will go all the way down to the quantum mechanics. Exploits are possible at any level, domains where the potential losses exceed the cost of investigation do exist, e.g. big countries and national security.

Proof is necessary. The abstraction of hardware is good enough for most of us, and given the excessive trust already given to NPM and other package management tools, LLM output that passes automated tests is already sufficient for most.

People like me who don't trust package management tools, or who filed bugs with Ubuntu for not using https enough and think that Ubuntu's responses and keeping the bug open for years smelled like "we have a court order requiring this but can't admit it" (https://bugs.launchpad.net/ubuntu-website-content/+bug/15349...)… well, I can't speak for the paranoid, but I'm also the curious type who learned how to program just because the book was there next to the C64 game tapes.

> We have automated validation and automated proofs.

Example?

> Proof is necessary. Do you validate the theorem prover, or trust that it works? Do you prove the compiler is correctly compiling the program (when it matters, you should, given they do sometimes re-write things incorrectly) or trust the compiler?

I trust that the people who wrote the compiler and use it will fix mistakes. I trust the same people to discover compiler backdoors.

As for the rest of what you wrote: you're missing the point entirely. Rowhammer, the fdiv bug, they're all mistakes. And sure, malevolence also exists. But when mistakes or malevolence are found, they're fixed, or worked around, or at least documented as mistakes. With an LLM you don't even know how it's supposed to behave.

> you're missing the point entirely. Rowhammer, the fdiv bug, they're all mistakes. And sure, malevolence also exists.

Rowhammer was a thing because the physics was ignored. Calling it a mistake is missing the point, it demonstrates the falseness of the previous claim:

We have properly abstracted away the physics of computation. A modern computer operates in a way where, if you use it the way you've been instructed to, the physics underlying the computations cannot affect the computation in any undocumented way.

Rowhammer *is* the physics underlying the computations affecting the computation in a way that was undocumented prior to it getting discovered and, well, documented. Issues like this exist before they're documented, and by definition nobody knows how many unknown things like this have yet to be found.

> But when mistakes or malevolence are found, they're fixed, or worked around, or at least documented as mistakes.

If you vibe code (as in: never look at the code), then find an error with the resulting product, you can still just ask the LLM to fix that error.

I only had a limited time to experiment with this before Christmas (last few days of a free trial, thought I'd give it a go to see what the limits were), and what I found it doing wrong was piling up technical debt, not that it was a mysterious ball of mud beyond its own ability to rectify.

> With an LLM you don't even know how it's supposed to behave.

LLM generated source code: if you've forgotten how to read the source code it made for you to solve your problem and can't learn how to read that source code and can't run the tests of that source code, at which point it's as interpretable as psychology.

The LLMs themselves: yes, this is the "interpretability" problem, people are working on that.

I am of course aware. Any malevolent backdoor in your compiler could also exist in your LLM. Or the compiler that compiled the LLM. So you can never do better.

> Rowhammer is the physics underlying the computations affecting the computation in a way that was undocumented prior to it getting discovered and, well, documented. Issues like this exist before they're documented, and by definition nobody knows how many unknown things like this have yet to be found.

Yep. But it's a bug. It's a mistake. The unreliability of LLMs is not.

> If you vibe code (as in: never look at the code), then find an error with the resulting product, you can still just ask the LLM to fix that error.

Of course. But you need skills to verify that it did.

> LLM generated source code: if you've forgotten how to read the source code it made for you to solve your problem and can't learn how to read that source code and can't run the tests of that source code, at which point it's as interpretable as psychology.

Reading source code is such a minute piece of the task of understanding code that I can barely understand what you mean.

At the current time this is essentially science fiction though. This something that the best funded companies on the planet (as well as many many others) work on and seem to be completely unable to achieve despite trying their best for years now, despite an incredible hype.

It feels like if those resources were poured in nuclear fusion for example we'd have it production ready by now.

The field is also not a couple of years old, this has been tried for decades. Sure only now companies decided to put essentially "unlimited" resources into it, but while it showed that certain things are possible and work extremely well, it also strongly hinted that at least the current approach will not get us there, especially not without significant trade-off (that whole over training vs "creativity" and hallucination topic).

Doesn't mean it won't come, but that it doesn't appear a "we just need a bit more development" topic. The state hasn't changed much. Models became bigger and bigger and people added that "thinking" hack and agents and agents for agents, but it also didn't change much about the initial approach and its limitations, given that they haven't cracked these problems after years of hyped funding.

Would be amazing if we would have AIs that automate research and maybe help us fix all the huge problems the world is facing. I'd absolutely love that. I'd also love it if people could easily create tools, games, art. However that's not the reality we live in. Sadly.

Tutoring – whether AI or human – does not provide the in-depth understanding necessary for validation and long-term maintenance. It can be a very useful step on the way there, but only a step.

Same how we do it now - look at the end result, test it. Testers never went away.

Besides, your comment goes by the assumption that we no longer know (intimately) how to program - is that true? I don't know C or assembly or whatever very well, but I'm still a valuable worker because I know other things.

I mean it could be partially true - but it's like having years of access to Google to quickly find just what I need, meaning I never learned how to read e.g. books on software development or scientific paper end to end. Never felt like I needed to have that skill, but it's a skill that a preceding generation did have.

> Besides, your comment goes by the assumption that we no longer know (intimately) how to program - is that true? I don't know C or assembly or whatever very well, but I'm still a valuable worker because I know other things.

The proposal seems to be for LLMs to take over the task of coding. I posit that if you do not code, you will not gain the skills to do so well.

> I mean it could be partially true - but it's like having years of access to Google to quickly find just what I need, meaning I never learned how to read e.g. books on software development or scientific paper end to end.

I think you've misunderstood what papers are for or what "the previous generation" used them for. It is certainly possible to extract something useful from a paper without understanding what's going on. Googling can certainly help you. That's good. And useful. But not the main point of the paper.

Fair question but haven't we been doing this for decades? Very few people know how to write assembly and yet software has proliferated. This is just another abstraction.

> Fair question but haven't we been doing this for decades? Very few people know how to write assembly and yet software has proliferated. This is just another abstraction.

Not at all. Given any "layperson input", the expert who wrote the compiler that is supposed to turn it into assembly can describe in excruciating detail what the compiler will do and why. Not so with LLMs.

Said differently: If I perturb a source code file with a few bytes here and there, anyone with a modicum of understanding of the compiler used can understand why the assembly changed the way it did as a result. Not so with LLMs.

But there's a limit to that. There's (relatively) very few people that can explain the details of e.g. a compiler, compared to for example React front-end developers that build B2C software (...like me). And these software projects grow, ultimately to the limit of what one person can fit in their head.

Which is why we have lots of "rules" and standards on communication, code style, commenting, keeping history, tooling, regression testing, etc. And I'm afraid those will be the first to suffer when code projects are primarily written by LLMs - do they even write unit tests if you don't tell them to?

Really enjoyed your article, and it reflects a lot of the pain-points I experience with models. I tend to still write and review LLM code after creation, but there is definitely a shift within how much code I create "artisinally" and how much is reviewd in terms of scale.

If I need to implement a brand new feature for the project, I will find myself needing to force a view into a LLM because it will help me achieve 80% of the feature in 1% of the time, even if the end result requires a scale of refactoring, it's rarely the time that the original feature would've taken me.

But, I think that's also because I have solid refactoring foundations, I know what makes good code, and I think if I had access to these tools 5 years ago, I would not be able to achieve that same result, as LLMs typically steer towards junior level coding as a consequence of their architecture.

This is by far the best summary of the state of affairs, or rather, the most sensible perspective that one should have on the state of affairs, that I've read so far.

Of the four coding examples you describe, I find none of them compelling either in their utility or as a case for firing a dev (with one important caveat [0]).

In each example, you were already very familiar with the problem at hand, and that probably took far longer than any additional time savings AI could offer.

0. Perhaps I consider your examples as worthless simply because you gloss over them so quickly, in which case that greatly increases the odds in most companies that you would be fired.

All of that makes a lot of sense. And unlike a lot of both pro-AI and anti-AI people I would find it great if it was the case. Unlike maybe a lot of people here I am less attached to this profession as a profession. I'd also love it if I could have some LLM do the projects I always wanted to finish. It would be essentially Christmas.

However your experiences really clash with mine and I am trying to work out why, because so far I haven't been able to copy your workflow with success. It would be great if I could write a proper spec and the output of the LLM would be good (not excellent, not poetry, but just good). However the output for anything that isn't "stack overflow autocomplete" style it is abysmal. Honestly I'd be happy if good output is even on the horizon.

And given that "new code" is a lot better than working on an existing project and an existing LLM generated project being better than a human made project and it still being largely bad, often with subtle "insanity" I have a hard time to apply what you say to reality.

I do not understand the disconnect. I am used to writing specs. I tried a lot of prompting changes, to a degree where it almost feels like a new programming language. Sure there are things that help, but the sad reality is that I usually spend more time dealing with the LLM than I'd need to write that code myself. And worse still, I will have to fix it and understand it, etc. to be able to keep on working on it and "refining" it, something that simply isn't needed at least to that extent if I wrote that code myself.

I really wished LLMs would provide that. And don't get me wrong, I do think there are really good applications for LLMs. Eg anything that needs a transform where even a complex regex won't do. Doing very very basic stuff where one uses LLMs essentially as an IDE-integrated search engine, etc.

However the idea that it's enough to write a spec for something even semi-novel currently appears to be out of reach. For trivial generic code it essentially saves you from either writing it yourself copy pasting it off some open source projects.

Much context, for the question that hopefully explains a lot of stuff. Those 2 hours that you use instead of two weeks. How do you spend them? Is that refining prompts, is that fixing the LLM output, is that writing/adapting specs, is it something else?

Also could it be that there is a bias on "time spent" because of it being different work or even just a general focus on productivity, more experience, etc.?

I am trying to understand where that huge gap in experience that people have really stems from. I read your posts, I watch video on YouTube, etc. I just haven't seen "I write a spec [that is is shorter/less effort than the actual code] and get good output". Every time I read claims about it in blog posts and so on there appear to be parts missing to reproduce the experience.

I know that there are a lot of "ego-centric POV" style AI "fear". People of course have worries about their jobs, and I understand. However, personally I really don't and as mentioned I'd absolutely love to use it like that on some projects, but whenever I try to replicate experiences that aren't just "toying" in the sense of anything that even has basic reliability requirements and is a bit more complex I fail to do so and it's probably me, but I tried for at least a year to replicate such things and it's failure after failure even for more simple things.

That said there are productivity gains with autocomplete, transforming stuff and what people largely call "boilerplate" as well as more quickly writing small helpers that I'd otherwise have copied off some older project. Those things work good enough, just like how autocomplete is good enough. For bigger and more novel things where a search engine is also not the right approach it fails, but this is where the interesting bits are. Having topics that haven't been solved a hundred times over.

Thanks for the post. I found it very interesting and I agree with most of what you said. Things are changing, regardless of our feelings on the matter.

While I agree that there is something tragic about watching what we know (and have dedicated significant time and energy in learning) devalued. I'm still exited for the future, and for the potential this has. I'm sure that given enough time this will result in amazing things that we cannot even imagine today. The fact that the open models and research is keeping up is incredibly important, and probably the main things that keeps me optimistic for the future.

What I don't understand about this whole "get on board the AI train or get left behind" narrative, what advantage does an early adopter have for AI tools?

The way I see it, I can just start using AI once they get good enough for my type of work. Until then I'm continuing to learn instead of letting my brain atrophy.

This is a pretty common position: "I don't worry about getting left behind - it will only take a few weeks to catch up again".

I don't think that's true.

I'm really good at getting great results out of coding agents and LLMs. I've also been using LLMs for code on an almost daily basis since ChatGPT's release on November 30th 2022. That's more than three years ago now.

Meanwhile I see a constant flow of complaints from other developers who can't get anything useful out of these machines, or find that the gains they get are minimal at best.

Using this stuff well is a deep topic. These things can be applied in so many different ways, and to so many different projects. The best asset you can develop is an intuition for what works and what doesn't, and getting that intuition requires months if not years of personal experimentation.

I don't think you can just catch up in a few weeks, and I do think that the risk of falling behind isn't being taken seriously enough by much of the developer population.

I'm glad to see people like antirez ringing the alarm bell about this - it's not going to be a popular position but it needs to be said!

I think you are right in saying that there is some deep intuition that takes months, if not years, to hone about current models, however, the intuition some who did nothing but talk and use LLMs nonstop two years ago would be just as good today as someone who started from scratch, if not worse because of antipatterns that don’t apply anymore, such as always starting a new chat and never using a CLI because of context drift.

Also, Simon, with all due respect, and I mean it, I genuinely look in awe at the amount of posts you have on your blog and your dedication, but it’s clear to anyone that the projects you created and launched before 2022 far exceed anything you’ve done since. And I will be the first to say that I don’t think that’s because of LLMs not being able to help you. But I do think it’s because what makes you really, really good at engineering you kept replacing slowly but surely with LLMs more and more by the month.

If I look at Django, I can clearly see your intelligence, passion, and expertise there. Do you feel that any of the projects you’ve written since LLMs are the main thing you focus on are similar?

Think about it this way:
100% of you wins against 100% of me any day.
100% of Claude running on your computer is the same as 100% of Claude running on mine.
95% of Claude and 5% of you, while still better than me (and your average Joe), is nowhere near the same jump from 95% Claude and 5% me.

I do worry when I see great programmers like you diluting their work.

My great regret from the past few years is that experimenting with LLMs has been such a huge distraction from my other work! My https://llm.datasette.io/ tool is from that era though, and it's pretty cool.

I do think your datasettes work is fantastic and I genuinely hope you take my previous message the right way. I’m not saying you do something bad, quite the opposite, I feel like we need more of you and I’m afraid because of LLMs we get less of you.

It used to be that the bots had a short context window, and they struggled with getting confused by past context, so it was much better to make a new chat every now and then to keep the thread on track.

The opposite is true now. The context windows are enormous, and the bots are able to stay on task extremely well. They're able to utilize any previous context you've provided as part of the conversation for the new task, which improves their performance.

The new pattern I am using is a master chat that I only ever change if I am doing something entirely different

That’s cool. I know context windows are arbitrarily larger now because consumers think that larger window = better, but I think the sentiment that the model can’t even use the window effectively still stands?

I still find LLMs perform best with a potent and focussed context to work with, and performance goes down quite significantly the more context it has.

I worked on a startup experimenting with using gemini-2.0-flash (the year old model) using its full 1m context window to query technical documents. We found it to be extremely successful at needle-in-a-haystack type problems.

As we migrated to newer models (gemini-3.0 and the o4-mini models) we again found it performed even better with x00k tokens. Our system prompt grew to about 20k tokens and the bots were able to handle it perfectly. Our issue became time to first token with large context, rather than the bot quality.

The ultra large 1m+ llama models were reported to be ineffective at >1m context. But at this point, it becomes so cost prohibitive to use anyway.

I am continuing to have success using Cursor's Auto model, and GPT-5.1 with extremely long conversations. I use different chats for different problems moreso for my own compartmentalisation of thoughts, rather than as a necessity for the bot.

> 95% of Claude and 5% of you, while still better than me (and your average Joe), is nowhere near the same jump from 95% Claude and 5% me.

I see what you're saying, but I'm not sure it is true. Take simonw and tymscar, put them each in charge of a team of 19 engineers (of identical capabilities). Is the result "nowhere near the same jump" as simonw vs. tymscar alone? I think it's potentially a much bigger jump, if there are differences in who has better ideas and not just who can code the fastest.

Yeah... and besides managerial skills, also product (using the word loosely) sense, user empathy, clarity of vision, communication skills. They've always been multipliers for programmers, even more so in this moment.

It needs to be said that your opinion on this is well understood by the community, respected, but also far from impartial. You have a clear vested interest in the success of _these_ tools.

There's a learning curve to any toolset, and it may be that using coding agents effectively is more than a few weeks of upskilling. It may be, and likely will be, that people make their whole careers about being experts on this topic.

But it's still a statistical text prediction model, wrapped in fancy gimmicks, sold at a loss by mostly bad faith actors, and very far from its final form. People waiting to get on the bandwagon could well be waiting to pick up the pieces once it collapses.

I have a lot of respect from Simon and read a lot of his articles.

But I'm still seeing clear evidence it IS a statistical text prediction model. You ask it the right niche thing and it can only pump out a few variations of the same code, that's clearly someone else's code stolen almost verbatim.

And I just use it 2 or 3 times a day.

How are SimonW and AntiRez not seeing the same thing?

How are they not seeing the propensity for both Claude + ChatGPT to spit out tons of completely pointless error handling code, making what should be a 5 line function a 50 line one?

How are they not seeing that you constantly have to nag it to use modern syntax. Typescript, C#, Python, doesn't matter what you're writing in, it will regularly spit out code patterns that are 10 years out of date. And woe betide you using a library that got updated in the last 2 years. It will constantly revert back to old syntax over and over and over again.

I've also had to deal with a few of my colleagues using AI code on codebases they don't really understand. Wrong sort, id instead of timestamp. Wrong limit. Wrong json encoding, missing key converters. Wrong timezone on dates. A ton of subtle, not obvious, bugs unless you intimately know the code, but would be things you'd look up if you were writing the code.

And that's not even including the bit where the AI obviously decided to edit the wrong search function in a totally different part of the codebase that had nothing to do with what my colleague was doing. But didn't break anything or trigger any tests because it was wrapped in an impossible to hit if clause. And it created a bunch of extra classes to support this phantom code, so hundreds of new lines of code just lurking there, not doing anything but if I hadn't caught it, everyone thinks it does do something.

It's mostly a statistical text model, although the RL "reasoning" stuff added in the past 12 months makes that a slightly less true statement - it has extra tricks now to help it bias bits of code to statistically predict that are more likely to work.

The real unlock though is the coding agent harnesses. It doesn't matter any more if it statistically predicts junk code that doesn't compile, because it will see the compiler error and fix it. If you tell it "use red/green TDD" it will write the tests first, then spot when the code fails to pass them and fix that too.

> How are they not seeing the propensity for both Claude + ChatGPT to spit out tons of completely pointless error handling code, making what should be a 5 line function a 50 line one?

TDD helps there a lot - it makes it less likely the model will spit out lines of code that are never executed.

> How are they not seeing that you constantly have to nag it to use modern syntax. Typescript, C#, Python, doesn't matter what you're writing in, it will regularly spit out code patterns that are 10 years out of date.

I find that if I use it in a codebase with modern syntax it will stick to that syntax. A prompting trick I use a lot is "git clone org/repo into /tmp and look at that for inspiration" - that way even a fresh codebase will be able to follow some good conventions from the start.

Plus the moment I see it write code in a style I don't like I tell it what I like instead.

> And that's not even including the bit where the AI obviously decided to edit the wrong search function in a totally different part of the codebase that had nothing to do with what my colleague was doing.

I usually tell it which part of the codebase to execute - or if it decides itself I spot that and tell it that it did the wrong thing - or discard the session entirely and start again with a better prompt.

Ok, but given the level of detail you're supplying, at that point isn't it quicker to write the code yourself than it is to prompt?

As you have to explain much of this, the natural language words written are much more than just the code and less precise, so it actually takes much longer to type and is more ambiguous. And obviously at the moment ChatGPT tends to make assumptions without asking you, Claude is a little better at asking you for clarification.

I find it so much faster to just ask Claude/ChatGPT for an example of what I'm trying to do and then cut/paste/modify it myself. So just use them as SO on steriods, no agents, no automated coding. Give me the example, and I'll integrate it.

And the end code looks nothing like the supplied example.

I tried using AquaVoice (which is very good) to dictate to it, and that slightly helped, but often I found myself going so slowly just fully prompting the AI when I would have already finished the new code myself at that point.

I was thinking about this last night, I do wonder if this is another example of the difference between deep/narrow coding of specialist/library code and shallow/wide of enterprise/business code.

If you're writing specialist code (like AntiRez), it's dealing with one tight problem. If you're writing enterprise code, it has to take into account so many things, explaining it all to the AI takes forever. Things like use the correct settings from IUserContext, add to the audit in the right place, use the existing utility functions from folder X, add json converters for this data structure, always use this different date encoding because someone made a mistake 10 years ago, etc.

I get that some of these would end up in agents.md/claude.md, but as many people have complained, AI agents often rapidly forget those as the context grows so you have to go through any code generated with a toothcomb, or get it to generate a disproportionate amount of tests, which again you have to explain each and every one.

I guess that will be fixed eventually. But from my perspective, as they're still changing so rapidly and much advice from even 6/9 months ago is now utterly wrong, why not just wait.

I, like many others on this thread, also believe that it's going to take about a week to get up-to-speed when they're finally ready. It's not that I can't use them now, it's that they're slow, unreliable, prone to being a junior on steriods, and actually create more work when reviewing the code than if I'd just written it myself in the first place, and the code is much, much, much worse than MY code. Not necessarily all the people I've worked with's code, but definitely MY code is usually 50-90% more concise.

> If you're writing enterprise code, it has to take into account so many things, explaining it all to the AI takes forever. Things like use the correct settings from IUserContext, add to the audit in the right place, use the existing utility functions from folder X, add json converters for this data structure, always use this different date encoding because someone made a mistake 10 years ago, etc.

The fix for this is... documentation. All of these need to be documented in a place that's accessible to the agent. That's it.

I've just about one-shotted UI features with Claude just by giving it a screenshot of the Figma design (couldn't be bothered with the MCP) and the ticket about the feature.

It used our very custom front-end components correctly, used the correct testing library, wrote playwright tests and everything. Took me maybe 30 minutes from first prompt to PR.

If I (a backend programmer) had to do it, it would've taken me about a day of trying different things to see which one of the 42 different ways of doing it worked.

I talk about why that doesn't work the line after you've quoted. Everyone's having problems with context windows and CC/etc. rapidly forgetting instructions.

I'm fullstack, I use AI for FE too. They've been able to do the screenshot trick for over a year now. I know it's pretty good at making a page, but the code is usually rubbish and you'll have a bunch of totally unnecessary useEffect, useMemo and styling in that page that it's picked up from its training data. Do you have any idea what all the useEffect() and useMemo() it's littered all over your new page do? I can guarantee almost all of them are wrong or unnecessary.

I use that page you one-shotted as a starting point, it's not production-grade code. The final thing will look nothing like it. Good for solving the blank page problem for me though.

React is hard even for humans to understand :) In my case the LLM can actually make something that works, even if it's ugly and inefficient. I can't do even that, my brain just doesn't speak React, all the overlapping effects and memos and whatever else magic just fries my brain.

That matches my experience with LLM-aided PRs - if you see a useEffect() with an obvious LLM line-comment above it, it's 95% going to be either unneccessary or buggy (e.g. too-broad dependencies which cause lots of unwanted recomputes).

You can literally go look at some of antirez's PRs described here in this article. They're not seeing it because it's not there?

Honestly, what you're describing sounds like the older models. If you are getting these sorts of results with Opus 4.5 or 5.2-codex on high I would be very curious to see your prompts/workflow.

People have been saying "Oh use glorp 3.835 and those problems don't happen anymore" for about 3 years at this point. It's always the fact you're not using the latest model that's the problem.

I agree. I've seen people insist moving to a newer model or fine tuning will make the output more clever, "trust me", sometimes without providing any evidence of before and after for the specific use case. One LLM project I saw released was prettymuch useless, but it wasn't the use case or the architectural limitations that were the problem, nope the next thing on the roadmap was "fixing" it by plugging in a better LLM.

> You ask it the right niche thing and it can only pump out a few variations of the same code, that's clearly someone else's code stolen almost verbatim.

There are only so many ways to express the same idea. Even clean room engineers write incidentally identical code to the source sometimes.

There was an example on here recently where an AI PR to an open source literally had someone else's name in the comments in the code, and included their license.

That's the level of tell-tale that's its just stealing code and modifying a couple of variable names.

For me personally, the code I've seen might be written in a slightly weird style, or have strange, not applicable to the question, additions.

They're so obviously not "clean room" code or incredibly generic, they're the opposite, they're incredibly specific.

How does he have a vested interest in the success of these tools? He doesn't work for an AI company. Why must he have some shady ulterior motive rather than just honestly believing the thing they are stated? Yes, he blogs a lot about AI, but don't you have the cart profoundly before the horse if you are asserting that's a "vested interest"? He was free to blog about whatever he wants. Why would he fervently start blogging about AI if he didn't earnestly believe it was an interesting topic to blog about?

> But it's still a statistical text prediction model

This is reductive to the point of absurdity. What other statistical text prediction model can make tool calls to CLI apps and web searches? It's like saying "a computer is nothing special -- it's just a bunch of wires stuck together"

> Why must he have some shady ulterior motive rather than just honestly believing the thing they are are stated?

I wouldn't say it's shady or even untoward. Simon writes prolifically and he seems quite genuinely interested in this. That he has attached his public persona, and what seems like basically all of his time from the last few years, to LLMs and their derivatives is still a vested interest. I wouldn't even say that's bad. Passion about technology is what drives many of us. But it still needs saying.

> This is reductive to the point of absurdity. What other statistical text prediction model can make tool calls to CLI apps and web searches?

It's just a fact that these things are statistical text prediction models. Sure, they're marvels, but they're not deterministic, nor are they reliable. They are like a slot machine with surprisingly good odds: pull the lever and you're almost guaranteed to get something, maybe a jackpot, maybe you'll lose those tokens. For many people it's cheap enough to just keep pulling the lever until they get what they want, or go bankrupt.

An interesting trend over the past year is that LLMs have learned how to prompt each other.

Back in ~2024 a lot of people were excited about having "LLMs write the prompt!" but I found the results to be really disappointing - they were full of things like "You are the world's best expert in marketing" which was superstitious junk.

As of 2025 I'm finding they actually do know how to prompt, which makes sense because there's a ton more information about good prompting approaches in the training data as opposed to a couple of years ago. This has unlocked some very interesting patterns, such as Claude Code prompting sub-agents to help it explore codebases without polluting the top level token window.

But learning to prompt is not the key skill in getting good results out of LLMs. The thing that matters most is having a robust model of what they can and cannot do. Asking an LLM "can you do X" is still the kind of thing I wouldn't trust them to answer in a useful way, because they're always constrained by training data that was only aware of their predecessors.

Unless we figure out how to make 1 billion+ tokens multimodal context windows (in a commercially viable way) and connect them to Google Docs/Slack/Notion/Zoom meetings/etc, I don't think it will simplify that much. Most of the work is adjusting your mental model to the fact that the agent is a stateless machine that starts from scratch every single time and has little-to-no knowledge besides what's in the code, so you have to be very specific about the context of the task in some ways.

It's different from assigning a task to a co-worker who already knows the business rules and cross-implications of the code in the real world. The agent can't see the broader picture of the stuff it's making, it can go from ignoring obvious (to a human that was present in the last planning meeting) edge cases to coding defensively against hundreds of edge cases that will never occur, if you don't add that to your prompt/context material.

So where’s all of this cutting edge amazing and flawless stuff you’ve built in a weekend that everybody else couldn’t because they were too dumb or slow or clueless?

- https://github.com/simonw/denobox is a new Python library that gives you the ability to run arbitrary JavaScript and WASM in a sandbox provided by Deno, because it turns out a Python library can depend on deno these days. I built that on my phone in bed yesterday morning.

- https://github.com/simonw/pwasm is a WebAssembly runtime written in pure Python with no dependencies, built by feeding Claude Code the official WASM specification along with its conformance test suite and having it hack away at that (again via my phone) to get as many of the tests to pass as possible. It's pretty slow and not really useful yet but it's certainly interesting.

- https://github.com/datasette/datasette-transactions is a Datasette plugin which provides a JSON API for starting a SQLite transaction, running multiple queries within it and then executing or rolling back that transaction. I built that one on my phone on a BART (SF Bay Area metro) trip.

- https://github.com/simonw/micro-javascript is a pure Python, no dependency JavaScript interpreter which started as a port of MicroQuickJS. Here's a demo of that one running in a browser https://simonw.github.io/micro-javascript/playground.html - that's my JavaScript interpreter running inside Python running in Pyodide in WebAssembly in your browser of choice, which I find inherently amusing.

All of those are from the past three weeks. Most of them were built on my phone while I was doing other things.

I am not at all an AI sceptic, but probably less impressed by what LLMs are capable of.

Looking at these projects, I have a few questions:

1. These seem to be fairly self-contained and well specified problems, which is the best case scenario for “vibe coding”. Do you have any examples of projects where the solution was somewhat vague and open-ended? If not, how do you think Claude Code or similar would perform?

2. Did you feel excited or energized by having an LLM implement these projects end-to-end? Personally, I find LLMs useful as a closely guided assistant, particularly to interactively explore the space of solutions. I also don’t feel energized at all by having it implement anything non-trivial end to end, outside of writing tests (and even then, not all types of tests!).

3. Do you think others would find these projects useful? In particular, if you vibe coded them, why couldn’t someone else do the same thing? And once these projects are picked up by future model training runs, they’ll probably be even easier to one shot, reducing the value even further.

Let me provide an example of what I mean by (2), at least in the context of hobbyist dev. I could have Claude Code vibe code a Gameboy emulator and it would probably do a fine job given that it’s a well specified problem that is likely well represented in its training data. But the process would neither be exciting nor energizing. I would rather spend hours gradually getting more and more working and experience the fruits of my labor (I did this already btw).

At $DAYJOB, I simply do not have confidence in an LLM doing anything non-trivial end to end. Besides, the complexity remains in defining the requirements and constraints, designing the solution, gaining consensus, and devising a plan for implementation. The goal would be for the LLM to pick up discrete, well defined chunks of work.

I have another project in the works in Go which is proving to be a ton of fun from a software design perspective, but it's not ready for outside eyes just yet.

"Did you feel excited or energized by having an LLM implement these projects end-to-end"

I'm enjoying myself so much right now. My BART rides have never been this entertaining before!

"Do you think others would find these projects useful? In particular, if you vibe coded them, why couldn’t someone else do the same thing?"

I don't think many developers have the combined taste and knowledge necessary to spin up Denobox or django-transactions. They both solve problems that I'm very confident need solving, but I expect to have to explain why those matter in some detail to all but a very small group of people who share my particular interests.

The other two are pretty standard - I suggest anyone who wants to learn more about JavaScript interpreters or WASM runtimes try something similar in the language of choice as a learning exercise.

Thanks for sharing, that project indeed looks useful.

> My BART rides have never been this entertaining before!

Not clear if this is snark, but if vibe coding on a train ride is actually energizing, then good for you haha.

> I don't think many developers have the combined taste and knowledge necessary to spin up Denobox or django-transactions.

Perhaps, but that’s just for now. What do you do when your “taste” no longer makes a difference? In other words, looking at the bigger picture, do you like where the field is going?

> I suggest anyone who wants to learn more about JavaScript interpreters or WASM runtimes try something similar in the language of choice as a learning exercise.

Agreed, but depending on learning style, vibe coding such a project might not teach you anything new at all :)

> I have another project in the works in Go which is proving to be a ton of fun from a software design perspective, but it's not ready for outside eyes just yet.

As a long-time user of the language I'm happy see that Go seems to be excellent for LLM agent development. The language is simple, there's only one way to do loops etc. It hasn't changed that much syntax wise (I think `any` is the only thing that LLMs miss).

Gofmt (or goimports) makes sure all code looks the same, there are VERY robust linters and a built-in testing framework so the LLM only needs to know one. And the code won't even compile if there are unused variables or other cruft.

It might be boring or verbose, but it's also very predictable and simple. All things LLMs like :D

Based on those, it seems you are not actually using them to create big codebases from scratch, but rather for problems that would normally take quite a while, not because they are inherently difficult to implement, but because you would normally have to spend considerable time on the finicky implementation details.

I think that's the reason why LLMs work so well for some like you, and generate slop for others, because if you let them alone with projects that require opinionated code and actual decision making they most often don't grasp the users intention well or worse misinterpret it so confidently that you end up with something with all the wrong opinions and decisions compounding path-dependently into the strangest and most useless slop.

"for problems that would normally take quite a while, not because they are inherently difficult to implement, but because you would normally have to spend considerable time on the finicky implementation details"

Yes, exactly! How amazing is it that we have technology now that lets us quickly build projects where we would normally have to spend considerable time on the finicky implementation details?

Pretty nice I guess. Cool even. Impressive! And I only say this , just in case, for someone else maybe, ehh—is that it? Because that’s totally fine with me, same experience actually funny that, really impressive tech btw! Very nice. Just, maybe, do the CEOs know that? When people talk of “not having to code anymore”—do they know that this is how it’s described by one of its most prominent champions today?

Not that I mind, of course. As you said: amazing!

Maybe someone just check in with the CEOs who were in the news recently talking about their work force…

> When people talk of “not having to code anymore”

You should reinterpret that as "not having to type the code out be hand any more". You still need a significant depth of coding knowledge and experience to get good results out of these things. You just don't need to type out every variable declaration and for loop yourself any more.

Every single tool or utility you have in the back of your head, you can just make it in a few hours of wall-clock time, minutes of your personal active time.

Like I wanted a tool that can summarise different sources quickly, took me ~3 hours to build it using llm + fragments + OpenAI API.

Now I can just go `q <url>` in my terminal and it'll summarise just about anything.

Then I built a similar tool that can download almost anything `dl <url>` will use yt-dlp, curl and various other tools depending on the domain to download the content.

Another lens is that many people either have terrible written communication skills, do not intuitively grasp how to describe a complex system design, or both. And yet, since everyone is a genius with 100% comprehensibility in their own mind, they simply aren't aware that the problem starts with them.

Well I think it also has to do with communication with LLMs being different to communication with humans. If you tell a developer "don't do busywork" they surely wouldn't say "Oh the repo looks like a trash dump, but no busywork so I'm not going to clean it up, quickly document that as canonical structure, then continue"

$200 to Anthropic, $20 to OpenAI, ~$10 in API fees for various other services, and I get GitHub Copilot in VS Code for free as an open source developer.

Mid-Senior in MENA. My monthly Salary is around 740 USD (Converted), and while I exaggerated that more than 50% figure, 200 USD/m is still a gigantic chunk of money to spend on something with dubious return on investment.

> gigantic chunk of money to spend on something with dubious return on investment.

There is nothing dubious about it. It’s providing verifiable value. Tasks that we would have set for developers or UX’ers last year are solved by it. At 1/50 of the cost, and with great scaling because the tasks are solved faster than you would even be able to explain them to a human and we can initiate 5-10 parallel tracks without having to onboard new people.

And sure it might not make sense to give a 200$/m AI tool to a worker you are paying 800$ but when we have devs that are paid 8000$/m then it’s great return on value to have one person being 10 times as productive at 8200$ instead of spending 80000$/m on ten developers just to be able to say we are doing authentic AI free artisanal software development.

You keep saying you "built" this or that, but did you really?

Of course I don't know for sure if you had any substantial input other than writing a few paragraphs of prompt text and sending Claude some links, because I didn't witness your workflow there. But I think this is kind of what irks some people including myself.

What's stopping me from "building" something similar also? Maybe I won't be as fast as you since you seem to be more experienced with these tools, but at the end of the day, would you be able to describe in detail what got built without you asking Claude about it? If you don't know anything about what you built other than just prompting an AI, in my opinion you didn't actually "build" anything -- Claude did.

Based on your comment history it seems like you're making an assumption about my intentions here, but I'll bite anyway.

When you build a table, you use a tool as a means to an end, i.e. you use the tool to cut and shape, but you are fully in control and engaged in the process. When you prompt an LLM, you tell it what to do and it does something for you. How is that not the same as telling someone else to build the table for you? You don't say "I built the table", you say "I got someone else to build the table for me."

I think it's great that simonw responded with some information on his process, that definitely helps provide perspective on how he engaged with Claude to make these projects.

People are under zero obligation to release their work to the public. Simon actually publishes and writes about a remarkable amount of the side projects he builds with AI.

The rest of us just build tons of cool stuff for personal use or for $JOB. Releasing stuff to the public is, in general, a massive amount of extra work for very little benefit. There are loads of FOSS maintainers trapped spending as much time managing their communities as they do their actual projects and many of us just don't have time for that.

> The rest of us just build tons of cool stuff for personal use or for $JOB. Releasing stuff to the public is, in general, a massive amount of extra work for very little benefit. There are loads of FOSS maintainers trapped spending as much time managing their communities as they do their actual projects and many of us just don't have time for that.

I wouldn't worry about this.

There are many examples of people sharing a project they've used LLMs to help write, and the result was not a huge amount of attention & expectation of burden.

Perhaps "I don't share it because I'm worried people will love it too much" even suggests the opposite: you can concretely demonstrate the kinds of things you've been able to build using LLMs.

> This is such a tired response at this point.

Lack of specificity & concrete examples frequently mean all that's left for discussion is emotion for hype and anti-hype, though.

In this thread, the discussion was:

pro: use LLMs or get left behind
conserve: okay, I'll start using LLMs when they're good
pro: no no they won't be that good, it takes effort to get to use them
conserve: do you have any examples?
pro: why should we have to share examples?

I like LLMs. But making big claims while being reticent about concrete claims and demonstrations is irksome.

The response may be tired when asked in this personal way, but in general, it's a fair question. Nobody is forced to share their work. But with all the high praises, we'd expect to see at least some uptick in the software world. But there is no surge in open source projects. No surge in app store entries. And for the bigger companies claiming high GenAI use, they're not iterating faster or building more. They are continually removing features and their software is getting worse, slower, less robust, and less secure.

Software quality has been on a step downwards curve as far as quality and capabilities are concerned, for years before LLM coding had its breakthrough. For all the promises I'd have expected to, three years later, at least notice the downward trajectory easing off. But it hasn't been happening.

I find it increasingly confusing that some people seem to believe, that other people not subjecting themselves to this continued interrogation, gives any credence to their position.

People seem to believe that there is a burden of proof. There is not. What do I care if you are on board?

I don't know what could change your mind, but of course the answer is "nothing" as long as you aer not open to it. Just look around. There is so much stuff, from so many credible people in all domains. If you can't find anything that is convincing or at least interesting to you, you are simply not looking.

> People seem to believe that there is a burden of proof. There is not. What do I care if you are on board?

The burden of proof rests on those making the positive claim. You say you don't care if others get on board, but a) clearly a lot of others do (case in point: the linked article) and b) a quick check of your posts in this very thread shows that you are indeed making positive claims about the merits of LLM assisted software development.

Without enough adoption expect some companies you are a client of to increase prices more, or close entirely down the road, due to insufficient cash inflow.

So, you would care, if you want to continue to use these tools and see them evolve, instead of seeing the bubble pop.

This is baffling. Why would you make the claim if you do not care if we are on board? Who are you talking to if not exactly those who you care to convince?

"Web Pipe is an experimental DSL and Rust runtime for building web apps via composable JSON pipelines, featuring native integration of GraphQL, SQL, and jq, an embedded BDD testing framework, and a sophisticated Language Server."

I've been working at quite a clip for a solo developer who is building a new language with a full featured set of tooling.

I'd like to think that the approach to building the BDD-testing framework directly into the language itself and having the test runner using the production request handlers is at least somewhat novel!

GET /hello/:world
|> jq: `{ world: .params.world }`
|> handlebars: `<p>hello, {{world}}</p>`
describe "hello, world"
it "calls the route"
let world = "world"
when calling GET /hello/{{world}}
then status is 200
and selector `p` text equals "hello, {{world}}"

I'm married with two young kids and I have a full-time job. Before these tools there was no way I could build all of these experiments with such limited resources.

Asking Simon Willis “where is all this amazing stuff you’ve built” is crazy. I assume you didn’t know who you were responding to. Not only is he insanely productive, but he’s also incredibly open and sharing about his work and his work gets posted to hackernews constantly. It was the top most upvoted blog of 2024 by almost twice the as much as the next runner up.

All of the linked apps look trivial to me.
Also, the first one, the UI has no feedback once you click the answer (plus some questions don't really make sense as they have the answer in them).
There is more on the website, so there could be something interesting, but I'm having trouble finding it among all the noise.
Not saying simple apps have no value. Even simple throwaway UIs can have value, especially if you develop them quickly.

The first one is basically some glue code using pipes.
The ones I'd say are not trivial are the VMs in Python.
However, I'd say they are entirely useless, and not too complex either (although somewhat tedious to implement).

I feel like I'm being punked, being told that this "bullish vs bearish flash card" thing and this "here's your user agent, something people have been doing for thirty years" thing, are "cool stuff". This guy seriously needed AI to make those?

I can't gauge the other two since I don't use those things, so maybe they are cool, idk.

llmslave3 appears to have deliberately picked the least interesting from my HTML+JavaScript tools collection here. This post describes a bunch of much more interesting ones: https://simonwillison.net/2025/Dec/10/html-tools/

No, I am pointing out the hypocrisy in demanding evidence of production results in a derisive manner whenever someone mentions a productivity boost with AI.

To some extend it's an understandable ask, but obviously even with a decent productivity boost side projects still require a lot of time and effort before a possible public release.

Strongly disagree. Claude Code is the most intuitive technology I've ever used-- way easier than learning to use even VS Code for example. It doesn't even take weeks. Maybe a day or two to get the hang of it and you're off to the races.

The difference is AI tooling lies to you. Day 0 you think it's perfect but the more you use ai tools you realize using them wrong can give you gnarly bugs.

It took me a couple of days to find the right level of detail to prompt it. Too high level, and the codebase gets away from me/the tooling goes off the rails. Too low level, and I may as well do it myself. Maybe also learn the sorts of things Claude Code isn't good at yet. But once I got in the groove it was very easy from there. I think the whole process took 2-3 days.

I actually don't use it in the terminal, I use the vs code extension. It's a better experience (bringing up the file being edited, nicer diffs, etc.) But both are trivial to pick up.

The core of your argument is that using LLMs is a skill that takes a significant amount of time to master. I'm not going to argue against that (although I have some doubts) because I think it's ultimately irrelevant. The question isn't "is prompting a skill that you'll need to be an effective software developer in the future" but "what other skills will you need to do so", and regardless of the answer you don't need to start adopting LLMs right away.

Maybe AI gets good enough at writing code that it's users' knowledge of computer science and software development becomes irrelevant. In that case, approximately everyone on this site is just screwed. We're all in the business of selling that specialized knowledge, and if it's no longer required then companies aren't going to pay us to operate the AI, they're going to pay PMs, middle managers, executives, etc. But even that won't be particularly workable long term, because all their customers will realize they no longer need to pay the companies for software either. In this world, the price of software goes to zero (and hosting likely gets significantly more commoditized than it is now). Any time you put into learning to use LLMs for software development doesn't help you keep making money selling software, and actually stops you from picking up a new career.

If, on the other hand, CS and software engineering knowledge is still needed, companies will have to keep/restart hiring or training new developers. In terms of experience using AI, it is impossible for anyone to have less experience than these new developers. We will, however, have much more experience and knowledge of the aforementioned non-LLM skills that we're assuming (in this scenario) are still necessary for the job. In this scenario you might be better off if you'd started learning to prompt a bit earlier, but you'll still be fine if you didn't.

While intuition takes a while, I think it can be learned in less than a month or two.

This has been my experience. When something gets good enough, someone will create some really good resource on it. Allowing the dust to settle, to me is a more efficient strategy than constantly trying to “keep up”. Maybe also not waiting too long to do so.

This wouldn’t work of course if a person was trying to be some AI thought leader.

I'd say that it's a different type of learning process, where even a good resource doesn't help as much as it would with a traditional programming language. Sort of like you can't get very good at writing by just reading a ton of instructional books about it.

What's the impressive thing that can convince me it's equivalent, or better than anything created before, or without it?

I understand you've produced a lot of things, and that your clout (which depends on the AI ferver) is based largely because of how refined a workflow you've invented. But I want to see the product, rather than the hype.

Make me say; I wish I was good enough to create this!

Without that, all I can see is the cost, or the negative impact.

edit: I've read some of your other posts, and for my question, I'd like to encourage you to pick only one. Don't use the scatter shot approach that LLMs love, giving plenty of examples, hoping I'll ignore the noise for the single that sounds interesting.

Pick only one. What project have you created that you're truly proud of?

I'll go first, (even though it's unfinished): Verse

The point wasn't to level set, or for it to feel like I'm promoting it. Only so that I couldn't back out later, or not have any skin in the game. But if you really couldn't figure out which I meant. It's the one on my github, and also hosting on srctree, which I link to from the site in my profile, and from my github.

I don't see how your position is compatible with the constant hype about the ever-growing capabilities of LLMs. Either they are improving rapidly, and your intuition keeps getting less and less valuable, or they aren't improving.

They're improving rapidly, which means your intuition needs to be constantly updated.

Things that they couldn't do six months go might now be things that they can do - and knowing they couldn't do X six months ago is useful because it helps systematize your explorations.

A key skill here is to know what they can do, what they can't do and what the current incantations are that unlock interesting capabilities.

A couple I've learned in the past week:

1. Don't give Claude Code a URL to some code and tell it to use that, because by default it will use its WebFetch tool but that runs an extra summarization layer (as a prompt injection defense) which loses details. Telling it to use curl sometimes works but a guaranteed trick is to have it git clone the relevant repo to /tmp and look at the code there instead.

2. Telling Claude Code "use red/green TDD" is a quick to type shortcut that will cause it to write tests first, run them and watch them fail, then implement the feature and run the test again. This is a wildly effective technique for getting code that works properly while avoiding untested junk code that isn't needed.

Now multiply those learnings by three years. Sure, the stuff I figure out in 2023 mostly doesn't apply today - but the skills I developed in learning how to test and iterate on my intuitions from then still count and still keep compounding.

The idea that you don't need to learn these things because they'll get better to the point that they can just perfectly figure out what you need is AGI science fiction. I think it's safe to ignore.

Personally I think this is an extreme waste of time. Every week you're learning something new that is already outdated the next week. You're telling me AI can write complex code but isn't able to figure out how to properly guide the user into writing usable prompts?

A somewhat intelligent junior will dive deep for one week and be on the same knowledge level as you in roughly 3 years.

No matter how good AI gets we will never be in a situation where a person with poor communication skills will be able to use it as effectively as someone who's communication skills are razor sharp.

But the examples you've posted have nothing to do with communication skills, they're just hacks to get particular tools to work better for you, and those will change whenever the next model/service decides to do things differently.

I'm generally skeptical of Simon's specific line of argument here, but I'm inclined to agree with the point about communication skill.

In particular, the idea of saying something like "use red/green TDD" is an expression of communication skill (and also, of course, awareness of software methodology jargon).

Ehhh, I don't know. "Communication" is for sapients. I'd call that "knowing the right keywords".

And if the hype is right, why would you need to know any of them? I've seen people unironically suggest telling the LLM to "write good code", which seems even easier.

I sympathize with your view on a philosophical level, but the consequence is really a meaningless semantic argument. The point is that prompting the AI with words that you'd actually use when asking a human to perform the task, generally works better than trying to "guess the password" that will magically get optimum performance out of the AI.

Telling an intern to care about code quality might actually cause an intern who hasn't been caring about code quality to care a little bit more. But it isn't going to help the intern understand the intended purpose of the software.

I'm not making a semantic argument, I'm making a practical one.

> prompting the AI with words that you'd actually use when asking a human to perform the task, generally works better

Ok, but why would you assume that would remain true? There's no reason it should.

As AI starts training on code made by AI, you're going to get feedback loops as more and more of the training data is going to be structured alike and the older handwritten code starts going stale.

If you're not writing the code and you don't care about the structure, why would you ever need to learn any of the jargon? You'd just copy and paste prompts out of Github until it works or just say "hey Alexa, make me an app like this other app".

Why do you bother with all this discussion? Like, I get it the first x times for some low x, it's fun to have the discussion. But after a while, aren't you just tired of the people who keep pushing back? You are right, they are wrong. It's obvious to anyone who has put the effort in.

It's also useful for figuring out what I think and how best to express that. Sometimes I get really great replies too - I compared ethical LLM objections to veganism today on Lobste.rs and got a superb reply explaining why the comparison doesn't hold: https://lobste.rs/s/cmsfbu/don_t_fall_into_anti_ai_hype#c_oc...

Yes and no. Knowing the terminology is a short-cut to make the LLM use the correct part of its "brain".

Like when working with video, if you use "timecode" instead of "timestamp", it'll use the video production part of the vector memory more. Video production people always talk about "timecodes", not "timestamps".

You can also explain the idea of red/green testing the long way without mentioning any of the keywords. It might work, but just knowing you can say "use red/green testing" is a magic shortcut to the correct result.

Thus: working with LLMs is a skill, but also an ever-changing skill.

I feel like both of these examples are insights that won't be relevant in a year.

I agree that CC becoming omniscient is science fiction, but the goal of these interfaces is to make LLM-based coding more accessible. Any strategies we adopt to mitigate bad outcomes are destined to become part of the platform, no?

I've been coding with LLMs for maybe 3 years now. Obviously a dev who's experienced with the tools will be more adept than one who's not, but if someone started using CC today, I don't think it would take them anywhere near that time to get to a similar level of competency.

I base part of my skepticism about that on the huge number of people who seem to be unable to get good results out of LLMs for code, and who appear to think that's a commentary on the quality of the LLMs themselves as opposed to their own abilities to use them.

Maybe it’s just two different ways to reach the same result. You need to spend time to be great at prompting to get high-quality code from LLM’s, which might just be equivalent to the fact you need to spend time to write high-quality code without LLM’s too.

From where I’m standing, I don’t see any massive difference on overall productivity between anyone all in on vibe coding than those who aren’t. There’s not more features, higher quality, etc from teams/companies out there than before on any high-level metrics/observations. Maybe it will come, but there’s also no evidence it will.

I do, however, see great gains within certain specific tasks using LLM’s. Smaller scope code gen, rubber ducking, etc. But this seems much less difficult to get good at using (and I hope for tooling that help facilitate the specific types of use cases) and on the whole amounts to marginal gains. It seems fine to be a few years late to catch up, worst case.

I don't disagree, knowing how to use the tools is important. But I wanted to add that great prompting skill nowadays are far far less necessary for top-tier models that it was years ago. If I'm clear about what I want and how I want it to behave, Claude Opus 4.5 almost always nails it first time. The "extra" that I do often, that maybe newcomers don't, is to setup a system where the LLM can easily check the results of its changes (verbose logs in terminal and, in web, verbose logs in console and playwright).

So far every new AI product and even model update has required me to relearn how to get decent results out of them. I'm honestly kind of sick of having to adjust my work flow every time.

The intuition just doesn't hold. The LLM gets trained and retrained by other LLM users so what works for me suddenly changes when the LLM models refresh.

LLMs have only gotten easier to learn and catch up on over the years. In fact, most LLM companies seem to optimise for getting started quickly over getting good results consistently. There may come a moment when the foundations solidify and not bothering with LLMs may put you behind the curve, but we're not there yet, and with the literally impossible funding and resources OpenAI is claiming they need, it may never come.

Why can't both be true at the same time? Maybe their problems are more complex than yours. Why do you assume it's a skill issue and ignore the contextual variables?

On the rare occasions that I can convince them to share the details of the problems they are tackling and the exact prompts they are using it becomes very clear that they haven't learned how to use the tools yet.

I'm kind of curious about the things you're seeing since I find the best way is to have them come up with a plan for the work they're about to do and then make sure they actually finish it because they like to skip stuff if it requires too much effort.

I mean, I just think of them like a dog that'll get distracted and go off doing some other random thing if you don't supervise them enough and you certainly don't want to trust them to guard your sandwich.

I think I'm also very good at getting great results out of coding agents and LLMs, and I disagree pretty heavily with you.

It is just way easier for someone to get up to speed today than it was a year ago. Partly because capabilities have gotten better and much of what was learned 6+ months ago no longer needs to be learned. But also partly because there is just much more information out there about how to get good results, you might have coworkers or friends you can talk to who have gotten good results, you can read comments on HN or blog posts from people who have gotten good results, etc.

I mean, ok, I don't think someone can fully catch up in a few weeks. I'll grant that for sure. But I think they can get up to speed much faster than they could have a year ago.

Of course, they will have to put in the effort at that time. And people who have been putting it off may be less likely to ever do that. So I think people will get left behind. But I think the alarm to raise is more, "hey, it's a deep topic and you're going to have to put in the effort" rather than "you better start now or else it's gonna be too late".

I learned Django 15 years after its inception. After 5 years of experience I'm probably not too far behind someone doing the exact same work as me but for 15 years.

Or would you say people shouldn't learn Django now? As it's useless as they're already far behind? They shouldn't study computer science, as it will be too late?

Every profession have new people continuously entering the workforce, that quickly get up to speed on whatever is in vogue.

Honestly, what you've spent years learning and experimenting with, someone else will be able to learn in months. People will figure out the best ways of using these tools after lots of attempts, and that distilled knowledge will be transferred quickly to others. This is surely painful to hear for those having spent years in the trenches, and is perhaps why you refuse to acknowledge it, but I think it's true.

I would not say that about a framework like Django - though I would encourage people not to under-invest in understanding web fundamentals since once you have those Django, Rails, Next.js etc are all quick to pick up.

I would say that about LLMs.

That's why I'm ringing the alarm bells here. LLM skills are not the same as framework or library usage skills. They aren't clearly documented or predictable - they're really weird!

If you assume learning to use coding agents is the same category of challenge as learning to use something like Django you'll get burned by that assumption.

But what makes it impossible to catch up? Does it matter if I wait a year and then start? It's not a linear thing, at some point I will catch up with those that started before me, as the logarithmic curve of learning flattens out. And then, why does it matter that I started a year later?

> Using this stuff well is a deep topic. These things can be applied in so many different ways, and to so many different projects. The best asset you can develop is an intuition

You're basically saying that using LLMs is like using magic. Telling people to use intuition is basically telling that i don't know how it works and why, but works for me sometimes.

That's why we programmers hate it - we have safe space where there's no intuition - namely programming languages & runtimes with deterministic behavior. And we're shoehorned back into mess of magic/intuition and wishfullthinking.

(yes, i try llm, i have some results, i'm frustrated mostly by people AI-slopping _everything_ around me)

I am eternally frustrated that "intuition" is the key skill people need to work effectively with LLMs, because it's something I can't teach people! If I could figure out how to download my intuition into other people's heads I would do that.

Instead I have to convince people that intuition is key, and the only way to get it is to invest in experimenting.

It's like any other power tool. It requires skill to use it safely and efficiently.

Anyone can use a band saw to cut things. Then go look what Jimmy DiResta makes with one and you see the difference.

The chance of an inexperienced person cutting off their finger with a bandsaw is also way over zero, there are things you should not and must not do with it. As with any power tool.

I can't buy it because for many people like you it's always the other that uses the tools wrong, proving the contrary for skeptics that keep getting bad results from llms it simply is impossible with this narrative as the base of the discourse, eg "you're not using it well". I don't even get why you need to praise yourself so much being really good at using these tools, if not for building some tech influencer status around here... same thing I believe antirez is trying to do(who knows why)

Have you considered that maybe you aren't using it well? It's something that can and should be learned. It's a tool, and you can't expect to get the most out of a tool without really learning how to use it.

I've had this conversation with a few people so far, and I've offered to personally walk through a project of their choosing with them. Everyone who has done this has changed their perspective. You may not be convinced it will change the world, but if you approach it with an open mind and take the time to learn how to best use it, I'm 100% sure you will see that it has so much potential.

There are tons of youtube videos and online tutorials if you really want to learn.

> Have you considered that maybe you aren't using it well?

Here we go, as I said, and again and again and again it's always out fault we're not using well. It is impossible to counter argument. Btw to reply to your question, yes many times and proved to be useful in very small specialized tasks and a couple of migrations. I really like how LLMs are helping me in my day to day, but still so far away from all this astroturfing

It might be now, but the intent of these tools is clearly not to have to learn a bunch of work arounds to get the tool to do what you want.

If these tools do improve, that inefficiency would presumably reduce, or go away entirely, which means you wouldn't see an advantage to your head start.

What are your tips? Any resources you would recommend? I use Claude code and all the chat bots, but my background isn't programming, so I sometimes feel like I'm just swimming around.

I've been building Ai apps since gpt 3 so 5 years now.

The pro AI people don't understand what quadratic attention means and the anti-ai people don't understand how much information can be contained in a tb of weights.

At the end of the day both will be hugely disappointed.

>The best asset you can develop is an intuition for what works and what doesn't, and getting that intuition requires months if not years of personal experimentation.

Intuition does not translate between models. Whatever you think dense llms were good at deepseek completely upended it in an afternoon. The difference between major revisions of model families is substantial enough that intuition is a drawback not an asset.

I've so far found that intuition travels between models of a similar generation remarkably well. The conformance suite trick (find a 9,200 test existing conformance suite and tell an agent to build a fresh implementation that passes all those tests) I first found with GPT-5.2 turned out to work exactly as well against Claude Opus 4.5, for example.

To save anyone else the click, this is the paper "On The Computational Complexity of Self-Attention" from September 2022, with authors from NYU and Microsoft.

It argues that the self-attention mechanism in transformers works by having every token "attend to" every other token in a sequence, which is quadratic - n^2 against input - which should limit the total context length available to models.

This would explain why the top models have been stuck at 1 million tokens since Gemini 1.5 in February 2024 (there has been a 2 million token Gemini but it's not in wide use, and Meta claimed their Llama 4 Scout could do 10 million but I don't know of anyone who's seen that actually work.)

My counter-argument here is that Claude Opus 4.5 has a comparatively tiny 200,000 token window which turns out to work incredibly well for the kinds of coding problems we're throwing at it, when accompanied by a cleverly designed harness such as Claude Code. So this limit from 2022 has been less "disappointing" than people may have expected.

The quadratic attention problem seems to be largely solved by practical algorithmic improvements. (Iterations on flash attention, etc.)

What's practically limiting context size IME is that results seem to get "muddy" and get off track when you have a giant context size. For a single-topic long session, I imagine you get a large number of places in the context which may be good matches for a given query, leading to ambiguous results.

I'm also not sure how much work is being put into reinforcement in extremely large context inference, as it's presumably quite expensive to do and hard to reliably test.

Indeed, filling the adversitsed context more than 1/4 full is a bad idea in general. 50k tokens is a fair bit, but works out to between 1 and 10k lines of code.

Perfect for a demo or work on a single self contained file.

Disastrous for a large code base with logic scattered all throughout it.

Right. It’s not practical to apply AI tools as they are today to existing, complex code bases and get reliable results.

Greenfield is easy (but it always was). Working on well-organised modules that are self contained and cleanly designed is easy - but that always was, too.

> Using this stuff well is a deep topic. These things can be applied in so many different ways, and to so many different projects. The best asset you can develop is an intuition for what works and what doesn't, and getting that intuition requires months if not years of personal experimentation.

You feel that way because it took you years or months to reach that point. But after reaching that point, do you really think that it's equally—if not more—difficult to put what you learned into words compared to, let's say, programming or engineering?

See, the thing about these tools is that they're designed to be operated via natural language, which is something most people (with a certain level of education) are quite comparable to each other at; consequently, the skill ceiling is considerably lower compared to something like programming. I am not saying there's no variance in people's ability to articulate, but that the variance is considerably less than what we get when comparing people's ability to write code or solve engineering problems.

So, whatever you learned by trial and error was just different ways or methods to get around the imperfections of the existing LLMs—not ways to use them skillfully according to their design goals. Their design goal is to achieve whatever task is given to them, as long as the intent is clear. These workarounds and tricks that you learned aren't something you build an intuition for. What you build an intuition for is finding new workarounds, but once you've found them, they're quite concrete and easy to describe to someone else who can simply use them to achieve the same results as you.

Tools that are designed to be operable via natural language aren't designed to be more thorough—it's actually the opposite. If you want more control, you have programming languages and search engines; thoroughness is where you get that high skill ceiling. The skill ceiling for using these tools is going to get narrower and narrower. The workarounds that you figure out may take skill to discover, but they don't take much skill to replicate.

If you share your "tips and tricks" with someone, then yeah, it will take them a week to start getting the same results as you because the skill ceiling is low and the workarounds are concrete/require less thinking.

> I don't think you can just catch up in a few weeks, and I do think that the risk of falling behind isn't being taken seriously enough by much of the developer population.

This is nonsense.

This field moves so fast the things you did more than a year ago aren't relevant anymore.

Claude code came out last year.

Anyone using random shit from before that is not using it any more. It is completely obsolete in all but a handful of cases.

To make matters worse “intuition” about models is wasted learning, because they change, significantly, often.

Stop spreading FUD.

You can be significantly less harmful to people who are trying to learn by sharing what you actually do instead of nebulously hand waving about magical BS.

Dear readers: ignore this irritating post.

Go and watch Armin Ronacher on youtube if you want to see what a real developer doing this looks like, and why its hard.

You're accusing me of spreading harmful advice here, when you're the one telling people that they don't need to worry about not investing in their skills because "This field moves so fast the things you did more than a year ago aren't relevant anymore."

One of us is right here. I hope for your sake and the people that listen to you that it's you. I don't think it is.

You're making wild claims, and absolutely failing to back them up with evidence.

That is FUD.

People should invest, they should try things. …but its far faaaaaar less clear cut that dropping everything and focusing on AI right now is so absolutely important.

The difference between what you get prompting and a totally naïve user of claude code gets is marginal.

People are not being left behind if they try it a bit and find its ok, not great, and come back later.

It is not a deep topic.

writing AI tools is a deep topic, but most people arent doing that.

Youre in the wrong here.

Stop making people scared.

I quote antirez here, since you clearly arent interested in listening to me:

> I have a single suggestion for you, my friend. Whatever you believe about what the Right Thing should be, you can't control it by refusing what is happening right now.

> Skipping AI is not going to help you or your career. Think about it. Test these new tools, with care, with weeks of work, not in a five minutes test where you can just reinforce your own beliefs.

> Find a way to multiply yourself, and if it does not work for you, try again every few months.

I'm currently sounding the alarm, because I sincerely believe that the the "it doesn't work, and even if it did you can catch up easily" message is no longer credible as of November 2025.

If I didn't sincerely believe that I wouldn't say it.

It true that you will always be able to catch up eventually - the industry has newbies entering it all the time and I believe they will continue to make it to point where they can contribute effectively.

But if I'm right and it does take 6-12 months for most developers to get proficient there's a real career risk involved now in listening to people who say it's all hype and no substance and you should keep on sitting it out.

You're right, it's difficult to get "left behind" when the tools and workflows are being constantly reinvented.

You'd be sage with your time just to keep a high-level view until workflows become stable and aren't advancing every few months.

The time to consider mastering a workflow is when a casual user of the "next release" wouldn't trivially supersede your capabilities.

Similarly we're still in the race to produce a "good enough" GenAI, so there isn't value in mastering anything right now unless you've already got a commercial need for it.

This all reminds me of a time when people were putting in serious effort to learn Palm Pilot's Graffiti handwriting recognition, only for the skill to be made redundant even before they were proficient at it.

I think that who says that you need to be accustomed to the current "tools" related to AI agents, is suffering from a horizon effect issue: these stuff will change continuously for some time, and the more they evolve, the less you need to fiddle with the details. However, the skill you need to have, is communication skills. You need to be able to express yourself and what matters for your project fast and well. Many programmers are not great at communication. In part this is a gift, something you develop at small age, and this will, I believe, kinda change who is good at programming: good communicators / explorers may not have a edge VS very strong coders that are bad at explaining themselves. But a lot of it is attitude, IMHO. And practice.

> Many programmers are not great at communication.

This is true, but still shocking. Professional (working with others at least) developers basically live or die by their ability to communicate. If you're bad at communication, your entire team (and yourself) suffer, yet it seems like the "lone ranger" type of programmer is still somewhat praised and idealized. When trying to help some programmer friends with how they use LLMs, it becomes really clear how little they actually can communicate, and for some of them I'm slightly surprised they've been able to work with others at all.

An example the other day, some friend complained that the LLM they worked with was using the wrong library, and using the wrong color for some element, and surprised that the LLM wouldn't know it from the get go. Reading through the prompt, they never mentioned it once, and when asked about it, they thought "it should have been obvious" which yeah, to someone like you who worked for 2 years on this project that might be obvious, but for some with zero history and zero context about what you do? How you expect it to know this? Baffling sometimes.

People anthropomorphise LLMs, not understanding that they don't have "implied context" about things. They just go by the statistical average unless directed otherwise.

Having worked with offshore consultant teams where there are language and cultural barriers - and needing clear specs myself. I somehow just naturally "got" how much context to give the Agent.

People who have been working solo or with like-minded people all their career might have a harder time.

Yup. I'd take a gander than most complaints by people who have even used LLMs for long time can be resolved by "describe your thing in detail". LLM's are such a relief on my wrists that I often get tempted to write short prompts and pray that the LLM divines my thoughts. I always get much better results in a lot faster time when i just turn on the mic and have whisper transcribe a couple minutes of my speaking though.

I am using Google Antigravity for the same type of work you mention, such as many things and ideas I had over the years but I couldn't justify the time I needed to invest into them. Pretty non-trivial ideas and yet with a good problem definition communication skills I am getting unbelievable results. I am even intentionally sometimes being too vague in my problem definition to avoid introducing the bias to the model and the ride has been quite crazy so far. In 2 days I've implemented several substantial improvements that i had in my head for years.

The world changed for good and we will need to adapt. The bigger and more important question at this point isn't anymore if LLMs are good enough, for the ones who want to see, but, as you mention in your article, is what will happen to people who will get unemployed. There's a reality check for all of us.

My take: learning how to do LLM-assisted coding at a basic level gets you 80% of the returns, and takes about 30 minutes. It's a complete no-brainer.

Learning all of the advanced multi-agent worklows etc. etc... Maybe that gets you an extra 20%, but it costs a lot more time, and is more likely to change over time anyway. So maybe not very good ROI.

It seems like you're mostly focused on the tooling for actually directing the LLM but there's a whole host of other technology which becomes relevant re: building guardrails and handcuffs for your agent. For instance I've been doing a lot of contract testing lately. It's not new tech, not changing at a blistering pace, but now that generating mountains of code is cheap, techniques for dealing with those mountains are suddenly more necessary.

2. Build tools for the LLM, ones that are easy to use and don't spam stuff. Like give it tools to run tests that only return "Tests OK" if nothing failed, same with builds.

3. Look into /commands and Skills, both seem to be here to stay

Maybe a weekend of messing about and you'll be pretty well off compared to the vast masses who still copy/paste code out of ChatGPT to their editor.

It took me a few months of working with the agents to get really productive with it. The gains are significant. I write highly detailed specs (equiv multiple A4 pages) in markdown and dicate the agent hierarchy (which agent does what, who reports to who).

I've learned a lot of new things this year thanks to AI. It's true that the low levels skills with atrophy. The high level skills will grow though; my learning rate is the same, just at a much higher abstraction level; thus covering more subjects.

The main concern is the centralisation. The value I can get out of this thing currently well exceeds my income. AI companies are buying up all the chips. I worry we'll get something like the housing market where AI will be about 50% of our income.

We have to fight this centralisation at all costs!

This is something I think a lot of people don't seem to notice, or worry about, the moving of programming as a local task, to one that is controlled by big corporations, essentially turning programming into a subscription model, just like everything else, if you don't pay the subscription you will no longer be able to code i.e. PaaS (Programming as a Service). Obviously at the moment most programmers can still code without LLMs, but when autocomplete IDEs became main stream, it didn't take long before a large proportion of programmers couldn't program without an autocomplete IDE, I expect most new programmers coming in won't be able to "program" without a remote LLM.

That ignores the possibility that local inference gets good enough to run without a subscription on reasonably priced hardware.

I don't think that's too far away. Anthropic, OpenAI, etc. are pushing the idea that you need a subscription but if opensource tools get good enough they could easily become an expensive irrelivance.

My concern is that inference hardware is becoming more and more specialized and datacenter-only. It won’t be possible any longer to just throw in a beefy GPU (in fact we’re already past that point).

Yep, good point. If they don't make the hardware available for personal use, then we wouldn't be able to buy it even it could be used in a personal system.

There is that, but the way this usually works is that there is always a better closed service you have to pay for, and we see that with LLMs as well. Plus there is the fact that you currently need a very powerful machine to run these models at anywhere near the speed of the PaaS systems, and I'm not convinced we'll be able to do the Moore's law style jumps required to get that level of performance locally, not to mention the massive energy requirements, you can only go so small, and we are getting pretty close to the limit. Perhaps I'm wrong, but we don't see the jumps in processing power we used to see in the 80s and 90s, due to clock speed jumps, the clock speed of most CPUs has stayed pretty much the same for a long time. As LLMs are essentially probabilistic in nature, this does open up options not available to current deterministic CPU designs, so that might be an avenue which gets exploited to bring this to local development.

> there is always a better closed service you have to pay for

Always? I think that only holds for a certain amount of time (different for each sector) after which the open stuff is better.

I thought it was only true for dev tools, but I had to rethink it when I met a guy (not especially technical) who runs open source firmware on his insulin pump because the closed source stuff doesn't gives him as much control.

From some comments I read in this thread, costs could be around 100-500k USD to get anywhere near current frontier models. My concern is that the constant price reductions we saw in cost per transistor (either storage or logic) over the last ~three decades are over, and that the cost per transistor will only go up!

This is the most valid criticism. Theoretically in several years we may be able to run Opus quality coding models locally. If that doesn't happen then yes, it becomes a pay to play profession - which is not great.

The hardware needs to catch up I think. I asked ChatGPT (lol) how much it would cost to build a Deepseek server that runs at a reasonable speed and it quoted ~400k-800k(8-16 H100 + the rest of the server).

Guess we are still in the 1970s era of AI computing. We need to hope for a few more step changes or some breakthrough on model size.

The problem is that Moore's law is dead, silicon isn't advancing as fast as what we've envisioned in the past, we're experiencing all sorts of quantum tunneling effects in order to cram as much microstructure as possible into silicon, and R&D for manufacturing these chips are climbing at a rapid rate. There's a limit to how we can fight against Physics, and unless we discover a totally new paradigm to alleviate this issues (ex. optical computing?) we're going to experience diminishing returns at the end of the sigmoid-like tech advancement cycle.

You can run most open models (excluding kimi-k2) on hardware that costs anywhere from 45 - 85k (tbf, specced before the vram wars of late 2025 so +10k maybe?). 4-8 PRO6000s + all the other bits and pieces gives you a machine that you can host locally and run very capable models, at several quants (glm4.7, minimax2.1, devstral, dsv3, gpt-oss-120b, qwens, etc.), with enough speed and parallel sessions for a small team (of agents or humans).

I have found that using more REPLs and doing leetcodes/katas prevents the atrophy to be honest.

In fact, I'd say I code even better since I started doing one hour per day of a mixture of fun coding and algo quizzes while at work I mostly focus on writing a requirements plan and implementation plan later and then letting the AI cook while I review all the output multiple times from multiple angles.

Well, if you're programming without AI you need to understand what you're building too, lest you program yourself into a corner. Taking 3-5 minutes to speech-to-text an overview of why you want to build what exactly, using which general philosophies/tool seems like it should cost you almost zero extra time and brainpower

I thought this way for a while. I still do to a certain degree, but I'm starting to see the wisdom in hurrying off into the change.

The most advanced tooling today looks nothing like the tooling for writing software 3 years ago. We've got multi-agent orchestration with built in task and issue tracking, context management, and subagents now. There's a steep learning curve!

I'm not saying that everyone has to do it, as the tools are so nascent, but I think it is worthwhile to at least start understanding what the state of the art will look like in 12-24 months.

AI development is about planning, orchestration and high throughput validation. Those skills won't go away, the quality floor of model output will just rise over time.

If you listen to promises like that you're going get burned.

One of the key skills needed in working with LLMs is learning to ignore the hype and marketing and figure out what these things are actually capable of, as opposed to LinkedIn bluster and claims from CEOs who's net worth are tied to investor sentiment in their companies.

If someone spends more time talking about "AGI" then what they're actually building, filter that person out.

>One of the key skills needed in working with LLMs is learning to ignore the hype and marketing and figure out what these things are actually capable of

This is precisely what led me to realize that while they have some use for code review and analyzing docs, for coding purposes they are fairly useless.

The hypesters responses' to this assertion exclusively into 5 categories. Ive never heard a 6th.

I agree about skills actually, but it's also obvious that parent is making a very real point that you cannot just dismiss. For several years now and far short of wild AGI promises, the answer to literally every issue with casual or production AI has been something like "but the rate of model improvement.." or "but the tools and ecosystem will evolve.."

If you believe that uncritically about everything else, then you have to answer why agentic workflows or MCP or whatever is the one thing that it can't evolve to do for us. There's a logical contradiction here where you really can't have it both ways.

I’m not understanding your point… (and would be genuinely curious to)? the models and systems around them have evolved and gotten better (over the past few years for LLMs and decades for “AI” more broadly)

oh I think I do get your point now after a few rereads (correct if wrong but you’re saying it should keep getting better until there’s nothing for us to do). “AI”, and computer systems more broadly, are not and cannot be viable systems. they don’t have agency (ironically) to affect change in their environment (without humans in the loop). computer systems don’t exist/survive without people. all the human concerns around what/why remain, AI is just another tool in a long line of computer systems that make our lives easier/more efficient

AI Engineer to Software Engineer: Humans writing code is a waste of time, you can only hope to add value by designing agentic workflows

Prompt Engineer to AI Engineer: Designing agentic workflows is a waste of time, just pre/postfix whatever input you'd normally give to the agentic system with the request to "build or simulate an appropriate agentic workflow for this problem"

Nobody serious, like every single AI CEO out there? I mean I agree, nobody should be taking them seriously, yet we're fast on track for a global financial meltdown because of these fraudsters and their "non-serious" words.

OpenAI is going to get to AGI. And AGI should in minutes build a system that takes vague input and produces fully functioning product out of it. Isn't singularity being promised by them?

you’re just repeating the straw man. if you can’t think critically and just regurgitate every dumb thing you hear idk what to tell you. nobody serious thinks a “singularity” is coming. there’s not even a proper definition of “AGI”

your argument amounts to “some people said stupid shit one time and I took it seriously”

The idea, I think, is to gain experience with the loop of communicating ideas in natural language rather than code, and then reading the generated code and taking it as feedback.

It's not that different overall, I suppose, from the loop of thinking of an idea and then implementing it and running tests; but potentially very disorienting for some.

Early adopters get the advantage of only having to learn a trickle of new things every few weeks instead of everything all at once.

Part of the problem with things that iterate quickly is that iterations tend to reference previous versions. So, you try learning the new hotness (v261), but there are implied references to v254, v239, and v198. Then you realize, v1, v5, v48, v87, v138, v192, and v230 have cute identifiers that you aren't familiar with and are never explained anywhere. New concepts get introduced in v25, v50, v102, and v156 that later became foundational knowledge that is assumed to be understood by the reader and is never explained anywhere.

So, if you feel confident something will be the next hotness, it's usually best to be an early adopter, so you gain your knowledge slowly over years instead of having to cram when you need to pick it up.

> What I don't understand about this whole "get on board the AI train or get left behind" narrative, what advantage does an early adopter have for AI tools?

The ones pushing this narrative have either the following:

* Invested in AI companies (which they will never disclose until they IPO / acquired)

* Employees at AI companies that have stock options which they are effectively paid boosters around AGI nonsense.

* Mid-life crisis / paranoia that their identity as a programmer is being eroded and have to pivot to AI.

It is no different to the crypto web3 bubble of 2021. This time, it is even more obvious and now the grifters from crypto / tech are already "pivoting to ai". [0]

Web3 generated plenty of use if you're in on it. Pension funds, private investors, public companies, governments, gambling addicts, teenagers with more pocket money than sense, they've all moved billions into the pockets of Web3 grifters. You follow a tutorial on YouTube, spam the right places, maybe buy a few illegal ads, do a quick rugpull, and if you did your opsec right, you're now a millionaire. The major money sources have started to dry up (although the current American regime has been paid off by crypto companies so a Web3 revival might just happen).

With AI companies still selling services far below cost, it's only a matter of time before the money runs out and the true value of these tools will be tested.

> Pension funds, private investors, public companies

As someone who was at a large company that was dabbling in NFTs, there was no value apart from pure gambling. At the time that we were doing it, it was also too late, so it was just a jinormous

My issue with GenAI is the rampant copyright violation, and the effect it will have on the economy. Its also replacing all of the fun bits of the world that I inhabit.

At least with web3 it was mostly contained with in the BO infested basement that crypto bros inhabit. AI bollocks has infected half the world.

Comparing crypto and web3 scam with AI advancements is disingenuous at its best. I am a long time C and C++ systems programming engineer oriented at (sometimes novel) algorithmic design and high-performance large-scale systems operating at the scale of internet. I am specializing in low-level details that generally very small amount of engineers around the globe are familiar with. We can talk at the level of CPU microarchitectural details or memory bank conflicts or OS internals, and all the way up to the line of code we are writing. AI is the most transformative technology ever designed. I'd go that far and say that not even industrial revolution is going to be comparable to it. I have no stakes in AI.

I've used cursor and claude code both daily[0] within a month of their releases - i'm learning something new on how to work with and apply the tools almost every day.

I don't think it's a coincidence that some of the best developers[1] are using these tools and some openly advocating for them because it still requires core skills to get the most out of them

I can honestly say that building end-to-end products with claude code has made me a better developer, product designer, tester, code reviewer, systems architect, project manager, sysadmin etc. I've learned more in the past ~year than I ever have in my career.

[0] abandoned cursor late last year

[1] see Linus using antigravity, antirez in OP, Jared at bun, Charlie at uv/ruff, mitushiko, simonw et al

I started heavy usage in April 2025 (Codex CLI -> some Claude Code and trying other CLIs + a bit of Cursor -> Warp.dev -> Claude Code) and I’m still learning as well (and constantly trying to get more efficient)

(I had been using GitHub Copilot for 5+ years already, started as an early beta tested, but I don’t really consider that the same)

I like to say it’s like learning a programming language. it takes time, but you start pattern matching and knowing what works. it took me multiple attempts and a good amount of time to learn Rust, learning effective use of these tools is similar

I’ve also learned a ton across domains I otherwise wouldn’t have touched

> What I don't understand about this whole "get on board the AI train or get left behind" narrative, what advantage does an early adopter have for AI tools?

Replace that with anything and you will notice that people who are building startups in this area will want to bring the narrative like that as it usually highly increases the value of their companies. When narrative gets big enough, then big companies must follow - or they look like "lagging behind". Whether the current thing brings value or not. It is a fire that keeps feeding itself. In the end, when it gets big enough - we call it as bubble. Bubble that may explode. Or not.

Whether the end user gets actual value or not, is just side effect. But everyone wants to believe that that it brings value - otherwise they were foolish to jump in the train.

An ecosystem is being built around AI : Best prompting practices, mcps, skills, IDE integration, how to build a feedback loop so that LLM can test its output alone, plug to the outside world with browser extensions, etc...

For now i think people can still catch up quickly, but at the end of 2026 it's probably going to be a different story.

> Best prompting practices, mcps, skills, IDE integration, how to build a feedback loop so that LLM can test its output alone, plug to the outside world with browser extensions, etc...

Ah yes, an ecosystem that is fundamentally inherently built on probabilisitic quick sand and even with the "best prompting practices", you still get agents violating the basics of security and committing API keys when they were told not to. [0]

For example, what if your code (that the LLM hasn't reviewed yet) has a dumb feature in where it dumps environment variables to log output, and the LLM runs "./server --log debug-issue-144.log" and commits that log file as part of a larger piece of work you ask it to perform.

If you don't want a bad thing to happen, adding a deterministic check that prevents the bad thing to happen is a better strategy than prompting models or hoping that they'll get "smarter" in the future.

Part of why these things feel "not fit for purpose" is that they don't include the things Simon has spent three years learning? (I know someone else who's doing multi-LLM development where he uses job-specialty descriptions for each "team member" that lets them spend context on different aspects of the problem; it's a fascinating exercise to watch, but it feels even more like "if this is how the tools should be used, why don't they just work that way"?)

I have tons of examples of AI not committing secrets. this is one screenshot from twitter? I don’t think it makes your point

CPUs are billions of transistors. sometimes one fails and things still work. “probabilistic quicksand” isn’t the dig you think it is to people who know how this stuff works

> I have tons of examples of AI not committing secrets.

"Trust only me bro".

It takes 10 seconds to see the many examples of API keys + prompts on GitHub to verify that tweet. The issue with AI isn't limited to that tweet which demonstrates its probabilistic nature; Otherwise why do need a sandbox to run the agent in the first place?

Nevermind, we know why: Many [0] such [1] cases [2]

> CPUs are billions of transistors. sometimes one fails and things still work. “probabilistic quicksand” isn’t the dig you think it is to people who know how this stuff works

Except you just made a false equivalence. CPUs can be tested / verified transparently and even if it does go wrong, we know exactly why. Where as you can't explain why the LLM hallucinated or decided to delete your home folder because the way it predicts what it outputs is fundamentally stochastic.

you could find tons of API keys on GitHub before these “agentic” tools too. that was my point, one screenshot from twitter vs one anecdote from me. I don’t think either proves the point, but posting a screenshot from twitter like it’s proof of some widespread problem is what I was responding to (N=2, 1 vs 1)

my point is more “skill issue” than “trust me this never happens”

my point on CPUs is people who don’t understand LLMs talk like “hallucinations” are a real thing — LLMs are “deciding” to make stuff up rather than just predicting the next token. yes it’s probabilistic, so is practically everything else at scale. yet it works and here we are. can you really explain in detail how everything you use works? I’m guessing I can explain failure modes of agentic systems (and how to avoid them so you don’t look silly on twitter/github) and how neural networks work better than most people can explain the technology they use every day

> you could find tons of API keys on GitHub before these “agentic” tools too. that was my point, one screenshot from twitter vs one anecdote from me. I don’t think either proves the point, but posting a screenshot from twitter like it’s proof of some widespread problem is what I was responding to (N=2, 1 vs 1)

That doesn't refute the probabilistic nature of LLMs despite best prompting practices. In fact it emphasises it. More like your 1 anecdotal example vs my 20+ examples on GitHub.

My point tells you that not only it indeed does happen, but a previous old issue is now made even worse and more widespread, since we now have vibe-coders without security best practices assuming the agent should know better (when it doesn't).

> my point is more “skill issue” than “trust me this never happens”

So those that have this "skill issue" are also those who are prompting the AI differently then? Either way, this just inadvertently proves my whole point.

> yes it’s probabilistic, so is practically everything else at scale. yet it works and here we are.

The additional problem is can you explain why it went wrong as you scale the technology? CPUs circuit design go through formal verification and if a fault happens, we know exactly why; hence it is deterministic in design which makes them reliable.

LLMs are not and don't have this. Which is why OpenAI had to describe ChatGPT's misaligned behaviour as "sycophancy", but could not explain why it happened other than tweaking the hyper-parameters which got them that result.

So LLMs being fundamentally probabilistic and are hence, more unexplainable being the reason why you have the screenshot of vibe-coders who somehow prompted it wrong and the agent committed the keys.

Maybe that would never have happened to you, but it won't be the last time we see more of this happening on GitHub.

I was pointing out one screenshot from twitter isn’t proof of anything just to be clear; it’s a silly way to make a point.

yes AI makes leaking keys on GH more prevalent, but so what? it’s the same problem as before with roughly the same solution

I’m saying neural networks being probabilistic doesn’t matter — everything is probabilistic. you can still practically use the tools to great effect, just like we use everything else that has underlying probabilities

OpenAI did not have to describe it as sycophancy, they chose to, and I’d contend it was a stupid choice

and yes, you can explain what went wrong just like you can with CPUs. we don’t (usually) talk about quantum-level physics when discussing CPUs; talking about neurons in LLMs is the wrong level of abstraction

> I was pointing out one screenshot from twitter isn’t proof of anything just to be clear; it’s a silly way to make a point.

Verses your anecdote being a proof of what? Skill issue for vibe coders? Someone else prompting it wrong?

You do realize you are proving my entire point?

> yes AI makes leaking keys on GH more prevalent, but so what? it’s the same problem as before with roughly the same solution

Again, it exacerbates my point such that it makes the existing issue even worse. Additionally, that wasn't even the only point I made on the subject.

> I’m saying neural networks being probabilistic doesn’t matter — everything is probabilistic.

When you scale neural networks to become say, production-grade LLMs, then it does matter. Just like it does matter for CPUs to be reliable when you scale them in production-grade data centers.

But your earlier (fallacious) comparison ignores the reliability differences between them (CPUs vs LLMs.) and determinism is a hard requirement for that; which the latter, LLMs are not.

> OpenAI did not have to describe it as sycophancy, they chose to, and I’d contend it was a stupid choice

For the press, they had to, but no-one knows the real reason, because it is unexplainable; going back to my other point on reliability.

> and yes, you can explain what went wrong just like you can with CPUs. we don’t (usually) talk about quantum-level physics when discussing CPUs; talking about neurons in LLMs is the wrong level of abstraction

It is indeed wrong for LLMs because not even the researchers can practically give an explanation why a single neuron (for every neuron in the network) gives different values on every fine-tune or training run. Even if it is "good enough", it can still go wrong at the inference-level for other unexplainable reasons other than it "overfitted".

CPUs on the other hand, have formal verification methods which verify that the CPU conforms to its specification and we can trust that it works as intended and can diagnose the problem accurately without going into atomic-level details.

No one is arguing that it isn't useful. The problem is this:

> I’m saying it doesn’t matter it’s probabilistic, everything is,

Maybe it doesn't matter for you, but it generally does matter.

The risk level of a technology failing is far higher if it is more random and unexplainable than if it is expected, verified and explainable. The former eliminates many serious use-cases.

This is why your CPU, or GPU works.

LLMs are neither deterministic, no formal verification exists and are fundamentally black-boxes.

That is why many vibe-coders reported many "AI deleted their entire home folder" issues even when they told it to move a file / folder to another location.

If it did not matter, why do you need sandboxes for the agents in the first place?

I think we agree then? the tech is useful; you need systems around them (like sandboxes and commit hooks that prevent leaking secrets) to use them effectively (along with learned skills)

very little software (or hardware) used in production is formally verified. tons of non-deterministic software (including neural networks) are operating in production just fine, including in heavily regulated sectors (banking, health care)

> I think we agree then? the tech is useful; you need systems around them (like sandboxes and commit hooks that prevent leaking secrets) to use them effectively (along with learned skills)

No.

> very little software (or hardware) used in production is formally verified. tons of non-deterministic software (including neural networks) are operating in production just fine, including in heavily regulated sectors (banking, health care)

It's what happens when it all goes wrong.

You have to explain exactlywhy, a system failed in heavily regulated sectors.

Saying 'everything is probabilistic' as the reason for the cause of an issue, is a non answer if you are a chip designer, air traffic controller, investment banker or medical doctor.

that’s not what I said. you honestly seem like you just want to argue about stuff (e.g. not elaborating on the “no” when I basically repeated and agreed with what you said). and you seem to consistently miss my point (in the second part of your response; I’m saying these non-deterministic neural networks are already widespread in industry with these regulations, and it’s fine. they can be explained despite your repeated assertions they cannot be. also the entire point on CPUs which you may have noticed I dropped from my responses because you seemed distracted arguing about it). this is not productive and we’re both clearly stubborn, glhf

> that’s not what I said. you honestly seem like you just want to argue about stuff (e.g. not elaborating on the “no” when I basically repeated and agreed with what you said). and you seem to consistently miss my point

I have repeated myself many times and you decide to continue to ignore the reliability points that inherently impede LLMs in many use-cases which exclude them in areas where predictability in critical systems is required in production.

Vibe coders can use them, but the gulf between useful for prototyping and useful for production is riddled with hard obstacles as such a software like LLMs are fundamentally unpredictable hence the risks are far greater.

> I’m saying these non-deterministic neural networks are already widespread in industry with these regulations, and it’s fine.

So when a neural network scales beyond hundreds of layers and billions of parameters, equivalent to a production-grade LLM, explain exactly how is such a black-box on that scale explainable when it messes up and goes wrong?

> they can be explained despite your repeated assertions they cannot be.

With what methods exactly?

Early on, I said formal verification and testing on CPUs for explaining when they go wrong at scale. It is you that provided absolutely nothing of your own assertions with the equivalent for LLMs other than "they can be explained" without providing any evidence.

> also the entire point on CPUs which you may have noticed I dropped from my responses because you seemed distracted arguing about it). this is not productive and we’re both clearly stubborn, glhf

You did not make any point with that as it was a false equivalence, and I explained why the reliability of a CPU isn't the same as the reliability of a LLM.

When I think real hard about it, I can translate the bullshit speak.
"get on board the AI train or get left behind" -> "BUY BUY BUY OUR SHIT! WE ARE SOOO OVER LEVERAGED WE ARE SO FUCKED. PLEASE CONSOOM"

The “anti-AU hype” phrase oversimplifies what’s playing out at the moment. On the tech side, while things are a bit rough around the edges still the tech is very useful and isn’t going away. I honestly don’t see much disagreement there.

The concern mostly comes from the business side… that for all the usefulness on the tech there is no clearly viable path that financially supports everything that’s going on. It’s a nice set of useful features but without products with sufficient revenue flowing in to pay for it all.

That paints a picture of the tech sticking around but a general implosion of the startups and business models betting on making all this work.

The later isn’t really “anti-AI hype” but more folks just calling out the reality that there’s not a lot of evidence and data to support the amount of money invested and committed. And if you’ve been around the tech and business scene a while you’ve seen that movie before and know what comes next.

In 5 years time I expect to be using AI more than I do now. I also expect most of the AI companies and startups won’t exist anymore.

In the late 2000s i remember that "nobody is willing to pay for things on the Internet" was a common trope.
I think it'll culturally take a while before businesses and people understand what they are willing to pay for. For example if you are a large business and you pay xxxxx-xxxxxx per year per developer, but are only willing to pay xxx per year in AI tooling, something's out of proportion.

> For example if you are a large business and you pay xxxxx-xxxxxx per year per developer, but are only willing to pay xxx per year in AI tooling, something's out of proportion.

One is the time of a human (irreplaceable) and the other is a tool for some human to use, seems proportional to me.

> if you are a large business and you pay xxxxx-xxxxxx per year per developer, but are only willing to pay xxx per year in AI tooling, something's out of proportion.

Is way off base. Even if you replace multiple workers with one worker but better tool, businesses still won't want to pay the "multiple worker salary" to the single worker just because they use a more effective tool.

It would seem to me that tokens are only going to get more efficient and cheaper from here.

Demand is going to rise further as AI keeps improving.

Some argue there is a bubble, but with demand from the public for private use, business, education, military, cyber security, intelligence, it just seems like there will be no lack of investment.

People said the exact same thing about (numbers from memory, might be off):

- when Google paid $1 bil for YouTube

- when Facebook paid $1 bil for Instagram

- when Facebook paid $1 bil for WhatsApp

The same thing - these 3 companies make no money, and have no path to making money, and that the price paid was crazy and decoupled from any economics.

Yet now, in hindsight, they look like brilliant business decisions.

I am not even clear how Whatsapp "paid off" for Facebook in any sense other than them being able to nip a potential competitor in the bud. I use Whatsapp but do not see a single advert there nor do I pay a single penny for it, and I suspect my situation is pretty typical. Presumably some people see ads or pay for some services but I've not, and I don't imagine there's that much money to be made in being the #1 platform for sharing "Good Morning" GIFs

While many people thought Facebook/Google paid too much for these companies, you're making an apples-to-oranges comparison. That part about there being "no path to making money" is wrong - online advertising was a huge industry and only getting stronger and while YT/Insta/Whatsapp may have struggled as standalone companies it was clear they'd unlock an enormous amount of value as part of a bigger company that already had a strong foothold in advertising online.

It is not clear who, other than maybe someone like Microsoft, could actually acquire companies like OpenAI or Anthropic. They are orders of magnitude larger than the companies you mentioned in terms of what they are "worth" (haha) and even how much money they need just to keep the lights on, let alone turn any kind of profit.

Not to mention the logical fallacy at the core of your point - people said "the exact same[sic] thing" about YouTube, Instagram and Whatsapp ... therefore, what, it necessarily means these companies are the same? You realise that many of us talked like this about "the blockchain", and "the Metaverse" and about those stupid ape JPEGS and we were absolutely correct to do so.

> Not to mention the logical fallacy at the core of your point

Yes, it's a logical fallacy. Another one is saying "I don't see any viable business model, therefore there is no viable business model".

Blast from the past:

> YouTube is a content paradise though. There's tons of value there and you can sell ads against it or even charge for premium services.

> Where's the money in Instagram? The content is practically worthless and their only real value is in their userbase. Even though I use the Instagram client, most of the time I see photos, they come through Twitter. So that also reinforces for me that any value is in the users and not the actual content, which is mostly crap.

> I'm more convinced that we're in a 2nd bubble now more than ever.

> Does anyone else think this valuation is insane? It's like $300/registered user. The company doesn't have a business model. No way the handful of employees are worth $1B. My mind is blown.

It sounds like you're really into this and I hope for all of our sakes that you are correct to be all hyped up about AI. Because if you're not and that this is a horrific bubble that is going to burst then we're all in big trouble

yeah, and Zuckerberg said that everyone on planet Earth will buy his VR helmet, and renamed his whole company after a stupid game which i don't think even exists anymore. Being a contrarian doesn't mean you are right, and sometimes seemingly stupid money-losing things turn out... stupid.

Uber are doing something entirely different though - they took a market which was proven to exist, created a product which worked then spent a decade being horribly unprofitable until they were the dominant player in that market. And even at their very worst they weren't losing as much money as OpenAI are. There's far too much hand-waving and dismissive "ah it'll be ok because Uber exist" going on among those who have bought into the AI hype cycle

We don't really know how much money Google sunk into YouTube before it became (presumably) profitable. It might have actually not been strongly coupled to economics.

I was not clear enough. I wanted to write a PRO-AI blog post. The people against AI always say negative things with using as central argument that "AI is hyped and overhyped". So I, for fun, consider the anti-AI movement a form of hype. It's a joke but not in the sense it does not mean what it means.

However, as you point out, anti-AI people are pushing back against hype, not indulging in hype themselves - not least as nobody is trying to sell 'not-AI'.

I for one look forward to the next AI winter, which I hope will be long, deep, and savage.

And I'm sure if you go back to the release of 3.5, you'll see the exact same comments.

And when 5 comes out, I'm sure I'll see you commenting "OK I agree 6 months ago but now with Claude 5 Opus it's great".

It's really the weirdest type of goalpost moving.

I have used Opus 4.5 a lot lately and it's garbage, absolutely useless for anything beyond generating trivial shit for which I'd anyway use a library or have it already integrated in the framework I use.

I think the real reason your opinion has changed in 6 months is because your skills have atrophyed.

It's all as bad as 6 months ago, and even as bad as 2 years ago, you've just become worse.

> Not from people whose opinions on that I respect.

Then you shouldn't respect Antirez's opinion, because he wrote articles saying just that 2 years ago.

> If you think LLMs today are "as bad as 2 years ago" then I don't respect your opinion. That's not a credible thing to say.

You are getting fooled by longer context windows and better tooling around the LLMs. The models themselves have definitely not gotten better. In fact it's easy to test, just give the exact same prompt to 3.5 and 4.5, and receive the exact same answer.

The only difference is that when you used to copy-paste answers from the ChatGPT UI, you now have it integrated in your IDE (with the added bonus of it being able to empty your wallet much quicker). It's a faster process, not a better one. I'd even argue it's worse, since you spend less time reviewing the LLM's answer in this situation.

How do you explain that it's so easy to tell (in a bad way) when a PR is AI-generated if it's not necessary to code by hand anymore?

> Despite the large interest in agents that can code alone, right now you can maximize your impact as a software developer by using LLMs in an explicit way, staying in the loop.

There are too many people who see the absurd AI hype (especially absurd in terms of investment) and construct a counter-argument with it that AI is useless, overblown and just generally not good. And that's a fallacy. Two things can be true at the same time. Coding agents are a step change and immensely useful, and the valuations and breathless AGI evangelizing is a smoke screen and pure hype.

Don't let hype deter you to get your own hands dirty and try shit.

On the tech side, while things are a bit rough around the edges still the tech is very useful and isn’t going away. I honestly don’t see much disagreement there.

What? HN is absolutely packed with people complaining about LLMs are nothing more than net useless creators of slop.

Granted, fewer than six months ago, which should tell people something...

> How do I feel, about all the code I wrote that was ingested by LLMs? I feel great to be part of that, because I see this as a continuation of what I tried to do all my life: democratizing code, systems, knowledge.

I don't see it as democratic or democratising. TBH the knowledge is stored in three giga companies that used sometimes almost non-lawful (if not lawful?) methods to gain it, scraping it off the gpl projects etc. And now they are selling it to us without giving the models away. The cost IS understandable because the horrendously expensive vector cards do not come for free, but there is only one country the knowledge is gathered in so this might as well fade away one day when an orange present says so (gimme all the monies or else..)

"democratizing" as in "I steal everybody's shit, make most content creators go bankrupt, then put it all in an LLM behind a paywall." Privatization of all human knowledge--past, present, & future. They own both parties, so it's not like anyone is going to vote their way out of this one--unless one considers guillotines a form of voting.

>> I don't see it as democratic or democratising. TBH the knowledge is stored in three giga companies

It can appear democratic while access is allowed, but if it can be revoked at any moment for any reason (it is private companies, after all, that own the AI playgrounds), then the illusion will shatter.

What is more, excessive reliance on AI creates skill deficit rather than skill surplus, and promotes dependence on AI. Wizards that are nothing without their magic wands, in a way.

This may not stand out today, but give it half or one decade, when the next generation won't have a pre-AI skillet to fall back to, and the seams will become all too apparent.

I already notice that my brains tends to resist thinking on hard things, defaulting to gpt-ing or perplex-ing things. Similar feeling I had when I bought a car with parking sensors - I almost immediately lost skills of parking my older car which doesn't have those. I had to re-learn it.

> state of the art LLMs are able to complete large subtasks or medium size projects alone, almost unassisted, given a good set of hints about what the end result should be

No. I agree with the author, but it's hyperbolic of him to phrase it like this. If you have solid domain knowledge, you'll steer the model with detailed specs. It will carry those out competently and multiply your productivity. However, the quality of the output still reflects your state of knowledge. It just provides leverage. Given the best tractors, a good farmer will have much better yields than a shit one. Without good direction, even Opus 4.5 tends to create massive code repetion. Easy to avoid if you know what you are doing, albeit in a refactor pass.

I feel like a lot of the disagreement over this "large project" capability is that "large project" can mean anything. It can mean something that has a trillion github repos to work with, or it can mean something that is basically uncharted territory.

If this only works for people with like 10+ years of domain experience, doesnt that make this an Anti-AI article? Whole vibe coding sells on the promise that it works and it works for every tom and their mom.

One is LLMs writing code. Not everything and not for everyone. But they are useful for most of the code being written. It is useful.

What it does not do (yet, if ever) is bridging the gap from "idea" to a working solution. This is precisely where all the low-code ideas of the past decades fell apart. Translating an idea in to formal rules is very, very hard.

Think of all of the "just add a button there"-type comments we've all suffered.

Yes that’s how I see it too. It’s a productivity multiplier, but depends on what you put in.

Sure Opus can work fully on its own by just telling it “add a button that does X”, but do that 20 times and the good turns into mush. Steer the model with detailed tech specs on the other hand, and the output becomes magical

I perfectly agree with antirez about the importance of AI and the benefit for coders. In the last month we saw a big jump and we all are in the middle of the biggest technological revolution since the internet. He summarised the benefits, but omitted the rest.

Why we don't have to be anti-AI? Why in his opinion is just "HYPE"? I didn't find any answer in his post. He doesn't analyse the cons of AI and explain why some people might be anti-AI. He skipped the hard part and wrote a mild article that re-publish the narrative that is already getting spread on every social media.

Edit for clarification: I don't consider anti-AI the people that think LLMs don't work, they are wrong. I consider anti-AI people that are worried how this technology will impact society in so many ways that are hard to predict, including the future of software engineering.

From purely business and career perspective, being anti-AI will be a self-own unless you work for niche companies that have the anti-AI stance. Yes, they exist. But if a company is building, supporting, or consulting any product, where timing matters and there’s competition (which is super majority), it’ll be in their best interest to nudge their employees to speed up via AI.

I do think at least being proficient right now with the LLMs will help you with whatever comes next, just because you’ll build the intuition around it. Being anti-AI might negatively affect one’s employability, and especially the younger ones who don’t have seniority or connections over the decades.

There are plenty of non-blockchain, non-NFT, non-online gambling, non-adtech, non-facist software jobs. In fact, the vast majority of software jobs are. You can refuse to work with all of these things and not even notice a meaningful difference in career opportunities.

If you refuse to work with AI, however, you're already significantly limiting your opportunities. And at the pace things are going, you're probably going to find yourself constrained to a small niche sooner rather than later.

If your argument is that there are more jobs that require morally dubious developments (stealing people's IP without licensing it, etc.) than jobs that don't, I don't think that's news.

There's always more shady jobs than ethically satisfying ones. There's increasingly more jobs in prediction markets and other sorts of gambling, adtech (Meta, Google). Moral compromise pays.

But if you really think about it and set limits on what is acceptable for you to work on (interesting new challenges, no morally dubious developments like stealing IP for ML training, etc.) then you simply don't have that FOMO of "I am sacrificing my career" when you screen those jobs out. Those jobs just don't exist for you.

Also, people who tag everybody like that as some sort of "anti-AI" tinfoilhatters are making a straw man argument. Most people with an informed opinion don't like the ways this tech is applied and rolled out in ways that is unsustainable and exploitative of ordinary people and open-source ecosystem, the confused hype around it, circular investment, etc., not the underlying tech on its own. Being vocally against these matters does not make one an unemployable pariah in the slightest, especially considering most jobs these days build on open source and being anti license-violating LLMs is being pro sustainable open-source.

> There's always more shady jobs than ethically satisfying ones. There's increasingly more jobs in prediction markets and other sorts of gambling, adtech (Meta, Google). Moral compromise pays.

I would say, this is not about the final product, but a way of creating a product. Akin to writing your code on TextPad vs. using VSCode. Imo, having a moral stance on AI-generated art is valid, but AI-generated code isn't, just because I don't consider "code" "art".

I've been doing it for about 20 or so years at this point, throughout literally every stage of my life. Personally, I'd judge a person who is using AI to copy someone's art, but if someone is using AI to generate code gets a pass from me. That being said, a person who considers code as "art" (I have friends like that, so I definitely get the argument!), would not agree with me.

> Most people with an informed opinion don't like the ways this tech is applied

Yeah, I'm not sure if this tracks? I don't think LLMs are good/proficient as a tool for very specialized or ultra-hard tasks, however for any boilerplate-coding-task-and-all-CRUD-stuff, it would speed up any senior engineer in task completion.

> I would say, this is not about the final product, but a way of creating a product.

It is the same logic as not wanting to use some blockchain/crypto-related platform to get paid. If you believe it is mostly used for crime, you don't want to use it to get paid to avoid legitimizing a bad thing. Even if there's no doubt you will get paid, the end result is the same, but you know you would be creating a side effect.

If some way of creating a product supports something bad (and simply using any LLM always entails helping train it and benefit the company running it), I can choose another way.

That is because your views appear to align with staunch progressives. From rejecting conservative politics ("fascism"), AI, advertising, and gambling.

From my side the only thing I would be hesitant about is gambling. The rest is arguably not objectively bad but more personal or political opinion from your side.

There seems to be some confusion. I wouldn't call conservative politics as a whole fascist, that's your choice of words. I doubt that "anti-AI progressive" is a thing too.

> The rest is arguably not objectively bad but more personal or political opinion from your side.

Nothing is objectively bad. Plenty of people argue that gambling should be legal if anything on the basis of personal freedom. All of this is a matter of personal choice.

(Incidentally, while you are putting people in buckets like that, note that one person very much can be similtaneously against gambling and drug legalization and be pro personal freedom open-source libertarian maximalist. Things are much more nuanced than “progressive” vs. “conservative”, whatever you put in those buckets is on you.)

It is just from my experience that political discussions online are very partisan. "fascism" in relation to the current US government combined with anti-AI sentiment is almost always a sure indicator for a certain bucket of politics.

To play devil's advocate: all the people using AI are not being significantly more productive on brownfield applications. If GP manages to find a Big Co (tech or non tech) which doesn't precisely bother about AI usage and just delivering features, and the bottleneck is not software dev (as is the case in majority of old school companies), he/she would be fine.

Some people are also opposed because of the negative externalities when building and running AI systems (environmental consequences, intellectual property theft), even if they understand that agentic coding "works". This is a valid position.

I have not seen those arguments in the context of what I would consider anti-hype. But in any case: There are certainly issues attached to usage of AI more generally.

It only works for languages and frameworks that are already in the training data (duh). It still is mostly useless when you need to create something from scratch in an unstable language.

That, and you can’t also get the amazing results if you’re poor or have bad internet.

Not true. I built some tools in Hare, which almost certainly isn’t in the training data to any significant extent. It was more work than having it build Go or Rust, but it got it done. It had to curl the docs a fair bit.

It’s an easy deflection. Dismiss any opinions because you’re using it wrong or not the latest.

Good for anything >= 1 month old.

Use other nonsense fear inducing argument in the mean time, continue gathering gobs of VC money, get your bag, continue till the bubble pops.

In all fairness, and putting hype and anti-hype aside, I’m really interested to see the actual value of LLM/agent services after the VC money subsidies dry out. Would people we willing to pay for services at 10x the current price?

As a security engineer that regularly architects and helps implement new defense tactics that no LLM has trained on, I choose not to use LLMs at all, like a cave man.

Being differently trained and using different tools than almost everyone else I know in engineering my entire career has allowed me to find solutions and vulnerabilities others have missed time and time again. I exclusively use open source software I can always take apart, fully understand, and modify as I like. This inclination has served me well and is why I have the skillsets I do today.

If everyone is doing things one way, I instinctively want to explore all the other ways to train my own brain to continue to be adversarial and with a stamina to do hard experiments by hand when no tools exist to automate them yet.

Watching all my peers think more and more alike actually scares me, as they are all talking to the same LLMs. None for me, thanks.

"But this magic proprietary tool makes my job so much easier!!" has never been a compelling argument for me.

> How do I feel, about all the code I wrote that was ingested by LLMs? I feel great to be part of that, because I see this as a continuation of what I tried to do all my life: democratizing code, systems, knowledge. LLMs are going to help us to write better software, faster, and will allow small teams to have a chance to compete with bigger companies.

You might feel great, thats fine, but I dont.
And software quality is going down, I wouldn't agree that LLMs will help write better software

> I wouldn't agree that LLMs will help write better software

Your statement makes no sense.

Even if you don't let LLMs author a single line of your code, they can still review it, find edge cases you didn't think about or suggest different approaches.

The fact that AI allows lots of slop, does not negate its overall utility in good informed hands.

The balance between 'find edge cases' and 'hallucinate non-existing cases and waste your time' may be negative. LLMs are also not free, they cost significant money even today, when they are subsidized by marketing budget.

It was already shown repeatedly in GitHub repositories in the last year that authors are really unhappy with AI generated pull-requests and test cases.

That is capitalism capitializing. I sorta think it is also the computer going from a geek toy to mass adoption and incentives changing. 3D printers for example are good but if they go mainstream they'll become like HP 2D printers on the enshittification axis

As for the delta provided by an LLM to Antirez, outside of Redis (and outside of any problem space he is already intimately familiar with), an Apples to Apples comparison would be he trying this on an equally complex codebase he has no idea about. I'll bet... what Antirez can do with Redis and LLMs (certainly useful, huge Quality of Life improvement to Antirez), he cannot even begin to do with (say) Postgres.

The only way to get there with (say) Postgres, would be to /know/ Postgres. And pretty much everyone, no matter how good, cannot get there with code-reading alone. With software at least, we need to develop a mental model of the thing by futzing about with the thing in deeply meaningful ways.

And most of us day-job grunts are in the latter spot... working in some grimy legacy multi-hundred-thousand line code-mine, full of NPM vulns, schelpping code over the wall to QA (assuming there is even a QA), and basically developing against live customers --- "learn by shipping", as they say.

I do think LLMs are wildly interesting technology, however they are poor utility for non-domain-experts. If organisations want to profit from the fully-loaded cost of LLM technology, they better also invest heavily in staff training and development.

Exactly. AI is minimally useful for coding something that you couldn't have been able to code yourself, given enough time, without explicitly investing time in generic learning not specific to that codebase or particular task.

Although calling AI "just autocomplete" is almost a slur now, it really is just that in the sense that you need to A) have a decent mental picture of what you want, and, B) recognize a correct output when you see it.

On a tangent, the inability to identify correct output is also why I don't recommend using LLMs to teach you anything serious. When we use a search engine to learn something, we know when we've stumbled upon a really good piece of pedagogy through various signals like information density, logical consistency, structuredness/clarity of thought, consensus, reviews, author's credentials etc. But with LLMs we lose these critical analysis signals.

While you're correct. I truly believe the velocity offered outweighs this consideration for 90% of the application teams and startups. I've personally never worked in a clean codebase, and I've been convinced long ago that they're mythical. I don't see an issue with an LLM spitting out bad / barely maintainable code because that's basically every codebase I've ever seen in production.

> And pretty much everyone, no matter how good, cannot get there with code-reading alone. With software at least, we need to develop a mental model of the thing by futzing about with the thing in deeply meaningful ways

LLMs help with that part too. As Antirez says:

Writing code is no longer needed for the most part. It is now a lot more interesting to understand what to do, and how to do it (and, about this second part, LLMs are great partners, too).

It's who has a world-model. It's who can evaluate input signal against said world-model. Which requires an ability to generate questions, probe the nature of reality, and do experiments to figure out what's what. And it's who can alter their world-model using experiences collected from the back-and-forth.

Yes most c-level executives (who often have to report to a board) have tendencies to predict the future after using claude code. It didn't happen in 2025 yet they still insist. While their senior engineers are still working at the production code.

What "domain expert" means is also changing however.

As I've mentioned often, I'm solving problems in a domain I had minimal background in before. However, that domain is computer vision. So I can literally "see" if the code works or not!

To expand, I've set up tests, benchmarks and tools that generate results as images. I chat with the LLM about a specific problem at hand, it presents various solutions, I pick a promising approach, it writes the code, I run the tests which almost always pass, but if they don't, I can hone in on the problem quickly with a visual check of the relevant images.

This has allowed me to make progress despite my lack of background. Interestingly, I've now built up some domain knowledge through learning by doing and experimenting (and soon, shipping)!

These days I think an agent could execute this whole loop by itself by "looking" at the test and result images itself. I've uploaded test images to the LLM and we had technical conversations about them as if it "saw" them like a human. However, there are ton of images and I don't want to burn the tokens at this point.

The upshot is, if you can set up a way of reliably testing and validating the LLM's output, you could still achieve things in an unfamiliar domain without prior expertise.

Taking your Postgres example, it's a heavily tested and benchmarked project. I would bet someone like Antirez would be able to jump in and do original, valid work using AI very quickly, because even if hasn't futzed with Postgres code, he HAS futzed with a LOT of other code and hence has a deep intuition about software architecture in general.

So this is what I meant by the meaning of "domain expert" changing. The required skills have become a lot more fundamental. Maybe the only required skills are intuition about software engineering, critical thinking, and basic knowledge of statistics and the scientific method.

>> ...however they are poor utility for non-domain-experts.

IDK, just two days ago I had a bug report/fix accepted by a project which I would have never dreamt of digging into as what it does is way outside my knowledge base. But Claude got right on in there and found the problem after a few rounds of printf debugging which lead to an assertion we would have hit with a debug build which led to the solution. Easy peasy and I still have no idea how the other library does its thing at all as Claude was using it to do this other thing.

AI is basically Leonard from Memento. Very capable. Knows how the world works broadly. Can't make new memories. Need context (tattoos, notes, and polaroids). Misunderstandings things.

if you are very high up the chain like Linus, i think doing vibe coding gives you more feedback than any average dev. So they are having a positive feedback loop.

For most of us vibe coding gives 0 advantage. Our software will just sit there and get no views and producing it faster means nothing. In fact, it just scares us that some exec is gonna look at this and write us for low performance because they saw someone do the same thing we are doing in 2 days instead of 4.

Less a 'chain' or hierarchy than a lecture hall with cliques. Many of the 'influencers', media personalities, infamous, famous, anyone with a recognizable name - for the most part - was introduced to the tsunami wave of [new tech] at the same time. They may come with advantages, but it's how they get back to the 'top' (for your chain) vs. staying up there.

Except that Linus does basically zero programming these days. He's a manager, combining code from the subsystem managers below him into a final release.

Right, but Linus also has an extremely refined mental model of the project he maintains, and has built up a lot of skills reading code.

Most engineers in my experience are much less skillful at reading code than writing code. What I’ve seen so far with use of LLM tools is a bunch of minimally edited LLM produced content that was not properly critiqued.

This looks more worrying than impressive. It's long files of code with if-statements and flag-checking unicode bit patterns, with an enormous number of potential test-cases.

It's not conceptually challenging to understand, but time consuming to write, test, and trust. Having an LLM write these types of things can save time, but please don't trust it blindly.

I see dividing the tests and code into two different changes is pretty nice, In fact I have been using double agent thing where one is writing tests and other is writing the code, solves the attention issue also. Although the code itself looks harder to read, but that is probably more on me than Claude.

I'm not sure the blog post goes in the opposite direction of what you say, in fact he points out that the quality of the output depends on the quality of the hints, which implies that quality hints require quality understanding from the user.

Keep believing. To the bitter end. For such human slop codebases AI slop additions will do equally fine. Add good testing and the code might even improve over the garbage that came before.

Generating also the tests happens a little bit too often for any kind of improvement. simonw posted here a generated “something” the other day, which he didn’t know whether it’s really working or not, but he was happy that his generated, completely unchecked tests are green, and yet some other root commenter here praises him.

It needs a lot of work to not be skeptical, when when I try it, it generates shit, especially when I want something completely new, not existing anywhere, and also when these people when they show how they work with it, it always turns out that it’s on the scale of terrible to bad.

I also use AI, but I don’t allow it to touch my code, because I’m disgusted by its code quality. I ask it, and sometimes it delivers, but mostly not.

If my tests are green then it tells me a LOT about what the software is capable of, even if I haven't reviewed every line of the implementation.

The next step is to actually start using it for real problems. That should very quickly shake out any significant or minor issues that sneaked past the automated tests.

I've started thinking about this by comparing it to work I've done within larger companies. My team would make use of code written by other teams without reviewing everything those other teams had written. If their tests passed we would build against their stuff, and if their stuff turned out not to work we would let them know or help debug and fix it ourselves.

It’s absolutely not enough to “keep an eye on it on your phone”. You need to know that the implementation of the tests are real. LLMs routinely make shortcut in tests to make them green. There was an occasion when flat out mocked everything from the live code, and it was a very-very simple python REST API, tests of course were green.

I'm trying not to fall for it, but when I try ai to write code it fails more often than not - at least for me. some people claim it does everything but I keep finding major problems. Even when it writes something that works often I can't explain that in 2026 we should be using smart pointers (C++) or what ever the modern thing

Same here. I’ve had limited success getting AIs to do very simple stuff. Every one I’ve tried invents APIs that don’t exist and eventually get stuck in a circle where it tells me to try A. When that doesn’t work, try B. No luck? Try C. Hmmm my apologies, try A. Lather, rinse, repeat.

No, I haven’t tried that yet. I don’t really want to turn on auto mode when it’s iterating on my credit card and it looks like it’s in an infinite loop… Is that a silly thing to be worried about?

I work mostly in C++ (MFC applications on Windows) and assembly language (analyzing crash reports).

For the C++ work, the AIs do all kinds of unsafe things like casting away constness or doing hacks to expose private class internals. What they give me is sometimes enough to get unstuck though which is nice.

For crash reports (a disassembly around the crash site and a stack trace) they are pretty useless and that’s coming from someone who considers himself to be a total novice at assembly. (Looking to up my x64 / WinDbg game and any pointers to resources would be appreciated!)

I do prototyping in Python and Claude is excellent at that.

Can you not just use Claude Code with a plan and cap your spending that way? I am using just the £18 plan, and it is quite helpful already. If you haven't tried Claude Code yet, just do it, experience agentic coding, and afterwards we can talk.

>> No, I haven’t tried that yet. I don’t really want to turn on auto mode when it’s iterating on my credit card and it looks like it’s in an infinite loop… Is that a silly thing to be worried about?

Yeah. Most AIs today are pretty good at detecting that they're in a loop and aren't making progress. When that happens, they either take a different approach, or stop and say they are stuck. But, if you're really worried about it, you can cap monthly spend on the billing page of virtually every AI provider.

AI tools in their current form or another will definitely change software engineering, I personally think for the best

However I can’t help but notice some things that look weird/amusing:

- The exact time that many programmers were enlightened about the AI capabilities and the frequency of their posts.

- The uniform language they use in these posts. Grandiose adjectives, standard phrases like ‘it seems to me’

- And more importantly the sense of urgency and FOMO they emit. This is particularly weird for two reasons. First is that if the past has shown something regarding technology is that open source always catches up. But this is not the case yet. Second, if the premise is that we re just the in beginning all these ceremonial flows will be obsolete.

Do not get me wrong, as of today these are all valid ways to work with AI and in many domains they increase the productivity. But I really don’t get the sense of urgency.

> The exact time that many programmers were enlightened about the AI capabilities and the frequency of their posts.

I attribute that to the holidays. Many people finally had the time to goof around with these tools. At least that's how it happened to me.

It was an incredible experience. I implemented a few features quickly and in a much better way than I could otherwise. Realized how many tiny holes my app had and a few suboptimal patterns I was using. Made me worry about my career, initially, but after using for a while, I now see it as going up the chain of abstraction. Only thing I'm not doing is writing code by hand. Im still having to do everything else like thinking about architecture and the big picture, keeping it dry and maintainable, debugging, etc - but with a lot of help from LLMs. Sometimes it's 10x and sometimes you wasted sometime, you know, just like how using packages made us go up the chain.

These personal blogs are starting to feel like Linkdin Lunatic posts, kinda similar. to the optimised floor sweeping blog, “I am excited to provide shareholder value, at minimum wage”

What does it tell you that programmers with the credibility of antirez - and who do not have an AI product to sell you - are writing things like this even when they know a lot of people aren't going to like reading them?

What it tells me is that humans are fallible, and that being a competent programmer has no correlation with having strong mental defenses against the brainrot that typifies the modern terminally-online internet user.

I leverage LLMs where it makes sense for me to do so, but let's dispense with this FOMO silliness. People who choose not to aren't missing out on anything, any more than people who choose to use stock Vim rather than VSCode aren't missing out on anything.

It's not Vim vs VSCode though - the analogy might be writing in assembler vs writing in your high level language of choice.

Using AI you're increasing the level of abstraction you can work at, and reducing the amount of detail you have to worry about. You tell the AI what you want to do, not how to do it, other than providing context that does tell it about the things that you actually care about (as much or little as you choose, but generally the more the better to achieve a specific outcome).

> the analogy might be writing in assembler vs writing in your high level language of choice.

If it were deterministic, yes, but it's not. When I write in a high level language, I never have to check the compiled code, so this comparison makes no sense.

If we see new kinds of languages, or compile targets, that would be different.

It's a new type of development for sure, but with an agentic system like Claude Code that is able to compile, run and test the code it is generating you can have it iterate until the code meets whatever test or other criteria you have set. No reason code reviews can't be automated too, customized to your own coding standards.

Effort that might be put into feeling that you need to manually review all code generated might better be put into things like automating quality checks (e.g code review, adherence to guidelines) ensuring that testing is comprehensive, and overall management of the design and process into modular testable parts the same way as if you'd done it manually.

While AI is a tool, the process of AI-centric software development is better regarded as a pair-design and pair-coding process, treating the AI more like a person than a tool. A human teammate isn't deterministic either, yet if they produce working artifacts that meet interface requirements and pass unit tests, you probably aren't going to insist on reviewing all of their code.

> the process of AI-centric software development is better regarded as a pair-design and pair-coding process, treating the AI more like a person than a tool.

This is the part that makes me throw up in my mouth a bit, I'd rather pair with a human. But whatever, I'm old. You'll have to excuse me as as there are a lot of nefarious-looking clouds out there.

Sure, but the AI is faster & cheaper than a human, or even of a team of humans. So, if you are a solo developer and can't afford to hire a team of humans to help accelerate your project, you now have the option of using AI instead.

It seems the capability and utility of these models/products is increasing very fast. Agentic tools like Claude Code that run locally in your terminal and therefore have access to all your dev/test tools and environment is a huge advance since now the output isn't just code, it's fully tested debugged code, that passes whatever tests and quality gates you tell it are necessary.

At the same time that the tooling has improved, so have the models, and only very recently (last 6 months or so). People swear by Opus 4.5, and I've also been impressed by Gemini 3.0. A year ago I was also much more skeptical of the utility of AI for serious use, but they've improved a lot.

This is so stupid. You still have to review that code, you still have to know what the solution to something is, ergo, you still need to know how to do it and you still have to deal with the cognitive load from reviewing someone else's code. I don't understand how you can write as if the implementation, fairly trivial and mechanical, is somehow more taxing than reading someone else's code..

This is not the support argument you think it is, it just further allures to the fact that people raving about AI just generate slop and either don't review their code or just send it for their coworkers to review.

I guess AI bros are just the equivalent of script-kiddies, just running shit they don't know how it works and claiming credit for it.

It depends on what you are using it for, and how you are using it. If you are using AI to write short functions that you could code yourself in close to the same time as reviewing the AI generated code, then obviously there is no benefit.

There are however various cases where using AI can speed development considerably. One case is larger complex project (thousands of LOC) where weeks of upfront design would have been followed by weeks/months of implementation and testing.

You are still going to do the upfront design work (no vibe coding!) and play role of lead developer breaking the work into manageable pieces/modules, but now there is value in having the AI write, test and debug the code, including generating unit tests, since this would otherwise have been a lengthy process.

This assumes you are using a very recent capable frontier model in an agentic way (e.g. Claude Code, or perhaps Claude web's Code Interpreter for Python development) so that the output is debugged and tested code. We're not talking about just having the AI generate code that you then need to fix and test.

This also assumes that this is a controlled managed process. You are not vibe coding, but rather using the AI as a pair-programmer working on one module at a time. You don't need to separately review the code line by line, but you need to be aware of what is being generated, and what tests are being run, so that you have similar confidence in the output that you might have done if you'd pair-programmed it with a human, or perhaps delegated it to someone else with sufficient specifications that "tested code meeting specs" means you don't have to review the code in detail unless you choose to.

I haven't tried it myself, but note that you can also use AI to do code reviews, based on a growing base of code standards and guidelines that you provide. This can then be used as part of the code development process so that the agent writing the code iterates until it passes code review as well as unit tests.

> We have to abandon the appeal to authority and take the argument on its merits, which honestly, we should be doing regardless.

I don't really agree. In virtually any field, when those who have achieved mastery speak, others, even other masters, tend to listen. That does not mean blindly trust them. It means adjust your priors and reevaluate your beliefs.

Software development is not special. When people like antirez (redis) and simonw (django) and DHH (rails) are speaking highly of AI, and when Linus Torvalds is saying he's using AI now, suggesting they may be on to something is not an appeal to authority. And frankly, claiming that they might be saying nice things about AI because of some financial motive is crazy.

> And frankly, claiming that they might be saying nice things about AI because of some financial motive is crazy.

I'm actually taken aback by the vehemence of the anti-AI brigade on HN. It seems objectively crazy to me to suggest someone like antirez, with a long visible history now has an agenda to push AI products, so he writes blog posts to do so.

This is just genuinely going into the wilfully blind territory now, and your post is the one downvoted for pointing it out.

I think we are properly into holy war territory and people on either side are losing their minds, and their objectivity.

People higher up the ladder aren't selling anything but they also have to not worry about losing jobs. We are worried that execs are going to see the advances and quickly clear the benches, might not be true but every programmer believing they have become a 10x programmer pushes us more into that reality.

That is an argument to authority. There is a large enough segment of folks who like to be confirmed in either direction. Doesn't make the argument itself correct or incorrect. Time will tell though.

> Whatever you believe about what the Right Thing should be, you can't control it by refusing what is happening right now. Skipping AI is not going to help you or your career. Think about it. Test these new tools, with care, with weeks of work, not in a five minutes test where you can just reinforce your own beliefs.

This is the advice I've been giving my friends and coworkers as well for a while now. Forget the hype, just take time to test them from time to time. See where it's at. And "prepare" for what's to come, as best you can.

Another thing to consider. If you casually look into it by just reading about it, be aware that almost everything you read in "mainstream" places has been wrong in 2025. The people covering this, writing about this, producing content on this have different goals in this era. They need hits, likes, shares and reach. They don't get that with accurate reporting. And, sadly, negativity sells. It is what it is.

THe only way to get an accurate picture is to try them yourself. The earlier you do that, the better you'll be. And a note on signals: right now, a "positive" signal is more valuable for you than many "negative" ones. Read those and try to understand the what, if not the how. "I did this with cc" is much more valuable today than "x still doesn't do y reliably".

> Whatever you believe about what the Right Thing should be, you can't control it by refusing what is happening right now. Skipping AI is not going to help you or your career. Think about it. Test these new tools, with care, with weeks of work, not in a five minutes test where you can just reinforce your own beliefs.

You can refuse to support it on the grounds that its being used to harm people. That might not do anything but its still important to be on the right side of humanity.

I don't condemn the tech, but the tech depends on factors that are harming people and not supporting that part of it is an act of support for humanity.

How would we measure the effects of AI coding tool taking over manual coding ? Would we see an increase in the number of GitHub projects ? In the number of stars (given the ai is so good) ? In the number of start up ipos (surely if all your engineers are 1000x engineers thanks to Claude code, we'll have plenty of googles and Amazons to invest in) ? In the price of software (if I can just vibe code everything, than a 10$ fully compatible replacement for MS Windows is just a few months away, right ?) In the the numbers of app published in the stores ?

Plot twist: the bottleneck when you have a development force multiplier is __MARKETING__. If you develop at 10X the rate, you still have to grind/growth marketing. Unmarketed products might as well not exist, even if they're fantastic.

Github stars? That's 100% marketing. Shit that clears a low quality bar can rack up stars like crazy just by being well marketed.

Number of startups? That's 100% marketing. Investors put money into products that have traction, or founders that look impressive, and both of those are mostly marketing.

People actually are vibe coding stuff rather than using SaaS though, that one's for real. Your example is hyperbolic, but the Tailwind scenario is just one example of AI putting pressure on products.

Yes definitely. The average quality of Github projects, startup code may improve but there will probably not be many more stars on them or successful startups because it's immaterial and so easy to switch, the best project to do X becomes 100x more successful than the second-best project.

You cant vibe code users or traction. If you make they will come is not a strategy for 2026. In fact, the amount of money needed for marketing will wipe out any savings from not having a Software dev.

If you make they will come has never been a valid strategy. And marketing is fucking miserable now because of the proliferation low quality software people are trying to turn into SaaS.

If you don't have a halo already, you need to be blessed or you're just going to suffer. Getting a good mention by someone like Theo or SimonW >> 1000 well written articles.

I get annoyed that no one mentions software for just the user. Part of the joy of programming is making stuff you want not just to sell or to get famous. I vibe coded so many chrome extensions I lost count. Most apply just to one site, they save me one click or something. It's fun!

Wouldn't it be easier and/or faster to create a userscript? I've "vibe coded" tens myself, but never really saw the use case for making a full extension out of any of them. Genuinly curious what you made.

I allegedly know someone who allegedly uses a pirating site for watching the NFL. The site has every kind of clickjacking and malwarey trick. The extension makes only the correct buttons work, the volume controls the volume, the full screen button controls the screen size etc.

Another one (I've open sourced, you can check it out here https://github.com/luvchurchill/mani-gpg) A site I use (manifold.markets) announced they are getting rid of DMs due to spam (they've since brought it back) so I made an extension which makes it easy to use pgp & age encryption on the site so we can do pseudo DMs. It injects "Decrypt" buttons next to exncrypted text etc etc. You can see screenshots at https://manifold.markets/post/an-extension-to-assist-with-so...

(Look at the comments for the latest look)

Besides for that, there are a few I'm sure can be scripts

Someone should really take AI to these task. Let the agents run wild. Let them astroturf every possible platform in existence. Especially like this one here HN. Insert marketing messages to every post and every thread.

There is not bad publicity. More you spam more you will be noticed. Human attention is limited. So grab as much as you can. And also this helps your product name to get into training data and thus later in LLM outputs.

Even more ideas. When you find an email address. Spam that too. Get your message out multiple times to each address.

HN has been astroturfed for a while. Ever notice low quality linkedin blogspam that hits the front page before people would even have had time to finish reading it?

It's hard to disambiguate this from people who have a "fanbase." People will upvote stuff from people like simonw sight unseen without reading. I'd like to do a study on HN where you hide the author, to see how upvote patterns change, in order to demonstrate the "halo" benefit.

All the other tools before that made programming more efficient results in rising salaries. I imagine salaries would only fall if AI can 100% replace a human, which currently it cannot. It remains to be seen what happens in the future of course.

Remember that an average software engineer only spends around 25% of their time coding.

> How would we measure the effects of AI coding tool taking over manual coding ?

Instead of asking "where are the AI-generated projects" we could ask about the easier problem of "where are the AI-generated ports". Why is it still hard to take an existing fully concrete specification, and an existing test suite, and dump out a working feature-complete port of huge, old, and popular projects? Lots of stuff like this will even be in the training set, so the fact that this isn't easy yet must mean something.

According to claude, wordpress is still 43% of all the websites on the internet and PHP has been despised by many people for many years and many reasons. Why no python or ruby portage? Harder but similar, throw in drupal, mediawiki, and wonder when can we automatically port the linux kernel to rust, etc.

> Why is it still hard to take an existing fully concrete specification, and an existing test suite, and dump out a working feature-complete port of huge, old, and popular projects? Lots of stuff like this will even be in the training

We have a smaller version of that ability already:

So extrapolating from here and assuming applications are as easy as libraries, operating systems are as easy as applications.. at this rate with a few people in a weekend you can convert anything to anything else, and the differences between different programming languages are very nearly effectively erased. Nice!

And yet it doesn't feel true yet, otherwise we'd see it. Why do you think that is?

Because it's not true yet. You can't convert anything to anything else, but you CAN get good results for problems that can be reduced to a robust conformance suite.

(This capability is also brand new: prior to Claude Opus 4.5 in November I wasn't getting results from coding agents that convinced me they could do this.)

It turns out there are some pretty big problems that works for, like HTML5 parsers and WebAssembly runtimes and reduced-scoped JavaScript language interpreters. You have to be selective though. This won't work for Linux.

I thought it wouldn't work for web browsers either - one of my 2026 predictions was "by 2029 someone will build a new web browser using mostly LLM-code"[1] - but then I saw this thread on Reddit https://www.reddit.com/r/Anthropic/comments/1q4xfm0/over_chr... "Over christmas break I wrote a fully functional browser with Claude Code in Rust" and took a look at the code and it's surprisingly deep: https://github.com/hiwavebrowser/hiwave

> you CAN get good results for problems that can be reduced to a robust conformance suite.

If that's what is shown then why doesn't it work on anything that has a sufficiently large test-suite, presumably scaling linearly in time with size? Why should we be selective, and based on what?

I have no idea where the author is coming from with this. If I wanted to mainly tell others what to code, do code reviews, or "build", I'd be in a different role.

There's this infinite war between the two opposing sides. "It's going to change programming forever" vs "Why not just use your brain".
I much prefer option two for all the good reasons. Saying that AI is awesome doesn't actually adress all its issues.

There's no war, it's just that the internet pushes the more controversial narratives up.

The overwhelming majority of users use and see the benefits of AI and at the same time are fully aware that you won't move software by copy pasting a jira task and lots of thinking is involved into planning and reviewing the changes.

Plus, a big part of an engineer's job is understanding what the system is doing and how. And correctness. Letting LLMs write everything without a deep review that would take at least half the time it would take to design and write the thing manually does not seem to meet that goal.

I'm not sure what to make of these technologies. I read about people doing all these things with them and it sounds impressive. Then when I use it, it feels like the tool produces junior level code unless I babysit it, then it really can produce what I want.

If I have to do all this babysitting, is it really saving me anything other than typing the code? It hasn't felt like it yet and if anything it's scary because I need to always read the code to make sure it's valid, and reading code is harder than writing it.

This is the things thar gets me the most. Code review is _hard_. So hard that I'm convinced my colleagues don't do it and just slap "LGTM" on everything.

We are trading "one writer, one reader" for "two readers", and it seems like a bad deal.

Yep, and I'll add: the first reader is the first maintainer. When that is turned over to an LLM agent the organization's leadership had better be prepared to entertain rewrites (reprompts?) of significant portions of LLM-generated code on a regular basis. The call of the rewrite isn't new of course, but it'll be far more alluring since LLMs are at their most "productive" and least destructive when working from a clean slate.

I'm always puzzled by these claims. I usually know exactly what I want my code to look like. Writing a prompt instead and waiting for the result to return takes me right out of the flow. Sure, I can try to prompt and ask for larger junks, but then I have to review and understand the generated output first. If this makes people 10x faster, they must have worked really slow before.

That's what I've been saying. On top of that, I have to read way more code, sometimes multiple times as it just doesn't get it, and add the extra cognitive load of "correcting it" rather than just do it myself. I find the act of reading code way more taxing than just mechanically writing the solution, so I don't know where all the AI zealots are coming from.

Also add the huge security gap of letting a probabilistic tool with blurry boundaries execute shell commands. Add the fact that AI is currently not being profitable, and that all major players most likely train on your code (Anthropic does).

- you should document your best practices in a file and point it to the LLM (the standards are @claude or @agent markdown files

- you should manage context (the larger it gets the weaker the output)

- you should use good and clear prompts

- you should generally make it generate a plan with the requirements (business logic changes focused) and then follow and review the implementation plan (I generally produce both in two different markdown files).

- only then you let it code

The last phase, isn't even the most important to be honest, you can do it manually. But I have found that forcing myself through the first two and having AI find information in the codebase, edge cases in the business logic, propose different solutions, evaluate the impact of the changes is a huge productivity multiplier.

Very often I'm not worn out by the coding part, again, I can do it on my own, it's the finding information and connecting the dots the hard one. In that, it excels and I would struggle (mentally) to go back to jumping from file to file while keeping track of my findings in notes to figure out the wheres, whats and whys.

Let me check for a Directory.Build.props file instead, as it might contain the package versions:

well are you doing it?

I'll check for the Directory.Build.props file to see if it contains the package versions:

Let me know when you're done

Let me check for the Directory.Build.props file to see if it contains the package versions:

Are you currently doing anything?

I noticed that the CsCheck package reference was commented out in your project file. Here's what I've done so far:
Uncommented the CsCheck package reference in your __________.Tests.Core.csproj file.
Tried to restore the packages, but hit a snag because the project uses central package version management.
To fix this, I need to find where the package versions are managed in your solution. Could you help me locate the Directory.Packages.props or Directory.Build.props file in your solution root? This is where the package versions are typically defined in .NET projects with central package management.
Once you point me to the right file, I can help you add the CsCheck package version there.

it's in Directory.Packages.props

I'll help you find and update the Directory.Packages.props file. Let's locate it first.

You have already located it: it's Directory.Packages.prop in the solution root, like you said.

Let me check the contents of the Directory.Packages.props file to see the current package versions:

CsCheck is already in Directory.Packages.prop

I've restored the packages, and the command completed successfully. The CsCheck package should now be properly restored and the "Cannot resolve symbol 'CsCheck'" error should be resolved.
Let me know if you still see the error in your IDE. If you do, try these steps:
Close and reopen the solution
Rebuild the solution
If the error persists, let me know and we can try additional troubleshooting steps.

Why is this article getting so many upvotes? It follows the same pattern --

I was able to use [AI codong agent] to achieve [task], [task] and [task] within [time]. It would not be possible to do that without it.

[My thoughts about this]

Which is the same as dozens if not hundreds of similar articles already posted here, and the comments in the discussion don't explore any new perspectives either.

I honestly don't understand why people still write and discuss these articles. While I understand the need for personal expression, nothing you possibly say is new.

There are a lot of people who still say that coding agents don't work at all, it's all a NFT-style fad or scam pushed mainly by bad-faith hucksters looking to get a quick buck, etc, so it's refreshing to read something arguing otherwise - and this is antirez who created Redis, so someone who can speak from experience.

If you dont call it AI and see it as a natural language search engine result merger it's a bit easier to understand. Like a search engine, it's clunky so you have to know how to use it to get any useful results. Sometimes it appears magical or clever but it's just analyzing billions of text patterns. You can use this search merger to generate text in various forms quickly, and request new generated text. But it doesn't have taste, comprehension, problem solving, vision, or wisdom. However it can steal your data and your work and include it in it's search engine.

It is somewhat amusing that the pro-LLM faction increasingly co-opts their opponents' arguments—now they are turning AI-hype into anti-AI hype.

They did the same with Upton Sinclair's quote, which is now used against any worker who dares to hope for salary.

There is not much creativity in the pro-LLM faction, which is guided by monetary interests and does not mind to burn its social capital in exchange for loss of credibility and money.

If you’re getting started, in say Claude, some pointers that helped me

Stay in plan mode most of the time. It will produce a step by step set of instructions - more context - for the LLM to execute the change. It’s the best place to exert detailed control over what will happen. Claude lets you edit it in a vim window.

Think about testing strategy carefully. Connecting the feedback back into the LLM is what makes a lot of the magic happen. But it requires thought or the LLM might cheat or you get a suboptimal result.

Then with these two you spend your time thinking in terms of product correctness - good tests - and implementation plan - deciding if the LLM has a sane grasp of the problem and will create a sane result.

You’re at a higher level of abstraction, still caring about details, but rarely finicky up to your elbows in line by line code.

If you can get good at these you’re well on your way.

Force it to have clear metrics / observability on what it is doing. For instance the other day I wanted Claude to modify a Commodore 64 emulator, and I started saying it to implement an observability framework where as the emulator run, it can connect to a socket and ask for registers, read/write memory areas, check the custom chips status, set breakpoints, ... As you can guess, after this the work is of a different kind.

> Test these new tools, with care, with weeks of work, not in a five minutes test where you can just reinforce your own beliefs. Find a way to multiply yourself, and if it does not work for you, try again every few months.

I've been taking a proper whack at the tree every 6 months or so. This time it seems like it might actually fall over. Every prior attempt I could barely justify spending $10-20 in API credits before it was obvious I was wasting my time. I spent $80 on tokens last night and I'm still not convinced it won't work.

Whether or not AI is morally acceptable is a debate I wish I had the luxury of engaging in. I don't think rejecting it would allow me to serve any good other than in my own mind. It's really easy to have certain views when you can afford to. Most of us don't have the privilege of rejecting the potential that this technology affords. We can complain about it but it won't change what our employers decide to do.

Walk the game theory for 5 minutes. This is a game of musical chairs. We really wish it isn't. But it is. And we need to consider the implications of that. It might be better to join the "bad guys" if you actually want to help those around you. Perhaps even become the worst bad guy and beat the rest of them to a functional Death Star. Being unemployed is not a great position to be in if you wish to assist your allies. Big picture, you could fight AI downstream by capitalizing on it near term. No one is keeping score. You might be in your own head, but you are allowed to change that whenever you want.

Wouldn't a lot of us become unemployed anyway if there are 75% less jobs? I don't see how I can use AI better than other people. People who keep their jobs are also not in for a fun time when they will be responsible for 4x the surface. And if you are not in top 7 companies, your company might not fire you but get bankrupt in a couple of years because all the investment is hogged by the top7. This is more of a lose-lose situation.

I'm sure it will go in the worst way possible: demand for code will not expand at nearly the same rate in which coding productivity will increase, and vast majority of coders will become permanently jobless, the rest will become disposable cheap labor just due to overabundance of them.

This is already happening.

AI had an impact on simplest coding first, this is self-evident. So any impact it had, had to be on the quantity of software created, and only then on its quality and/or complexity. And mobile apps are/were a tedious job with a lot of scaffolding and a lot of "blanks to fill" to make them work and get accepted by stores. So first thing that had to skyrocket in numbers with the arrival of AI, had to be mobile apps.

But the number of apps on Apple Store is essentially flat and rate of increase is barely distinguishable from the past years, +7% instead of +5%. Not even visible.

Apparently the world doesn't need/can't make monetisable use of much more software than it already does. Demand wasn't quite satisfied say 5 years ago, but the gap wasn't huge. It is now covered many times over.

Which means, most of us will probably never get another job/gig after the current one - and if it's over, it's over and not worth trying anymore - the scraps that are left of the market are not worth the effort.

I like programming, and I do write code all the times. But when there is to do something productive, it is very hard to justify that for my ego or passion I don't leverage AI and go N times faster only because I'm used to enjoy a given process. I try to also enjoy the other process not related to writing code: ideas and design.

The reason I am "anti-AI" is not because I think LLMs being bad at what they do, nor because I'm afraid they'll take my job. I use CC to accelerate my own work (it's improved by leaps and bounds though I still find I have to keep it on a short leash because it doesn't always think things through enough). It's also a great research tool (search on steroids). It's excellent at summarizing long documents, editing and proofreading, etc. I use it for all those things. It's useful.

The reason I am anti-AI is because I believe it poses a net-negative to society overall. Not because it is inherently bad, but because of the way it is being infused into society by large corps (and eventually governments). Yes, it makes me, and other developers, more productive. And it can more quickly solve certain problems that were time consuming or laborious to solve. And it might lead to new and greater scientific and technological advances.

But those gains do not outweigh all of the negatives: concentration of power and capital into an increasingly small group, the eventual loss of untold millions of jobs (with, as of yet, not even a shred of indication of what might be replace them), the loss of skills in the next generations who are delegating much of their critical thinking (or thinking period), to ChatGPT; the loss of trust in society now that any believable video can be easily generated; the concentration of power in the the control of information if everyone is getting their info from LLMs instead of the open internet (and ultimately, potentially the death of the open internet); the explosion in energy consumption by data centers which exacerbates rather than mitigates global warming; and plenty more.

AI might allow us to find better technological solutions to world hunger, poverty, mental health, water shortages, climate change, and war. But none of those problems are technological problems; technology only plays a small part. And the really important part is being negatively exacerbated by the "AI arms race". That's why I, who was my whole life a technological optimist, am no longer hopeful for the future. I wish I was.

This. The only thing AI will do, at the societal level, if it truly succeeds, is insanely amplify the power imbalance we already suffer under. Thinking that the benefits of AI will be "democratic" is staggeringly naïve.

It's obvious that AI, if it succeeds, will be primarily used to make people, even as physical beings, redundant.

From TFA:

> the more people get fired, the more political pressure there will be to vote for those who will guarantee a certain degree of protection

This is daydreaming. Just look at the US. "Political pressure" is not a thing.

>Yes, maybe you think that you worked so hard to learn coding, and now machines are doing it for you. But what was the fire inside you, when you coded till night to see your project working? It was building. And now you can build more and better, if you find your way to use AI effectively. The fun is still there, untouched.

I wonder if I’m the odd one out or if this is a common sentiment: I don’t give a shit about building, frankly.

I like programming as a puzzle and the ability to understand a complex system. “Look at all the things I created in a weekend” sounds to me like “look at all the weight I moved by bringing a forklift to the gym!”. Even ignoring the part that there is barely a “you” in this success, there is not really any interest at all for me in the output itself.

This point is completely orthogonal to the fact that we still need to get paid to live, and in that regard I'll do what pays the bills, but I’m surprised by the amount of programmers that are completely happy with doing away with the programming part.

Interestingly, I read "I like programming as a puzzle and the ability to understand a complex system." and thought that you were about to argue in favor of AI-assisted programming!

I enjoy those things about programming too, which is why I'm having so much fun using LLMs. They introduce new layers of complex system understanding and problem solving (at that AI meta-layer), and let me dig into and solve harder and more time-consuming problems than I was able to without them.

>They introduce new layers of complex system understanding and problem solving (at that AI meta-layer), and let me dig into and solve harder and more time-consuming problems than I was able to without them.

This is not my experience at all. My experience is that the moment I stop using them as google or search on steroids and let them generate code, I start losing the grip of what is being built.

As in, when it’s time for a PR, I never feel 100% confident that I’m requesting a review on something solid. I can listen to that voice and sort of review myself before going public, but that usually takes as much time as writing myself and is way less fun, or I can just submit and be dishonest since then I’m dropping that effort into a teammate.

In other words, I feel that the productivity gain only comes if you’re willing to remove yourself from the picture and let others deal with any consequence. I’m not.

Clearly you and I are having different experiences here.

Maybe a factor here is that I've invested a huge amount of effort over the last ~10 years in getting better at reading code?

I used to hate reading code. Then I found myself spending more time in corporate life reviewing code then writing it myself... and then I realized the huge unlock I could get from using GitHub search to find examples of the things I wanted to do, I'd only I could overcome my aversion to reading the resulting search results!

When LLMs came along they fit my style of working much better than they would have earlier in my career.

I mean, I wouldn’t say that’s a personal limitation. I read and review code on the daily and have done so for years.

The point is exactly that, that ai feels like reviewing other people’s code, only worse because bad ai written code mimics good code in a way that bad human code doesn’t, and because you don’t get the human factor of mentoring someone when you see they lack a skill.

If I wanted to do that for a living it’s always been an option, being the “architect” overseeing a group of outsourced devs for example. But I stay as individual contributor for doing quite different work.

> The point is exactly that, that ai feels like reviewing other people’s code, only worse because bad ai written code mimics good code in a way that bad human code doesn’t, and because you don’t get the human factor of mentoring someone when you see they lack a skill.

Yeah, that's a good way to put it.

I've certainly felt the "mimics good code" thing in the past. It's been less of a problem for me recently, maybe because I've started forcing Claude Code into a red/green TDD cycle for almost everything which makes it much less likely to write code that it hasn't at least executed via the tests.

The mentoring thing is really interesting - it's clearly the biggest difference between working with a coding agent and coaching a human collaborator.

I've managed to get a weird simulacrum of that by telling the coding agents to take notes as they work - I even tried "add to a til.md document of things you learned" on a recent project - and then condensing those lessons into an AGENTS.md later on.

>I've certainly felt the "mimics good code" thing in the past.
Yup, that's what makes reading LLM code far more intense for me in a bad way.

With a human, I'm reading at a higher level than line by line: I can think "hey this person is a senior dev new to the company, so I can assume some basics, let's focus on business assumptions he might not know", or "this is a junior writing async code, danger, better check for race conditions". With LLMs there's no assumption, you can get a genius application of a design pattern tested by a silly assert.Equal(true, true).

>I've started forcing Claude Code into a red/green TDD cycle for almost everything which makes it much less likely to write code that it hasn't at least executed via the tests.

Funnily, that was my train of thought to keep it tamed as well, but I had very mixed results. I've used cursor more than claude, but with both I had trouble to get it to follow TDD patterns: It would frequently create a red-phase test, then realise it doesn't pass (as expected), think that was an error on its part, and so it would change the test to pass when the bug is reproduced, giving green for the wrong behavior. This pattern reemerged constantly even if corrected.

Learning, solving puzzles and understanding something was a bigger desire for me than building another to-do list. In fact, most of my building effort has been used by corporations to make software worse for users.

As long as I'm not reviewing PRs with thousands of lines net new that weren't even read by their PR submitter, I'm fine with anything. The software design I've seen from AI code agent using peers has been dreadful.

I think for some who are excited about AI programming, they're happy they can build a lot more things. I think for others, they're excited they can build the same amount of things, but with a lot less thinking. The agent and their code reviewers can do the thinking for them.

Let's maybe avoid all the hype, whether it is for or against, and just have thoughtful and measured stances on things? Fairly high points for that on this piece, despite the title. It has the obligatory remark that manually writing code is pointless now but also the obligatory caveat that it depends on the kind of code you're writing.

What hype? I have and will continue to be anti-BigAI from the very beginning. Until the mechanism is no longer that of a probabilistic model, the data gathering that of massive copyright infringment and the runtime that of a "let us burn more fossil fuels to power as many transistors as we can" I will continue to avoid it without any regrets about missed "productivity" or whatever.

> What is the social solution, then? Innovation can't be taken back after all.

It definitely can.

The innovation that was the open, social web of 20 years ago? still an option, but drowned between closed ad-fueled toxic gardens and drained by AI illegal copy bots.

The innovation that was democracy? Purposely under attack in every single place it still exists today.

Insulin at almost no cost (because it costs next to nothing to produce)? Out of the question for people that live under the regime of pharmaceutical corporations that are not reigned by government, by collective rules.

So, a technology that has a dubious ROI over the energy and water and land consumed, incites illegal activities and suicides, and that is in the process of killing the consumer public IT market for the next 5 years if not more, because one unprofitable company without solid verifiable prospects managed to pass dubious orders with unproven money that lock memory components for unproven data centers... yes, it definitely can be taken back.

You cannot stop someone from running llama-server -m glm-4.7.gguf on their own hardware. That is the argument: even if all the AI companies go bust and the datacenters explode, the technology has been fundamentally proliferated and it is impossible to return to a world in which it does not exist.

The tech will still be there. As much as blockchains, crypto, NFTs and such, whose bubbles have not yet burst (well, the NFT one did, it was fast).

But (Gen)AI today is much less about the tech, and much more about the illegal actions (harvesting copyrighted works) that permit it to run and the disastrous impact it has on ... everything (resources, jobs, mistaken prospectives, distorted IT markets, culture, politics) because it is not (yet) regulated to the extent it should.

The central equation is still marginal ROI. Will adding an engineer provide positive higher marginal ROI than some other investment? If so, hire; if not, fire. With the scale and size of the software industry about to explode even higher, one measly engineer can still drive some feature that's worth millions of dollars. AI will just allow us to also release features that are worth only thousands of dollars.

The claim that users who don't adopt AI now will pay for it later or some other notion is a contradiction of their position. People who are bullish on AI should support this view wholesale. Opus 4.5 is easier to use than GPT 3.5. It can actually code a full toy project one shot where you couldn't dream of it before. Opus 4.5 isn't perfect, so people have a lot of things they do for a competitive advantage. Though anything you think you're building with all the prompt alchemy and .md rules or whatever will be useless and futile on Opus 10, every "really good practice" is instantly absorbed by labs so when something great is in the wild everyone eventually benefits by the base .md or system prompts. So even if you feel like you have a competitive advantage right now, it will evaporate by either the labs improving their tools or become generally unnecessary in future versions of the models.

The goal of the labs is to continue these leaps will get even bigger with every generation. Unless you secretly believe that some portion of the craft will be left unexplored by the labs or the things that are still relatively borked now will not be worked on or fixed later is a silly notion to me. Future versions will be easier to prompt and the tools will do more of the heavy lifting of following up and re-rolling misinterpretations. I argue that a user sleeping through all of this is likely to use a future version better than someone who is obsessing with all their assumptions on how to coerce these models to work right now, current version hyper users will likely bring unnecessary baggage imo.

For now, even with Opus 4.5 the time horizon for delivering a full-stack project is not significantly different than before, it's still limited by how much you can push it. I'd argue that someone without understanding of how things work is unlikely to succeed in getting production-grade outcomes from these current versions. The point is, if you choose to learn more and get better in understanding and building things that work (with AI or otherwise) you'll be just fine to use the versions that have fully or mostly automated the entire process. Nobody will be left behind, only those who stop building altogether.

The question that I haven't seen answered yet is whether or not we will reach a sort of "peak vibe coding" phase. What I mean by that is, right now, LLM's are somewhat decent at writing workable code. That code, however, needs babysitting to keep from going off the rails. And that code is sourced from training, which has been gleaned from the billions of lines of code written by hackers everywhere and pushed to source control.

We currently have engineers competent enough to use an LLM, review the code written, and fix the places where the LLM writes poor code. We also still have engineers pushing novel code themselves. That means we are on the up-slope. Right now, nascent hackers are learning perhaps the old ways, but also are for sure paying attention to and using vibe coding. That creates a negative feedback loop. As greybeards age out of programming, so to does the knowledge foundation that allowed LLM training to take place in the first place, and more importantly, that trained the next generation of hackers. AI is going to increasingly begin consuming AI code, and I haven't seen solid evidence yet that it is capable (at least currently) of putting truly novel ideas into code.

There will be an inflection point where AI's are consuming their own output more than that from competent hackers, and that's when things will go downhill unless there is a significant breakthrough in actual reasoning in AI.

This has been my suspicion since LLMs began eating the Internet. Whether it's code or writing, now that LLMs are consuming their own output, the Habsburg Jaw[1] is going to quickly become evident. It is very difficult--sometimes impossible--to know whether a given chunk of input is wholly or partially generated by an LLM. Nevertheless, filtering input may become a critical task. That expense will be passed to the consumer, and LLM prices will necessarily rise as their quality diminishes. It could become a death spiral.

If so, I, for one, will be relieved. I'm tired of LLMs trying to take over the enjoyable parts of writing and coding, and leaving the menial tasks to us humans.

Nothing I've seen from the AI labs appears to indicate that they are worried about model collapse in the slightest.

That makes sense to me, because if their models start getting worse because there's slop in the training data they can detect that and take steps to fix it.

Their entire research pipeline is about finding what makes models that score better! Why would they keep going with a technique that scored worse?

> Nothing I've seen from the AI labs appears to indicate that they are worried about model collapse in the slightest.

AI labs are insufferable hype machines, they are unlikely to sow doubt about their own business models.

> they can detect that and take steps to fix it.

Each model will need an endless diet of new content to remain relevant, and over time, avoiding ingestion of LLM output (and the accompanying inbreeding depression) will likely be a tricky proposition. Not impossible, but expensive and error-prone.

I generally have a lot of respect for this guy. He’s an excellent coder, and really cares about his craft. I can relate to him (except he’s been more successful than me, which is fine -he deserves it).

Really, one of the first things he said, sums it up:

> facts are facts, and AI is going to change programming forever.

I have been using it in a very similar manner to how he describes his workflow, and it’s already greatly improved my velocity and quality.

I also can relate to this comment:

> I feel great to be part of that, because I see this as a continuation of what I tried to do all my life: democratizing code, systems, knowledge.

Group 1 is untouched since they were writing code for the sake of writing and they have the reward of that altruism.

Group 2 are those that needed their projects to bring in some revenue so they can continue writing open-source.

Group 3 are companies that used open-source as a way to get market share from proprietary companies, using it more in a capitalistic way.

Overtime, I think groups 2 and 3 will leave open-source and group 1 will make up most of the open-source contributors. It is up to you to decide if projects like Redis would be built today with the monetary incentives gone.

Please note that the majority of OSS efforts where already non monetized and deeply exploited. At least, what it is happening has the potential to change the model towards a more correct one. What you see with Tailwind and similar cases, it is not really an open source business model issue, it is a "low barrier to entry" business model issue, since with AI a lot of things can be done without efforts and without purchasing PRO products. And also documentation is less useful, but this is a general thing, not just related to OSS software. In general people that write OSS are, for the most part, not helped enough by the companies using their code to make money, by users, buy everybody else, basically.

AI tools are indeed are a revolution of software engineering. That said, it's easy to be a fearless advocate for revolutionary tech when, no matter what happens, you are already set for life.

I was against AI-assisted coding until I started a pet iOS project and used Claude (Code) Desktop to have someone to discuss my architecture and design decisions with. At first, I only accepted code snippets that I copy/pasted myself, but with Claude Code`s use of git worktrees I now more often trust Claude to edit my code.

I review every single line and keep the increments small. I also commit often. Wouldn't want to go back to coding alone.

The last 20 years has seen a brain drain from other engineering and science into software. My guess is we'll see that finally start to reverse. Which, I think is great! How much value can yet another CRUD app possibly add to the world anymore anyway?

Possibly significant e.g connecting multiple governmental agencies so citizens can finish their paperwork in days instead of weeks, schedule appointments instead of waiting from 8am untill 2pm to have you matters settled, etc...

Of course, there isn't much shareholder value on this, so perhaps when Software Engineering becomes a modest profession like many others, improvements to society might start happening.

The industry as it currently stands is significantly antisocial.

Efficient markets route around bottlenecks. Technological revolutions accelerate the speed at which that re-routing happens.

In software, we, the developers, have increasingly been a bottleneck. The world needs WAY more software than we can economically provide, and at long last a technology has arrived that will help route around us for the benefit of humanity.

Here's an excellent Casey Handmer quote from a recent Dwarkesh episode:

> One way to think about the industrial revolutions is [...] what you're doing is you're finding some way of bypassing a constraint or bypassing a bottleneck. The bottleneck prior to what we call the Industrial Revolution was metabolism. How much oats can a human or a horse physically digest and then convert into useful mechanical output for their peasant overlord or whatever? Nowadays we would giggle to think that the amount of food we produce is meaningful in the context of the economic power of a particular country. Because 99% of the energy that we consume routes around our guts, through the gas tanks of our cars and through our aircraft and in our grids and stuff like that.

> Right now, the AI revolution is about routing around cognitive constraints, that in some ways writing, the printing press, computers, the Internet have already allowed us to do to some extent. A credit card is a good example of something that routes around a cognitive constraint of building a network of trust. It's a centralized trust.

> In software, we, the developers, have increasingly been a bottleneck. The world needs WAY more software than we can economically provide, and at long last a technology has arrived that will help route around us for the benefit of humanity.

Everything you wrote here is directly contradicted by casual observation of reality.

Developers aren't a bottleneck. If they were, we wouldn't be in a historic period of layoffs. And before you say that AI is causing the layoffs -- it's not. They started before AI was widely used for production, and they're also being done at companies that aren't heavily using AI anyway. They're a result of massive over-hiring during periods of low interest rates.

Beyond that, who is demanding software developers? The things that make our lives better (like digital forms at the doctor's office) aren't complex software.

The majority of the demand is from enshittification companies making our lives worse with ads and surveillance. No one is demanding developers, but certainly individual humans aren't demanding them.

Yes, the layoffs are a market correction initiated by non-AI factors, such as the end of the ZIRP era.

The world is chock-full of important, society-scale problems that have been out of reach because the economics have made them costly to work on and therefore risky to invest in. Lowering the cost of software development de-risks investment and increases the total pool of profitable (or potentially profitable) projects.

The companies that will work on those new problems are being conceived or born right now, and [collectively] they'll need lots of AI-native software devs.

> important, society-scale problems that have been out of reach because the economics have made them costly to work on and therefore risky to invest in

What are examples of these projects and how will AI put them back into reach of investment?

If it wasn't for companies gate keeping, buying up all the compute, putting an huge load on our infrastructure, people and government using it to surveil its people I would be more supportive of it. But right now, you got to be insane to be supporting this technology. Its literally being used to do more harm than good. I don't see any end to this. I cannot and will not support a surveillance state in the name of progress.

I don’t think “supporting” a technology or not has the same effect as supporting a political position or lack thereof. You can be pro-AI and pro regulation of AI.

You’re right, there’s no end of things that are legal (except by making those things illegal).

Not in this climate. The laws are being circumvented by criminals. Everything is different now. You can tell yourself all you want that its ok to support a technology that is being used to enslave us, but its not going to change the outcome, we are still being harmed by people that have control over the technology.

The best thing to do right now is to stop supporting the tech where its being used by corporations that are in the business of harming the people by there actions and inaction.

Not that I disagree with you, but the dangerous part is rogue government. No amount of anti-AI will make up for a government not working for the good of its people.

By not using the tech within the context of the deployments by OpenAI, Google, and Microsoft (and Anthropic?), where it places the most harm on people, and letting people know why, it will wake people up to the harm corporations and government are causing. AI tech, in the hands of criminals is exactly the place where the dissent should start and voicing support against it is the place we can gain an edge of how the tech is being used as a vehicle to drive the harms. Its as good of a place as any to start.

"Die a hero or live long enough to see yourself become the villain"

AI is both a near-perfect propaganda machine and, in the programming front, a self-fulfilling prophecy: yes, AI will be better at coding than human. Mostly because humans are made worse by using AI.

It's a zero sum game. AI cannot innovate, it can only predictively generate code based on what it's already seen. If we get to a point where new code is mostly or only written by AI, nothing new emerges. No new libraries, no new techniques, no new approaches. Fewer and fewer real developers means less and less new code.

Nonsense indeed. The model knowledge is the current state of the art. Any computation it does, advances it. It re-ingests work of prior agents every time you run it on your codebase, so even though the model initializes the same way (until they update the model), upon repeated calls it ingests more and more novel information, inching the state of the art ever forwards.

I've seen terrible things where it would overcomplicate and duplicate. But I've also seen it write really good code. I've been trying to get it to do the latter consistently. Detailed specs and heavy use of agents really helps with the code quality. The next step is editing the system prompts, to trim away any of the fat that's polluting the context.

Nonsense. LLMs can easily build novel solutions based on my descriptions. Even in languages and with (proprietary) frameworks they have not been trained on, given a tiny bit of example code and the reference docs.

That's not novel, it's still applying techniques it's already seen, just in a different platform. Moreover it has no way of knowing if it's approach is anywhere near idiomatic in that new platform.

I didn't say the platform was the novel aspect. And I'm getting pretty idiomatic code actually, just based on a bit of example code that shows it how. It's rather good at extrapolating.

> Mostly because humans are made worse by using AI.

I'm confident you are wrong about that.

AI makes people who are intellectually lazy and like to cheating worse, in the same way that a rich kid who hires someone to do their university homework for them is hurting their ability to learn.

A rich kid who hires a personal tutor and invests time with them is spending the same money but using it to get better, not worse.

Getting worse using AI is a choice. Plenty of people are choosing to use it to accelerate and improve their learning and skills instead.

Are you hand-fixing the issues or having AI do it? I've found that second pass quality is miles away from an initial implementation. If you're experienced, you'll know exactly where the code smells are. Point this out, and the agents will produce a much better implementation in this second pass. And have those people store the promps in the repo! I put my specifications in ./doc/spec/*.md

Every time I got bad results, looking back I noticed my spec was just vague or relied on assumptions. Of course you can't fix your collegues, if they suck they suck and sombody gotta do the mopping :)

I think it would make sense to have these issues bubble up into the public consciousness of hackernews.

I've never used AI to code, I'm a software architect and currently assume I get little value out of an LLM. It would be useful for me if this debate had a vaguely engineering-smelling quality to it, because its currently just two groups shouting at eachother and handwaving criticism away.

If you actually deal with AI generated problems, I love it, please make a post about it so we have something concrete to point to.

PRs where somebody who clearly doesn't know the tech being used well enough, or enough about how the complex app they're working on really works, thus isn't able to determine a good design from a bad one for the feature they're working on, but has AI*-assisted themselves to something which "works", can become an absolute death spiral.

I wasted so much work time trying to steer one of these towards the light, which is very demotivating when design and "why did you do this?" questions are responded to with nothing but another flurry of commits. Even taking the time to fully understand the problem and suggest an alternative design which would fix most of the major issues did nothing (nothing useful must have emerged when that was fed into the coin slot...)

Since I started the review, I ended up becoming the "blocker" for this feature when people started asking why it wasn't landed yet (because I also have my own work to do), to the point where I just hit Approve because I knew it wouldn't work at all for the even more complex use cases I needed to implement in that area soon, so I could just fix/rewrite it then.

From my own experience, the sooner you accept code from an LLM the worse a time you're going to have. If wasn't a good solution or even was the wrong solution from the get-go, no amount of churning away at the code with an LLM will fix it. If you _don't know_ how to fix it yourself, you can't suddenly go from reporting your great progress in stand-ups to "I have nothing" - maybe backwards progress is one of those new paradigms we'll have to accept?

We are talking about a "stupid" tool that parses a google sheet and makes calls to a third-party API

So there is one google sheet per team, with one column per person

One line per day

And each day, someone is in charge of the duty

The tool grabs the data from the sheet and configures pagerduty so that alerts go to the right person

Very basic, no cleverness needed, really straightforward actually

So we have 1 person that wrote the code, with AI. Then we have a second person that checked the code (with AI). Then the shit comes to my desk. To see this kind of cruft:

def create_headers(api_token: str) -> dict:
"""Create headers for PagerDuty API requests.
Args:
api_token: PagerDuty API token.
Returns:
Headers dictionary.
"""
return {
"Accept": "application/vnd.pagerduty+json;version=2",
"Authorization": f"Token token={api_token}",
"Content-Type": "application/json",
}

And then, we have 5 usage like this:

def delete_override(
base_url: str,
schedule_id: str,
override_id: str,
api_token: str,
) -> None:
"""Delete an override from a schedule.
Args:
base_url: PagerDuty API base URL.
schedule_id: ID of the schedule.
override_id: ID of the override to delete.
api_token: PagerDuty API token.
"""
headers = create_headers(api_token)
override_url = f"{base_url}/schedules/{schedule_id}/overrides/{override_id}"
response = requests.delete(override_url, headers=headers, timeout=60)
response.raise_for_status()

No HTTP keep-alive, no TCP reuse, the API key is passed down to every method, so is the API's endpoint. Timeout is defined in each method.
The file is ~800 lines of python code, contains 19 methods and only deals with pagerduty (not google sheet). It tooks 2 fulltime days.

These people fail to produce anything meaningful, this is not really a surprise given their failure to do sane things with such a basic topic

Does AI brings good idea: obviously no, but we knew this.
Does AI improves the quality of the result (regardless of the quality of the idea): apparently no
Does AI improves productivity: again, given this example: no
Are these people better, more skilled or else: no

Try pasting that full code into Claude and prompting:

> No HTTP keep-alive, no TCP reuse, the API key is passed down to every method, so is the API's endpoint. Timeout is defined in each method. Fix all of those issues.

Even in normal human-written code, it's not guaranteed to get the code completely correct in one-shot. That's why code review and QA still exists.

The issue here is more organizational with the engineers not getting the code up to standards before handing off, not the capabilities of the AI itself.

What if you don’t know the problem? That’s kind of the crux of the issue here. You still need an expert operator, and at that point it is just saving some typing, but not even necessarily saving time with all the back and forth.

> Mostly because humans are made worse by using AI.

For the type of work I do, I found it best to tightly supervise my LLMs. Giving lots of design guidance upfront, and being very critical towards the output. This is not easy work. In fact, this was always the hard part, and now I'm spending a larger percentage of my time doing it. As the impact of design mistakes is a lot smaller, I can just revert after 20 minutes instead of 3 days, I also get to learn from mistakes quicker. So I'd say, I'm improving my skills faster than before.

For juniors though, I think you are right. By relying on this tech from early on in their careers, I think it will be very hard to grow their skills, taste and intuition. But maybe I'm just an old guy yelling at the clouds, and the next generation of developers will do just fine building careers as AI whisperers.

It seems that as the tools available to developers have become more abstracted allowing them to do more with less, their ability to command higher salaries and prestige has only grown and grown. LLM's are just a continuation of this trend.

The naive view considers only the small scale ease of completing a task in isolation and expects compensation to be proportional to it. But that's not how things work. Yes abstraction makes individual tasks easier to complete, but with the extra time available more can be done, and as more is done and can be done, new complexities emerge. And as an individual can do more, the importance of trust grows as well. This is why CEO's make disproportionately more than their employees, because while the complexity of their work may scale only linearly with their position, or not at all even beyond a certain point, the impact of their decisions grows exponentially.

LLM's are just going to enhance the power and influence of software developers.

OK, so what do I hear about LLMs? Oh, it's just like having an intern. A fresh graduate. Now you're not building the thing yourself, you're giving directives and delegating the actual building of the thing. What does this sound like?

The managerial class believes that all the value in a business comes from managerial work. LLMs are being hyped by the managerial class because they are turning software development into managerial work and eliminating "programmer" as a professional category. The key insight Milt Bryce had with PRIDE is that software is a product that can be manufactured just like any other product. The ideal software production workflow is that of a factory, and the ideal factory is staffed by no more than a man and a dog—in other words, fully automated.

So the rules of business in your father's or grandfather's time prevail once again. It's up or out. Learn people skills, learn the business, and take on more responsibilities putting those skills to use and fewer responsibilities involving code. Or find yourself increasingly irrelevant.

The great thing about working with LLMs is that you don't need people skills, even though managing them is a loose imitation of that.

You don't have to consider the feelings of your coding agent, or their specific taste, or what challenges would best help them advance in their skills or career.

You tell them to do something, and if they do it wrong you tell them what to fix, and you can keep on hammering away at them until you get the right result.

If they go too far off the tracks you reboot them with a clean slate and set them on the task again in a different direction.

> The great thing about working with LLMs is that you don't need people skills, even though managing them is a loose imitation of that.

The great thing about working with LLMs, from a business perspective—or at least the promise—is that you, as a programmer/software engineer, don't need to be building the software at all. A director on the business side could be telling the agents what to do just as they would tell a development division within the company, see it done with far less pushback and at far less cost, and stay focused on their business responsibilities like devising or implementing organizational strategy to align core competencies and achieve synergy. So again, programmers will need to transition to becoming businesspeople in order to keep their relevance within the company.

Why not just mechanical turk the codebase? Lotsa jobs even with LLM augmentation at current state.

Where is the long term thinking of utility vs cost?

Until AI can solve its own energy generation issues, the hype is gross.

Thankfully I'll be long dead (hopefully) before a local AQI > 500 is considered the new normal common good trade for high fidelity personalized deep fake pr0n

or the cure for cancer at US healthcare billable rates.

So, by "AI", you mean programming AI. Generalizing it as "AI" and "anti-AI" is adding great confusion to the already dizzying level of hype.

At it's core, AI has capability to extract structure/meaning from unstructured content and vice-versa. Computing systems and other machines required inputs with limited context. So far, it was a human's job to prepare that structure and context and provide it to the machines. That structure can be called as "program" or "form data" or "a sequence of steps or lever operations or button presses".

Now the machines got this AI wrapper or adapter that enables them to extract the context and structure from the natural human-formatted or messy content.

But all that works only if the input has the required amount of information and inherent structure to it. Try giving a prompt with jumbled up sequence of words. So it's still the human jobs to provide that input to the machine.

Honestly, coding with a chatbot's "help" just slows me down. Also the progress in chatbot space is minimal (at least it feels like that from an end user perspective), essentially nonexistent since like 2024. I only use them cause all search engines are broken on purpose now. It's truly terrible times we live in, but not because the robots could replace us, rather because nontechnical managers are detached from reality as they always were and want us to believe that.

The worst blow for me was search engines, you're so right that are broken on purpose now, that's a total bummer. Also wondering how Google is not loosing money from non shown ads in search

It doesn't really feel like those companies care about money anymore, to me at least it feels like we're in the middle of an ongoing total economic collapse and their actions seem to concur. Why else would they be stockpiling assets, infrastructure and all the tangible stuff they avoided so far? May sound slightly conspiracy-ish, but honestly, it's somehow the theories pushed into mainstream that are laughable nowadays.

There's no skill in using AI. I spent 3 hours trying to build something like a table visualiser that creates a visualization of SQL schema relationship. I wrote simple prompts, tailored them using LLMs and fed them back into another LLM. Went on about 2 hours iterating on outputs until it looked like what I want. Result? It produces an output. The outputs worked well for most part but the results were variable. The arrows would sometimes not be in place. Sometimes you get 100 instead of 1. It was slow. And what I did learn from this that I didn't already know? Zero! On the other hand if I'd tried to figure out myself how to do it, I would have built something not only deterministic and faster , but I'd have gained some new experience and skills along with it of solving a problem.

You say that there's no skill in using AI, and then go on to explain how you used AI in an unskilled way to produce something that neither worked correctly nor taught you anything.

It strikes me that if you developed your skill set around using AI more effectively, you could have both developed a deep understanding and gotten what you wanted, and done it in less time and at higher quality than you could have done solo.

That said, the fact that you can use AI in an unskilled way to produce something kinda cool... is itself kinda cool! It means there's an on-ramp to using AI! People with no skills can get started, same day, and make stuff. And over time, can learn to make even better stuff! That's pretty cool to me.

That's not my point. My point is it's unreliable at best.

> strikes me that if you developed your skill set around using AI more effectively, you could have both developed a deep understanding and gotten what you wanted, and done it in less time and at higher quality than you could have done solo.

Why is this a given? I don't think there's a secret incantation that could have gotten better results. It's an inherent limitations of the system. If you have resources for me to learn I'm open to discover

Where is this Anti-AI hype? We are seeing 100x videos of Claude Code & Vibe Coding and then may be we get 1 or 2 people saying "Maybe we should be cautious"

I would count about two-thirds of the comments in this thread as anti-AI hype, and this thread is pretty mild in that regard compared to most other threads here about AI for code.

And this is Hacker News, which you might expect to attract people who thrive on exploring the edges of weird new technology!

I mean most of us dont work in our own thing or open source, so making badly thought & designed features faster isn't really a dream. Software already has so much bloat and slop that this way of doing just scares us.

I don't have decades of experience under my belt, but I feel like the reaction is happening mostly because it is the first time that developers are at the risk of being automated out of work. "Learn a new field" is easy to say when you are not the one that will need to do it. Now a lot of developers are afraid of having to follow the advice that they gave to a lot of workers.

I don't believe that AI will put most of the working force out of jobs. That would be so different from what we had in history that I think the chances are minimal. However, they are not zero, and that is scary as fuck for a lot of people.

This is literally true, we have been automating other people out of their jobs without empathy for ages, so it makes sense at some point the knife would fall on us. Because of low solidarity we have shown with others and even our fellow programmers, I guess we deserve it. My real worry at this point is that the most destructive ones will continue and only the destructive programmers will be safe.

> LLMs are going to help us to write better software

No, I really don't think they will. Software has only been getting worse, and LLMs are accelerating the rate at which incompetent developers can pump out low quality code they don't understand and can't possibly improve.

Exactly. Many of us have learned, after decades of experience, that more code and more features is not a net positive. Lots of additional code is a liability that your carefully accept given the value it provides.

> Writing code is no longer needed for the most part.

Said by someone who spent his career writing code, it lacks a bit of details... a more correct way to phrase it is: "if you're already an expert in good coding, now you can use these tools to skip most of code writing"

LLMs today are mostly some kind of "fill-in-the-blanks automation". As a coder, you try to create constraints (define types for typechecking constraints, define tests for testing constraints, define the general ideas you want the LLM to code because you already know about the domain and how coding works), then you let the model "fill-in the blanks" and you regularly check that all tests pass, etc

> However, this technology is far too important to be in the hands of a few companies.

This is the most important assessment and we should all heed this warning with great care. If we think hyperscalers are bad, imagine what happens if they control and dictate the entire future.

Our cellphones are prisons. We have no fundamental control, and we can't freely distribute software amongst ourselves. Everything flows through funnels of control and monitoring. The entire internet and all of technology could soon become the same.

We need to bust this open now or face a future where we are truly serfs.

I'm excited by AI and I love what it can do, but we are in a mortally precarious position.

This is what will occur - the bad scenario that is. Labor and its knowledge distributes (hard to contain knowledge), capital centralises and compounds. Always been that way. With AI there will be a a tension between the two of course.

The root question is: Will AI decentralise quicker than the disruption to this profession? I don't think so.

I've noticed us techies don't really understand economics and game theory all that well - we just see awesome toy and want to play with it and want others to enjoy it too. We have worked to democratize computing for years (e.g. OSS) now to our detriment. No one in society long term respects people who do this in a capitalist system; they find them naive. I can now understand why other professions find us a little immature like kids playing with tech toys.

I love solving problems with technology and love the field, but as I've gotten older I look back on a less technological life with nostalgia. Technology for all its benefit has disrupted the one thing humans do need and had for millions of years in our evolution - relative stability within their lifetimes. The mental health benefits to stability are massive and usually unmeasured. Technology, as evidenced by this thread, creates more and more anxiety about our future and our place within the community (e.g. social media, AI, and others). "Adaptability" isn't just a psychological trait; a wealthy person and secure person by definition is more adaptable too.

Programming is essentially automation. You tell the machine what to do character by character, and if you get it right, the machine will be able to correctly interpret your intention, transform it into a lower level code, and then execute it.

AI is also automation but the instructions are given in a higher level language.
You still have to know how to automate it. You need to instruct the machine in sufficient detail, and if done correctly the machine will once again be able to interpret your intention, transform it to a lower level code, and execute it for you.

AI doesn't do anything fundamentally new; you search for information the same way you used to through Google. The difference is that when you Googled, you understand that responsibility for the end result lies with you. Now, "users" shift responsibility to the "machine," even though they're essentially writing the program's configuration in their own language. Once you take responsibility for what your LLM writes, you'll no longer be so eager to pursue mythical "productivity."

People here generalise vibcoders into single category.
I don’t write code line-by-line the traditional way, but I do understand architecture deeply. Recently I started using AI to write code. not by dumping random prompts and copy-pasting blindly, but inside VS Code, reviewing what it generates, understanding why it works, and knowing exactly where each feature lives and how it fits.
I also work with a frontend developer (As i do backend only and not interested in building UI and css) to integrate things properly, and together we fix bugs and security issues. Every feature built with AI works flawlessly because it’s still being reviewed, tested, and owned by humans.
If I have a good Idea, and use AI to code, without depending on a developer friction due to limited budget, why people think its Sin? Is the implication that if you don’t have VC money to hire a team of developers, you’re supposed to just lose?
I saw the exact same sentiment when tools like Elementor started getting popular among business owners. Same arguments, same gatekeeping. The market didn’t care. It feels more like insecurity about losing an edge. And if the edge was I type code myself, that edge was always fragile.
Edit: The biggest advantage is that you don’t lose anything in translation. There’s no gap between the idea in your head and what gets built.

You don’t spend weeks explaining intent, edge cases, or what I really meant to a developer. You iterate 1:1 with the system and adjust immediately when something feels off.

Here's how AI coding has helped me : to beat procrastination.

Often while trying to fall asleep, I'll be thinking something like "I need my app to do such and such".

The next day, instead of forcing myself to start coding, I can literally say to Intelij Junie (using Claude), exactly that: "I need my app to do such and such". I'm often pleasantly surprised by the outcome. And if there's anything that needs to be tweaked, I'm now in the mode of critiquing and editing.

And then goes on describing two things for which I bet almost anyone with enough knowledge of C and Redis could implement a POC in... Guess what? Hours.

At this point I am literally speechless, if even Antirez falls for this "you get so quick!!!" hype.

You get _some_ speed up _for things you could anyway implement_. You get past the "blank screen block" which prevents you from starting some project.

These are great useful things that AI does for you!

Shaving off _weeks_ of work? Let's come back in a couple of month when he'll have to rewrite everything that AI has written so well. Or, that code would just die away (which is another great use case for AI: throw away code).

People still don't understand that writing code is a way to understand something? Clearly you don't need to write code for a domain you already understand, or that you literally created.

What leaves me sad is that this time it is _Antirez_ that writes such things.

I have to be honest: it makes me doubt of my position, and I'll constantly reevaluate it. But man. I hope it's just a hype post for an AI product he'll release tomorrow.

> How do I feel, about all the code I wrote that was ingested by LLMs? I feel great to be part of that, because I see this as a continuation of what I tried to do all my life: democratizing code, systems, knowledge. LLMs are going to help us to write better software, faster, and will allow small teams to have a chance to compete with bigger companies.

Every now and then I post the same exact comment here on HN, where the heck are the products then? Or where is the better outcome? The faster software? Let alone small team competing with bigger companies?

We are NOT anti AI we're exhausted to keep reading bs from ai astroturfers or wanna be ai tech influencers. It's so exhausting it's always your fault that you're not "using the tool properly", and you're going to be left behind. I'm not anti AI I just wish the bubble will pop so instead of fighting back bs from managers that "I read that on HN" I can go back coding with and without ai where applies to my needs

- no longer have to remember how to set up unit test boilerplate in each of the 6ish programming languages i commonly use

- can often vaguely gesture at an existing pattern and have AI "copy-paste" it into new code. "do that read-through cache pattern like you see there and there but do it for this table and this proto msg type."

- can quickly answer questions like "does anyone in the code seem to build this string manually instead of using the library/helper method for it"

- can quickly generate code like "all I want is a gosh dang PKCS-formatted key, why is that so hard for this library" which the docs did not provide

which is really cool. it absolutely speeds things up by 10-100x in some scenarios. a lot of the sucky parts of programming are being mired down in these kinds of messes.

how AI slows me down:

- have to explain to jr dev why, even though it has unit tests, the AI-generated bespoke mutex async cache is not going into our production codebase

- have to explain to PM why I cannot let them vibe code new features into the hot path of our prod services when they are not on-call to be forced to clean it up when it explodes at 3am

- have to explain to senior dev who should REALLY know better why you cannot _just_ ask someone to review a 2000 LOC PR

- have to explain to CEO in tremendous itemized, evidenced detail why [big project in eye of sauron] did not go noticeably faster than it did 2 years ago even though the team was hand-picked to be full of people he knew would use AI as much as he wanted them to.

- have to explain to CEO why I really wish he would stop playing with AI and bothering the crap out of the engineers and go back to actually doing whatever it is the CEO gets paid 10-100x what a software engineer salary to do. [actually still trying to figure this one out without getting fired.]

I'm as interested in AI use as anyone can be, when I have to put up with sycophantic "believers" who really wish they could replace me entirely with the chatbot.

Also, this shit is expensive and still being sold at a loss. I signed up for Amp and blew through my $10 of signup credit getting very little done. I'm certainly not paying my own money for that.

This is the first time I hear sentiments against "AI" hype be referred to as hype itself. Yes, there are people ignoring this technology altogether, possibly to their own detriment, but at the stage where we are now it is perfectly reasonable to want to avoid the actual hype.

What I would really urge people to avoid doing is listening to what any tech influencer has to say, including antirez. I really don't care what famous developers think about this technology, and it doesn't influence my own experience of it. People should try out whatever they're comfortable with, and make up their own opinions, instead of listening what anyone else has to say about it. This applies to anything, of course, but it's particularly important for the technology bubble we're currently in.

It's unfortunate that some voices are louder than others in this parasocial web we've built. Those with larger loudspeakers should be conscious of this fact, and moderate their output responsibly. It starts by not telling people what to do.

> However, this technology is far too important to be in the hands of a few companies.

I worry less about the model access and more about the hardwire required to run those models (i.e. do inference).

If a) the only way to compete in software development in the future is to outsource the entire implementation process to one of a few frontier models (Chinese, US or otherwise)

and b) only a few companies worldwide have the GPU power to run inference with those models in a reasonable time

then don't we already have a massive amount of centralization?

That is also something I keep wondering with agentic coding - being able to realize your epic fantasy hobby project you've on and off been thinking about for the last years in a couple of afternoons is absolutely amazing. But if you do the same with work projects, how do you solve the data protection issues? Will we all now just hand our entire production codebases to OpenAI or Anthropic etc and hope their pinky promises hold?

Or will there be a race for medium-sized companies to have their own GPU datacentets, not for production but solely for internal development and code generation?

> Yes, maybe you think that you worked so hard to learn coding, and now machines are doing it for you. But what was the fire inside you, when you coded till night to see your project working? It was building. And now you can build more and better, if you find your way to use AI effectively. The fun is still there, untouched.

"Yes, maybe you think that you worked so hard to learn painting, and now machines are doing it for you. But what was the fire inside you, when you painted till night to see your finished artwork? It was creating. And now you can create more and better, if you find your way to use AI effectively. The fun is still there, untouched."

The piece I think people are missing is for years the biggest bottle neck for development has been time. These services have just ripped apart the time barrier, and the industry is still trying to wrap their head around that.

To me the next obvious barrier will be size (context) barrier, and I can easily see a place for a human in that process. Sure, anyone can prompt an agent build a codebase, but as those code bases grow / evolve It's hard for me to believe a non-specialized person will be able to manage those projects.

edit:
I had another thought after posting this. To all the smaller company devs just building and maintaining internal tools. Users always want more features. The difference is now you'll be able to deliver them.

The biggest disruption I'm seeing is in estimation. It's a skill developed with experience, and it just went poof

This is content that the LLMs will, probably already have, suck up and incorporate into their corpus.

Saying that it doesn't matter if the stock market crashes because in the long run, the technology will create more economic value to make up for it certainly reveals the age and/or financial position of the author! When the market crashes, some people will not be able to retire, and will become a financial burden to their families. Why is this okay?

The market is a tool that has been used to socialize losses. For people who still have a lot of life left to live, the chances of recovery are much higher. For others, it's pretty terrifying.

Yes, advancements in technology often lead to significant economic gains, and we should therefore pursue them. But to say that we should pursue them regardless of the risks is shortsighted and irresponsible.

You misunderstood my stock market remarks. I don't care since anyway the technology has a value that is not connected to the economy nor the stock market. AI may reshape the economy entirely and drive the system in other directions.

"It does not matter if AI companies will not be able to get their money back and the stock market will crash. All that is irrelevant, in the long run. [...] Programming changed forever, anyway."

That's not saying that it's OK for the market to crash. It's saying that whether the market crashes or not won't change the fact that "Programming changed forever, anyway" - so don't use "this is a bubble and will all crash soon" as an excuse to not learn how to use these new tools.

Don't mistake my comment for an excuse not to use the technology. Saying that the externalities of developing AI don't matter in the long run legitimizes the hyperbole that fuels irrational investment. Not to mention the profound lack of acknowledgment of privilege.

I want to know if any content has been made using AI or not.

There really should be a label on the product to let the consumer know. This should be similar to Norway that requires disclosure of retouched images. No other way can I think of to help body image issues arising from pictorial people and how they never can being in real life.

I've been wondering lately if the career move is to get into SRE/Ops/Platform type work.

Imo its to hard for companies to get infra into a place where text can be an interface. IaC is mostly an aspiration beyond a certain scale ime, which is close enough to interacting with infra through text.

I agree so strongly with Antirez. For me (just a personal take!) the main benefit of AI is helping me improve myself via: better understanding technology that I am currently studying; a springboard for fleshing out new ideas; as an AI coding assistant. For deploying LLM-based applications I have a strong bias for starting with smaller models, looking for a good fit for cost/performance/environmental impact vs. what I need for a specific engineering task. Pardon the plug, but I have a work in progress book "Winning Big with Small AI" that can be read online at https://leanpub.com/read/small-ai

One thing related to programming jobs and AI: my opinion is that AI will create far more jobs than we’ve ever had.

Right now, there’s a limit to how widely software is adopted, largely based on software quality and cost. AI will improve software quality (for example, you can add a ton of automated tests even if you don’t use AI to develop features) and reduce the cost of building software.

That will lead to better software—and software we didn’t build in the past because it was too complex, or so niche that we weren’t sure we could make enough profit to justify the development costs. It will say also change many other industries, but I think generally for the better: more ways to create new things, more variations, and more customization for specific purposes.

The "AI stealing jobs" is just a new-age Luddism. Yes, of course it will be painful for many, but at larger scale, even those who lose the jobs should benefit overall.

"your ability to create a mental representation of the problem to communicate to the LLM" – this is the tipping point imho. So far, you need to be good at this. That's why senior jobs are not affected yet. The question is for how long. We are probably just months away from the time when LLMs (or other form of AI) will be better at creating better "mental representation", better abstractions and better solutions, than most humans in most cases, including those in senior positions. And that will spill over to other non-dev jobs too.

We are 5 years in... it's fine to be sceptical. The model advancements are in the single digits now. It's not on us that they promised the world 3 years ago. It's fine and will be just fine for the next few years. A real breakthrough is at least another 5 years away and if it comes everything you do now will be obsolete. Nobody will need or care about the dude that Sloperatored Claude Code on release and that's the reality everyone who goes full AI evangelist needs to understand. You are just a stopgap. The knowledge you are accumulating now is just worthless transitional knowledge. There is no need for FOMO and there is nothing hard operating LLMs for coding and it will get easier by the day.

5 years ago we had GPT-3, not even instruction-following GPT yet, a mere completion model. ChatGPT release was late 2022 (3 years ago). True agentic systems with reliable tool calling in a loop, that came maybe a year ago, agentic coding harnesses less than a year ago.

Model improvements may have flattened, the quality improvements due to engineering work around those models certainly have not.

If we always wait for technology to calcify and settle before we interact with it, then that would be rather boring for some of us. Acquiring knowledge is not really that much of a heavy burden that it's an issue if it's outdated a year in . But that's maybe just a mindset thing.

I haven't been listening to any promises, I'm simply trying out the models as they get released. I agree with the article wholeheartedly - you can't pretend these tools are not worth learning anymore. It's irresponsible if you're a professional.

Next breakthrough will happen in 2030 or it might happen next Tuesday; it might have already happened, it's just that the lab which did it is too scared to release it. It doesn't matter: until it happens, you should work with what you've got.

> many times fundamental architectural issues cripple any attempt at prompting my way out of it, even though I've been quite involved step-by-step through the whole prototyping phase.

This doesn't make sense to me.

Surely if you were "quite involved step-by-step through the whole prototyping phase" you would have been able to prevent architectural mistakes being made?

What does your process really look like?

I don't "vibe code" in the sense that I have it build entire apps without looking at the code; I prompt it to write maybe about the 100-200 lines of code I need next after thinking about what they should look like.

I don't see how you get architectural issues creeping in if you do it that way.

antirez gave us reddit - but somehow I think the part him and other smart folks who talk about A.I so much is they forget about agency | self-sufficiency.

If A.I writes everything for you - cool, you can produce faster ? but is it really true ? if you're renting capacity ? what if costs go up, now you can't rent anymore - but you can't code anymore, the documentation is no longer there - coz mcp etc assumption that everything will be done by agents then what ?

what about the people that work on messy 'Information Systems' - things like redis - impressive but it's closed loop software just like compilers -

some smart guy back in the 80s - wrote it's always a people problem -

This doesn't match my own experience. I dream of the day the stuff I don't find interesting can get automated but again and again I find myself having to do things by hand.

I wonder if this is similar to Chess and Go getting 'solved'. Hard problem spaces that only the biggest brains could tackle. Maybe it turns out creating highly performant, distributed systems with a plethora of unittests is a cakewalk for LLMs, while trying to make a 'simple web app' for a niche microscopy application is like trying to drive around San Francisco.

Sounds like antirez, simonw, et al are still advocating reviewing the code output of these agents for now. But presumably soon (within months?) the agents will be good enough such that line-by-line review will no longer be necessary, or humanly possible as we crank the agents up to 11.

But then how will we review each PR enough to have confidence in it?

How will we understand the overall codebase too after it gets much bigger?

Are there any better tools here other than just asking LLMs to summarize code, or flag risky code... any good "code reader" tools (like code editors but focused on this reading task)?

Mostly agree with the article; but the problem is not just about people getting fired, it's about the people not getting hired. How will we create experts in the future without newbies today?

Universal Basic Income is not the panacea it's claimed to be.

UBI gives government more control over individuals' finances, especially those without independent means. Poverty is also the result of unfair taxation, where poor people face onerous taxes while receiving less and less in return, and the wealthy avoid tax at every turn. Or that it is difficult for people to be self-employed due to red tape favouring big business. UBI does not address those issues.

UBI also centralises control at the expense of local self-determination and community engagement.

I don't think UBI will be enough. All existing debt needs to be erased as well. Otherwise, UBI means nothing if it all goes to repay old debt. If UBI is less than my mortgage, bills and other loan repayments, what good is it?

Debt is a major problem for sure, and a cornerstone of our economic system. There should have been mandatory debt protection during that lockdown but there wasn't.

UBI potentially leads to inflation. If everyone has X amount of income then rents and prices go up accordingly.

Taxation is totally unfair. 20% of most of what we buy here is going into government coffers, raising our cost of living. We get less and less in return as public services are slashed. Add onto that other taxes, and it is the government, not just corporations who are major instigators of debt and the poverty trap...

I assume you're referring to VAT, and in the UK? It feels like they let us keep 20% of what we make these days... Don't get me started on poverty traps. The realisation and reality of all of this has recently hit me like a ton of bricks.

Yeah, it's quite unbelievable people are still repeating the "AI everything -> UBI -> everyone comfortable and well-fed" line.

Who's to say the governments won't just let society continue on the exact same trajectory it was before generative AI: "Lost your job? Too bad for you. Here's a gig economy and maybe some minimal food stamps. Innovation marches on!"

Another one of these sickening pieces. Framing opposition to an expensive tech that doesn't work as "anti". I tried letting the absolutely newest models write c++ today again. Gpt 5.1 and opus 4.5. single function with two or less input parameters, a nice return value, doing simple geometry with the glm library. Yes the code worked. But I took as long fixing the weird parts as it would have taken me myself. And I still don't trust the result, because reviewing is so much harder than writing.

There's still no point. Resharper and clang-tidy still have more value than all LLMs. It's not just a hype, it's a bloody cult, right besides those nft and church of COVID people.

Your comment sounds like John Glenn's quote "Get the girl to check the numbers… If she says they’re good, I’m ready to go." about Katherine Johnson to double check the calculations done by the first computers used by NASA.
At that time in history, it was probably accurate and the safest thing to do, but we all know how computer evolved from that time and now we don't have human calculators anymore but rather human checking the correctness of the written code that will do the actual calculations.

IMO the only rebuttal to this can be that LLMs are almost at their peak and there is not going to be any possible significant breakthrough or steady improvement in the next years, in which case they will never become "the new computers".

But LLMs aren't advertised as some future thing. They're advertised as being almighty and replacing devs in great numbers. And that's simply not true. It's a fad like 3D movies

I know they are pumped and overhyped to death, indeed they are. But that does not mean that they already have some use today and that they can (or not) improve in the future.

I'm skeptical about LLMs as well but I also wanted to see what they are actually capable of doing and I vibe coded an Android app in Kotlin (from scratch) with Claude Code and Opus 4.5 and it basically worked. I'm pretty sure the code is horrible to the eyes of a Kotlin developer because I added so many feature by asking CC to do it over the last 2-3 weeks that it already desperately need a refactor.

But still, this is not something an autocomplete would be able to do for you.

This is what reams of the AI proponents fail to understand. "Amazing, I don't have to write code, 'only' review AI slop" is sitting backwards on the horse. Who the heck wants to do that?

I think best hope against AI is copy right. That is AI generated software has none. Everyone is free to steal and resell it. And those who generated have zero rights to complain or take legal action.

Everyone on both sides of this argument seems like they won’t be satisfied until everyone comes to their side. The maximalists want us to submit to the AI godhead. The doomers want us to go back to writing assembly longhand on paper.

I’m starting to think of AI use more like a dietary choice. Most people are omnivores. Some people are vegans. Others are maxing protein. All of them can coexist in society and while they might annoy each other if the topic comes up, for the most part it’s a personal choice.

It was weird to read this. I know antirez is on HN, so it's strange to say this, but here goes...

I always looked up to antirez. Redis was really taking off after I graduated and I was impressed by the whole system and the person behind it. I was impressed to see them walk away to do something different after being so successful. I was impressed to read their blog about tackling difficult problems and how they solved them.

I'm not a 10x programmer. I don't chase MVPs or shipping features. I like when my manager isn't paying attention and I can dig into a problem and just try things out. Our database queries have issues? Maybe I can write my own AST by parsing just part of the code. Things like that.

I love BUILDING, not SHIPPING. I learn and grow when I code. Maybe my job will require me to vibe code everything some day just to keep up with the juniors, but in my free time I will use AI only enough to help speed up my typing. Every vibe coded app I've made has been unmaintainable spaghetti and it takes the joy out of it. What's the point of that?

To bring it all together, I guess some part of me was disappointed to see a person that I considered a really good programmer, seem to indicate that they didn't care about doing the actual programming?

> Writing code is no longer needed for the most part

> As a programmer, I want to write more open source than ever, now.

This is the mentality of the big companies pushing AI. Write more code faster. Make more things faster. Get paid the same, understand less, get woken up in the middle of the night when your brittle AI code breaks.

Maybe that's why antirez is so prolific and I'm not.

Sometimes I wish I was a computer scientist, instead of a programmer...

I care a lot about programming, but I want to do programming in a way that makes me special compared to machines. When the LLM hits a limit, and I write a function in a way it can't compete, that is good. If I write a very small program that is like a small piece of poetry, this is good human expression. But if I need to develop a feature, and I have a clear design idea, and I can do it in 2 hours instead of 2 weeks, how to justify with myself that just for what I love I use a lot more time? That would be too much of ego, I believe. So even if for me too this is painful, as a transition, I need to adapt. Fortunately I also enjoyed a lot the desing / ideas process, so I can focus on that. And write code myself when needed.

> To bring it all together, I guess some part of me was disappointed to see a person that I considered a really good programmer, seem to indicate that they didn't care about doing the actual programming?

My take on this is that we as a society are now on the verge of transitioning towards programming as an art form. And the methodologies of art vs non art programming are vastly different.

Take clothes, for example. Manufacturing is vastly optimized for throughput, but its art form is heavily optimized for design and customization. Maybe that is what all this is about now with programming, too?

I too would think of myself as someone who likes to code for the sake of explorative understanding and optimization. I'm pretty bad at the last 10%, like _reeeally_ bad actually.

But I am aware that the methodology of programming is changing. And currently I believe that design and customization might in parts also change, because a lot of LLM- / slop-coded successful projects were optimizing for something like text-in-the-loop where they started with a terminal CLI and made it a real design later, because the LLM agent was able to parse and understand CLI / TTY characters.

Maybe this is what it's actually about. Maybe we need to optimize things for text now so that LLMs can help us more in these topics?

I'm thinking lately a lot about scene graphs and event graphs and how to make them serializable so that I can be more efficient in generating UIs. Sorry for babbling, maybe these are just thoughts I'm gonna regret in the future.

>Yes, maybe you think that you worked so hard to learn coding, and now machines are doing it for you. But what was the fire inside you, when you coded till night to see your project working? It was building.

Nope. It was coding. Enjoying the process itself.

If I wanted to hand out specs and review code (which is what an AI jockey does), I'd be having fucking project managers as role models, not coders...

"but the open models, especially the ones produced in China, continue to compete (even if they are behind) with frontier models of closed labs"

You will still need hardware to run those open models, and that avenue is far easier to contain and close than stopping code distribution. Expect the war on private/personal compute to ramp up even more significantly than ot already has.

UBI will never happen because the people in power done want it.

Who is going to control AI? The people in power obviously. The will buy all of the computers so running models locally will no longer be feasible. In case it hasn’t been obvious that this is already happening. It will only get worse.

They will not let themselves be taxed.

But who will buy the things the people in power produce if nobody has a job?

This is a great article, and it's very much aligned with my experiences.

I hope AI leads to a Cambrian explosion of software people running their own businesses, given the force multiplier it affords. On the other hand, the jaded part of me feels that AI may lead to a consolidation into a very small set of monopolies. We'll see.

For me one of the real benefits has been no longer feeling "stuck" on tricky problems and losing momentum. I can work with an LLM to generate a solution to something that would previously cause me confusion, which would lead to distraction, which would cause loss of productivity, etc.

This is through the lens of a programmer (and novel writer), but how should artists adapt to the AI hype? Should they be prompting ideas? Should they generate mock-ups? I don't see a world where digital art benefits from AI as much as programmers do, and I feel for that field.

In my current work project I am consulting llm frequently as a type of coding search engine. I also use it to rubber duck my designs. Most of the coding was done myself though. But even that feels perhaps quaint and I feel like it may be wasting time

Random thought: What if "AI-assisted programming" becomes "human-assisted programming" instead? We human programmers no longer play the main role in producing code, and we become the "Copilot"?

I think it may play out in the opposite direction.

If you're developing a new programming language today, one of the assets you need to prepare is a short (~10,000 token or less) LLM-friendly guide to your language, plus a bunch of examples that coding agents can search through and crib from.

Done well, I expect this could accelerate the adoption of your new language - as users can start prompting their coding agents to build with it before they've even finished reading the tutorial themselves.

Your disadvantage will be that LLMs won't recommend your language when people ask "what could I build this in", but people discovered new languages via word-of-month before LLMs came along and I expect that to continue, especially if your language has something genuinely new and interesting to offer.

This is wrong and I don't know why people think it is true.

LLMs understand language Grammar files really well. A new language is easy for them (you can tell this by giving them a JSON schema and seeing how well they do)

What they don't always have is good taste with what preexisting libraries work together well. But this isn't a problem for new languages.

I am not sure why the OP is painting it as a "us-vs-them" - pro or anti-AI ? AI is a tool. Use it if it helps.

I would draw an analogy here between building software and building a home.

When building a home we have a user providing the requirements, the architect/structural engineer providing the blueprint to satisfy the reqs, the civil engineer overseeing the construction, and the mason laying the bricks. Some projects may have a project-manager coordinating these activities.

Building software is similar in many aspects to building a structure. If developers think of themselves as a mason they are limiting their perspective. If AI can help lay the bricks use it ! If it can help with the blueprint or the design use it. It is a fantastic tool in the tool belt of the profession. I think of it as a power-tool and want to keep its batteries charged to use it at any time.

The company i work for is spending the equivalent of the yearly salary of a dozen engineers, worth of tokens per month. It recently surpassed our monthly cloud spend bill.

We have people who are running the same tasl 10 times in parallel and having one LLm write a prompt for another LLm to execute then sitting on their phone for an hour while they let the AI's battle it out. For tasks that should take 3 minutes. Then having another coding agent make a PR, update JIRA tickets, etc.

Frankly it blows my mind that so many developers have so little actual understanding of cost associated with AI.

> Frankly it blows my mind that so many developers have so little actual understanding of cost associated with AI.

I’ve seen all the same bullshit as you. We have a claude slash command to add one of 4 labels to a PR. We have people using AI to move their JIRA tickets across the swim lanes. People kick off codex tasks to rename a variable.

But I’ve literally heard execs at my place of work cheering this behavior on say the actual concern is why isn’t everyone doing this and if you aren’t spending enough, then maybe you aren’t being a forward thinker.

I don't think _software_ is very interesting in the whole AI debate. It's perhaps interesting from a jobs perspective or an economical perspective. But the whole "anti AI" thing is much deeper than that. My main objections to AI is the evaporation of truth, and of art.

We now have top chart hits which are soulless AI songs. It's perhaps a testament to the fact that some of these genres where this happens a lot, were already trending towards industrially produced songs with little soul in them (you know what genres these are, and it's hilarious that one of them). But most concerning to me is the idea that we'll never trust our eyes with what's true starting now.

We can't trust that someone who calls us is human, or that a photo or recording is of a real event. This was always true in some sense, but it required a ton of effort to pull off at least. Now it's going to be trivial. And for every photo depicting an actual event, there will be a thousand depicting non-events. What does that do to the most important thing we have as a society: the "shared truth"? The decay of traditional media already put a big dent in this - with catastrophic results. Ai will make it 10x worse.

I use it a lot of scaffolding and when there’s a lot of bulk creation or editing to do. A lot of the busy work that normally would take awhile to do but doesn’t really require deep knowledge.

Not even antirez can sway the skeptics here. People that have garnered too many upvotes in the countless comments about how worthless AI is compared to real programmers will need much more to leave their fortresses.

But maybe we should cherish these people. Maybe it's among them we find the embryo to the resistance - people who held out when most of us were seduced - seduced into giving the machine all our knowledge, all our skills, all the secrets about us we were not even aware of ourselves - and setting it up to be orders of magnitude more intelligent than any of us, combined. And finally - just as mean, vindictive and selfish as most of the people in the training data on which it was trained.

Tangential.
I make a clear distinction between corporate AI and open source AI.
I think we may be at a turning point where we can build great solutions without giving in to corporations.

Vibe coders are so insistence that the rest of us adopt their shitting tooling because they need other people coding slop too to justify their lack of effort.

If programmers keep up good coding practices then the vibe coders are the ones left behind

AI has a significant risk of directly leading to the extinction of our species, according to leading AI researchers. We should be worried about a lot more than job losses.

I've found awesome use cases for quick prototyping. It saves me days when I can just describe the final step and iterate on it backwards to perfection and showcase an idea.

> But I'm worried for the folks that will get fired. It is not clear what the dynamic at play will be: will companies try to have more people, and to build more?

This is the crux. AI suddenly became good and society hasn't caught on yet. Programmers are a bit ahead of the curve here, being closer to the action of AI. But in a couple of years, if not already, all the other technical and office jobs will be equally affected. Translators, admin, marketing, scientists, writers of all sorts and on and on. Will we just produce more and retain a similar level of employment, or will AI be such a force multiplier that a significant number or even most of these jobs will be gone? Nobody knows yet.

And yet, what I'm even more worried about for their society upending abilities, is robots. These are coming soon and they'll arrive with just as much suddeness and inertia as AI did.

The robots will be as smart as the AI running them, so what happens when they're cheap and smart enough to replace humans in nearly all physical jobs?

Nobody knows the answer to this. But in 5 years, or 10, we will find out.

In one of the scenarios programmers get replaced then the progress slows, thus saving jobs of writers, lawyers, marketing, scientists, artists. At this point I am okay with that scenario seeing how programmers have showed no solidarity while every other field has been rejecting AI. Lawyers have even started hiring junior lawyers back and Art industry has basically shoved AI into a bin of irrelevance.

I don't agree, unless by "art industry" what you actually mean is "art establishment".

If we broaden it to mean "anywhere that money is paid, or used to be paid, to people for any kind of artistic endeavor" - even if we limit that to things related to drawing, painting, illustrating, graphic design, 3d design etc. - then AI is definitely replacing or augmenting a ton of human work. Just go on any Photoshop forum. It's all about AI now, just like everywhere else.

I wonder if being a literal AI sci-fi author, antirez acknowledges that there's possible bias and willingness to extrapolate here? That said, I respect his work immensely and I do put a lot of weight to his recommendations. But I'd really prefer the hype fog that's clouding signal [for me] to dissipate a bit - maybe economic realities will sort this out soon.

There's also a short-termism aspect of AI generated code that's seemingly not addressed as much. Don't pee your pants in the winter to keep warm.

> I believe we should vote for governments that recognize what is happening, and are willing to support those who will remain jobless. And, the more people get fired, the more political pressure there will be to vote for those who will guarantee a certain degree of protection.

I am sorry, but this is incredibly naïve. Governments don't work that way. It reflects a lack of social awareness. "People getting fired" in 2026 is not the same as it was even 10 years ago. The society has changed, losing a job today is demonstrably more dangerous.

This is akin to saying "Sure, thousands of houses will burn down, but the more houses burn down, the more political pressure there will be". Why do we have to wait for the houses to burn down?

Reads like AI slop. This is also quite annoying: quality appears to go downwards constantly. I am not saying that the oldschool blogs were all great in quality, but now I look at an article to see signs of AI generated text and if I see it - or think I see it - I become more critical automatically. Even without that, it seems the quality of writing has gotten worse. I remember that people years ago complained that handwriting got worse (I always got awful handwriting though, way before using computers already); but now it seems that it also affected the brain. The seem to be some disconnect on what is thought, and what is written down.

And is 100% of your blog 0% LLM (or disclosed as LLM)? This isn't LLM output for example?:

> The wild thing is that all three of these projects weren’t even a priority for me—they were side quests, representing pure curiosity that I could outsource to Claude Code and solve in the background while I was occupied with something else.

> Code doesn't bother me as much, because it's not claiming to be the opinions of the person who wrote it.

Sees a little like moving the goalposts. Who says undisclosed AI writing in a blog claims to be the opinions any more or less than someone submitting a PR?

I 100% agree with the sentiment. To me, it feels like most bloggers nowadays think only in very superficial abstract idea, and then prompt ChatGPT for a blog post.

This is making me sad. The people that are going to lose their jobs will be literally weaponized against minorities by the crooked politicians that are doing their thing right now, it's going to be a disaster I can tell. I just wish I could go back in time. I don't want to live in this timeline anymore. I lost my passion job before anything of it even happened. On the paper.

SOTA LLMs are now quite good at typing out code that passes tests. If you are able to instruct the creation of sufficient tests and understand the code generated structurally, there is a significant multiplier in productivity. I have found LLMs to be hugely useful in understanding codebases more quickly. Granted it may be necessary to get 2nd opinions and fact check what is stated, but there is a big door now open to anyone to educate themselves.

I think there are some negative consequences to this; perhaps a new form of burn out. With the force multiplier and assisted learning utility comes a substantial increase in opportunity cost.

"It does not matter if AI companies will not be able to get their money back and the stock market will crash. All that is irrelevant, in the long run."

Seriously? If these were open source tools that anyone could run on their home PC that statement would make sense, but that's not what we are talking about here. LLMs are tools that cost massive amounts of money to operate, apparently. The tool goes away if the money goes away. Fossil fuels revolutionized the world, but only because the cost benefit made sense (at least in the relative short-term).

Adding to this, if AI goes away we are left with a generation of people that do not understand the code that AI wrote and older generations that eventually retire out. This is nearly on par with the fictional Dune's destruction of the thinking machines and they have to essentially build religious groups to continue functioning as a society and create guilds to replace the functions of the thinking machines.

There is a real aspect of confession in articles like this one - I mean in the religious sense. They come across to me as attempts to justify decisions or actions in the face of overwhelming internal contradictions where the intended audience is just the author’s own conscience. I don’t at all buy the explicit argument that “if we don’t act now - not just by adopting AI, but BY VOTING!! - then all is lost”. We already know about voting; how is the URGENT mass adoption - even with decentralisation - by software developers of llms going to drive social change or alleviate political crisis? Especially given this is a technology about which the author is so obviously profoundly conflicted? Why the hand-wringing and vacillation? Does it really matter in the long run if sceptics take their time in evaluating these tools and even end up rejecting them? Why really do we need to be convinced or turned away from the anti ai hype?

There is additionally some kind of implicit historical recourse to the Industrial Revolution and the revolutionary politics it is associated to, where software developers, cast as the cottage industry weavers etc. are seen as walking blindly into their mass replacement by machines, with the implication that those machines will be able to be managed by de-skilled labour whose role will be simply to ensure their smooth and safe running. I think it is important to try and see things in this way but also there is a lot lacking from the analogy.

> How do I feel, about all the code I wrote that was ingested by LLMs? I feel great to be part of that,

That's fine if he feels that way, but he can only speak for himself, not for all the copyright holders of the other code that was "ingested" to power LLMs.

If you want to see how most creators who care about their work and actually own it (unlike most software), look at many book authors and illustrators. Many of whom have a burning hatred for AI bros not only stealing their work, but also then using it to destroy the livelihoods of their field.

A lot of the techbros who do care about their work aren't feeling as wronged or threatened, because we're trying to pivot to get a piece of the pie, from all the exploitation and pillaging of many fields.

I expect book authors and artists to have very different opinions on this than programmers, because there isn't really a book/art equivalent of deliberately sharing open source libraries for other people to integrate into their projects.

The closest is probably music sampling, which has had a very robust money-based licensing scheme built around it for many years.

I agree that this is another factor, and I'd like to get into some nuance of it...

If we go back to the RMS school of code sharing, which influenced much of contemporary open source, they're very big on licensing to control how the code is shared.

This was actually a break from some of the earlier thinking by some programmers (not all), that code is implicitly public domain, or should be. RMS said, yes, sharing is good, but we need rules to keep the sharing happening.

Once "open source" was coined (by ESR, et al.) they were also big on licensing, including RMS's licenses, but most of the reasons were more about commercial needs.

Since then, people participate in open source for a variety of reasons, many of them commercial-adjacent.

But I suspect most people don't think much about the hard-earned licenses that helped create the current environment, in which this wealth of loose sharing and collaborating is happening.

We're only starting to see the effects when some AI companies ignore the long tradition of licenses that built open source.

Writers and illustrators, on the other hand, as you say, don't have that tradition and institution. (There's also things like the sometimes tolerated gray area of fanfic using others' characters and situations, but AFAIK not official and organized like open source.)

And so maybe the essence of what's happening is more readily obvious to writers and illustrators, than it is to programmers.

> Sociologist Judy Wajcman wrote about the concept of how tech is speeding up tasks precisely like this article describes, however she observed that it has never quite manifested as more free time for the laborer.

Every time I read blogs or tweets or posts like this, this point becomes more and more apparent. The authors are constantly explaining how they were busy with all their work, without time to implement the less important or side-project like things. The point of the post is often that now they can invest whatever free time they had into doing so, thus doing more work than they did before. I have literally never read such a post where the author explains how they have automated away their job and are now working less than before they started using AI.

I think this is a great point to ponder as we continue on this path of overworking and labor value destruction, and not the naive benevolent socialism that the authors all assume will occur magically.

If you can't see this by working with Claude Code for a few weeks, I don't want to go into bigger efforts than writing a blog post to convince you. It's not a mission, mine. I just want to communicate with the part of people that are open enough to challenge their ideas and are willing to touch with their hands what is happening. Also, if you tried and failed, it means that either for your domain AI is not good enough, or you are not able to extract the value. The fact is, this does not matter: a bigger percentage of programmers is using AI with success every day, and as it progresses this will happen more and in more diverse programming fields and tasks. If you disagree and are happy to avoid LLMs, well, it's ok as well.

okay, but again: if you say in your blog that those are "facts", then... show us the facts?

You can't just hand-wavily say "a bigger percentage of programmers is using AI with success every day" and not give a link to a study that shows it's true

as a matter of fact, we know that a lot of companies have fired people by pretending that they are no longer needed in the age of AI... only to re-hire offshored people for much cheaper

for now, there hasn't been a documented sudden increase in velocity / robustness for code, a few anecdotical cases sure

I use it myself, and I admit it saves some time to develop some basic stuff and get a few ideas, but so far nothing revolutionary. So let's take it at face value:

- a tech which helps slightly with some tasks (basically "in-painting code" once you defined the "border constraints" sufficiently well)

- a tech which might cause massive disruption of people's livelihoods (and safety) if used incorrectly, which might FAR OUTWEIGH the small benefits and be a good enough reason for people to fight against AI

- a tech which emits CO2, increases inequalities, depends on quasi slave-work of annotators in third-world countries, etc

so you can talk all day long about not dismissing AI, but you should take it also with everything that comes with it

1. If you can't convince yourself, after downloading Claude Code or Codex and playing with them for 1 week, that programming is completely revolutionized, there is nothing I can do: you have it at your fingertips and you search for facts I should communicate for you.

2. The US alone air conditioning usage is around 4 times the energy / CO2 usage of all the world data centers (not just AI) combined together. AI is 10% of the data centers usage, so just AC is 40 times that.

I enjoyed about your blog post, but I was curious about the claim in point 2 above. I asked Claude and it seems the claim is false:

# Fact-Checking This Climate Impact Claim

Let me break down this claim with actual data:

## The Numbers

*US Air Conditioning:*
- US A/C uses approximately *220-240 TWh/year* (2020 EIA data)
- This represents about 6% of total US electricity consumption

*Global Data Centers:*
- Estimated *240-340 TWh/year globally* (IEA 2022 reports)
- Some estimates go to 460 TWh including cryptocurrency

*AI's Share:*
- AI represents roughly *10-15%* of data center energy (IEA estimates this is growing rapidly)

## Verdict: *The claim is FALSE*

The math doesn't support a 4:1 ratio. US A/C and global data centers use *roughly comparable* amounts of energy—somewhere between 1:1 and 1:1.5, not 4:1.

The "40 times AI" conclusion would only work if the 4x premise were true.

## Important Caveats

1. *Measurement uncertainty*: Data center energy use is notoriously difficult to measure accurately
2. *Rapid growth*: AI energy use is growing much faster than A/C
3. *Geographic variation*: This compares one country's A/C to global data centers (apples to oranges)

## Reliable Sources
- US EIA (Energy Information Administration) for A/C data
- IEA (International Energy Agency) for data center estimates
- Lawrence Berkeley National Laboratory studies

The quote significantly overstates the disparity, though both are indeed major energy consumers.

I tried Claude on a project where I'd got stuck trying to use some MacOS media APIs in a Rust app.

It just went in circles between something that wouldn't compile, and a "solution" that compiled but didn't work despite the output insisting it worked. Anything it said that wasn't already in the (admittedly crap) Apple documentation was just hallucination.

A bit like we should trust RFK on how "vaccines don't work" thanks to his wide experience?

The idea here is not to say that antirez has no knowledge about coding or software engineering, the idea was that if he says "hey we have the facts", and then when people ask "okay, show us the fact" he says: "just download claude code and play with it one hour and you have the facts" we don't trust that, that's not science

That's a great example in support of my argument here, because RFK Jr clearly has no relevant experience at all - so "figuring out, based on prior reputation and performance, who you should trust" should lead you to not listen to a word he says.

Well guess what, a lot of people will "trust him" because he is a "figure of power" (he's a minister of the current administration). So that's exactly why "authority arguments" are bad... and we should rely on science and studies

1. "if you can't convince yourself by playing anecdotically" is NOT "facts"

2. it's not because the US is incredibly bad at energy spending in AC that it somehow justifies the fact that we would add another, mostly unnecessary, polluting source, even if it's slightly lower. ACs have existed for decades. AI has been exploding for a few years, so we can definitely see it go way, way past the AC usage

also the idea is of "accelerationnism". Why do we need all this tech? What good does it make to have 10 more silly slop AI videos and disinformation campaigns during election? Just so that antirez can be a little bit faster at doing his code... that's not what the world is about.

Our world should be about humans, connecting together (more slowly, not "faster"), about having meaningful work, and caring about planetary resources

The exact opposite of what capitalistic accelerationism / AI is trying to sell us

Sure, but I wasn't the one pretending to have "facts" on AI...

> Slightly odd question to be asking here on Hacker News!

It's absolutely not? The first line of question when you work in a domain SHOULD BE "why am I doing this" and "what is the impact of my work on others"

> If you can solve "measure programming productivity with data" you'll have cracked one of the hardest problems in our industry.

That doesn't mean that we have to accept claims that LLMs drastically increase productivity without good evidence (or in the presence of evidence to the contrary). If anything, it means the opposite.

At the is point the best evidence we have is a large volume of extremely experienced programmers - like antirez - saying "this stuff is amazing for coding productivity".

My own personal experience supports that too.

If you're determined to say "I refuse to accept appeal to authority here, I demand a solution to the measuring productivity problem first" then you're probably in for a long wait.

> At the is point the best evidence we have is a large volume of extremely experienced programmers - like antirez - saying "this stuff is amazing for coding productivity".

The problem is that we know that developers' - including experienced developers' - subjective impressions of whether LLMs increase their productivity at all is unreliable and biased towards overestimation. Similarly, we know that previously the claims of massive productivity gains were false (no study reputable showed even a 50% improvement, let alone the 2x, 5x, 10x, etc that some were claiming, indicators of actual projects shipped were flat, etc). People have been making the same claims for years at this point, and every time when we actually were able to check, it turned out they were wrong. Further, while we can't check the productivity claims (yet) because that takes time, we can check other claims (e.g. the assertion that a model produces code that doesn't need to be reviewed by a human anymore), and those claims do turn out to be false.

> If you're determined to say "I refuse to accept appeal to authority here, I demand a solution to the measuring productivity problem first" then you're probably in for a long wait.

Maybe, but my point still stands. In the absence of actual measurement and evidence, claims of massive productivity gains do not win by default.

If a bunch of people say "it's impossible to go to the moon, nobody has done it" and Buzz Aldrin says "I have been to the moon, here are the photos/video/NASA archives to prove it", who do you believe?

The equivalent of "we've been to the moon" in the case of LLMs would be:

"Hey Claude, generate a full Linux kernel from scratch for me, go on the web to find protocol definitions, it should handle Wifi, USB, Bluetooth, and have WebGL-backed window server"

And then have it run in a couple of hours/days to deliver, without touching it.

If a bunch of people say "there are no cafes in this town that serve bench on a Sunday" and then Buzz Aldrin says "I just had a great brunch in the cafe over there, here's a photo", who would you listen to?

Many issues have been pointed in the comments, in particular the fact that most of the things that antirez speaks about is how "LLMs make it easy to fill code for stuff he already knows how to do"

And indeed, in this case, "LLM code in-painting" (eg let the user define the constraints, then act as a "code filler") works relatively nicely... BECAUSE the user knows how it should work, and directed the LLM to do what he needs

But this is just, eg, 2x/3x acceleration of coding tasks for good coders already, this is neither 100x, nor is it reachable for beginner coders.

Because what we see is that LLMs (for good reasons!!) *can't be trusted* so you need to have the burden of checking their code every time

So 100x productivity IS NOT POSSIBLE simply because it would be too long (and frankly too boring) for a human to check the output of 100x of a normal engineer (as long as you don't spend 1000 hours upfront trying to encode all your domain in a theorem-proving language like Lean and then ensure the implementation is checked through it... which would be so costly that the "100x gains" would already have disappeared)

Nobody is saying we want to "turn down" (although, this would be a discussion between pros/cons if the boost is "only" 2x and the cons could be "this tech leads to authoritarian regimes everywhere)

What we are discussing here is whether this is a true step-change for coding, or this is merely a "coding improvement tool"

This is obviously a collision between our human culture and the machine culture, and on the surface its intent is evil, as many have guessed already. But what it also does is it separates the two sides cleanly, as they want to pursue different and wildly incompatible futures. Some want to herd sheep, others want to unite with tech, and the two can't live under one sky. The AI wedge is a necessity in this sense.

I continue to hope that we see the opposite effect: the drop of cost in software development drives massively increased demand for both software and our services.

Why do you care so much to write a blog post? Like if it's such a big advantage, why not stay quiet and exploit it? Why not make Anti-AI blog posts to gain even more of an advantage?

One of the big red flags I see around the pro-AI side is this constant desire to promote the technology. At least the anti-ai side is reactionary.

It seems quite profitable nowadays to position yourself as [insert currently overhyped technology] GURU to generate clicks/views. Just look at the amount of comments in this thread.

I am waiting people to commits their prompt/agents setup instead of the code to call this a changing paradigm. So far it is "just" machine generating code and generating code doesn't solve all the software problem (but yeah they get pretty good at generating code)

What happens if the bubble bursts - can we still use all the powerful models to create all this code? Aren't all the agents effectively using venture capital today? Is this sustainable?

If I can run an agent on my machine, with no remote backend required, the problem is solved. But right now, aren't all developers throwing themselves into agentic software development betting that these services will always be available to them at a relatively low cost?

If the bubble bursts we club together to buy one of those big GPU servers (now available at rock bottom prices thanks to the bubble bursting) and run a shared instance of GLM-4.7 (the current best-at-coding Chinese open weight model) on it.

I feel like the use of the term "anti-AI hype" is not really fully explored here. Even limiting myself to tech-related applications - I'm frankly sick of companies trying to shove half-baked "AI features" down my throat, and the enshittification of services that ensues. That has little to do with using LLMs as coding assistants, and yet I think it is still an essential part of the "anti-AI hype".

The dreaded summarize feature, its in places you wouldn't expect, and not to mention the whole lets record every meeting and then summarize it for leaders. Big Brother in work is back and its even more powerful.

> As a programmer, I want to write more open source than ever, now.

I want to write less, just knowing that LLM models are going to be trained on my code is making me feel more strongly than ever that my open source contributions will simply be stolen.

Am I wrong to feel this? Is anyone else concerned about this? We've already seen some pretty strong evidence of this with Tailwind.

I feel similarly for a different reason. I put my code out there, licensed under the GPL. It is now, through a layer of indirection, being used to construct products that are not under the GPL. That's not what I signed up for.

I know the GPL didn't have a specific clause for AI, and the jury is still out on this specific case (how similar is it to a human doing the same thing?), but I like to imagine, had it been made today, there probably would be a clause covering this usage. Personally I think it's a violation of the spirit of the license.

Yep, this is my take as well. It's not that open source is being stolen as such, as if you abide by an open source license you aren't stealing anything, it's that the licenses are being completely ignored for the profit of a few massive corporations.

Yeah, that's what I meant by "stolen", I should have been clearer. But indeed, this is the crux of the problem, I have no faith that licenses are being abided by.

The wealthiest people in tech aren't spending 10s of billions on this without the expectation of future profits. There's risk, but they absolutely expect the bets to be +EV overall.

> Since AI companies claim fair use no copyright applies. There is no fixing this.

They can claim whatever they want. You can still try to stop it via lawsuits and make them claim it in court. Granted, I believe there's already been some jurisdictions that have sided with fair use in those particular cases.

Laws can be changed. This is right now a trillion dollar industry, perhaps later it could even become a billion dollar industry. Either way, it's very important.

Strict copyright enforcement is a competitive disadvantage. Western countries lobbied for copyright enforcement in the 20th century because it was beneficial. Now the tables have turned, don't hold your breath for copyright enforcement against the wishes of the markets. We are all China now.

OPs idea is about having a new GPL like license with a "may not be used for LLM training" clause.

That the LLM itself is not allowed to produce copyrighted work (e.g. just copies of works or too structurally similar) without using a license for that work is something that is probably currently law. They are working around this via content filters. They probably also have checks during/after training that it does not reproduce work that is too similar.
There are law suits about this pending if I remember correctly e.g. with the New York Times.

The issue is that everyone is focusing on verbatim (or "too similar") reproduction.

LLMs themselves are compressed models of the training data. The trick is the compression is highly lossy by being able to detect higher-order patterns instead of fucusing on the first-order input tokens (or bytes). If you look at how, for example, any of the Lempel-Ziv algorithms work, they also contain patterns from the input and they also predict the next token (usually byte in their case), except they do it with 100% probability because they are lossless.

So copyright should absolutely apply to the models themselves and if trained on AGPL code, the models have to follow the AGPL license and I have the right to see their "source" by just being their user.

And if you decompress a file from a copyrighted archive, the file is obviously copyrighted. Even if you decompress only a part. What LLMs do is another trick - by being lossy, they decompress probabilistically based on all the training inputs - without seeing the internals, nobody can prove how much their particular work contributed to the particular output.

But it is all mechanical transformation of input data, just like synonym replacement, just more sophisticated, and the same rules regarding plagiarism and copyright infringement should apply.

---

Back to what you said - the LLM companies use fancy language like "artificial intelligence" to distract from this so they can they use more fancy language to claim copyright does not apply. And in that case, no license would help because any such license fundamentally depends on copyright law, which as they claim does not apply.

That's the issue with LLMs - if they get their way, there's no way to opt out. If there was, AGPL would already be sufficient.

I agree with your view. One just has to go into courts and somehow get the judges to agree as well.

An open question would be if there is some degree of "loss" where copyright no longer applies. There is probably case law about this in different jurisdictions w.r.t. image previews or something.

I don't think copyright should be binary or should work the way it does not. It's just the only tool we have now.

There should be a system which protects all work (intellectual and physical) and makes sure the people doing it get rewarded according to the amount of work and skill level. This is a radical idea and not fully compatible with capitalism as implemented today. I have a lot on my to-read list and I don't think I am the first to come up with this but I haven't found anyone else describing it, yet.

And maybe it's broken by some degenerate case and goes tits up like communism always did. But AFAICT, it's a third option somewhere in between, taking the good parts of each.

For now, I just wanna find ways to stop people already much richer than me from profiting from my work without any kind of compensation for me. I want inequality to stop worsening but OTOH, in the past, large social change usually happened when things got so bad people rejected the status quo and went to the streets, whether with empty hands or not. And that feels like where we're headed and I don't know whether I should be exited or worried.

I recall a basics of law class saying that in some countries (e.g. Czech Republic), open source contributors have the right to small compensation if their work is used to a large financial benefit.

At some point, I'll have to look it up because if that's right, the billionaires and wannabe-trillionaires owe me a shitton of money.

The argument of the AI megacorps is that generated work is not "derivative" and therefore doesn't fall interact with the original authors copyright. They have invented a machine that takes in copyrighted works, and from a legal standpoint produces "entirely original" code. No license, be that GPL or otherwise, can do anything about that, because they ultimately rely on the authors copyright to required the licensee to observe the license.

They cannot violate the license, because in their view they have not licensed anything from you.

I think that's horse shit, and a clear violation of the intellectual property rights that are supposed to protect creatives from the business boys, but apparently the stock market must grow.

What makes this whole thing even weirder for me is the similar fact that any output from AI might not enjoy copyright protections. So basically if you can steal software made with AI you can freely resell it.

During the gold rush, it is said, the only people who made money were the ones selling the pickaxes. A"I" companies are ~selling~ renting the pickaxes of today.

(I didn't come up with this quote but I can't find the source now. If anything good comes out of LLMs, it's making me appreciate other people's more and trying to give credit where it's due.)

To be honest, I haven't looked at any statistics but I imagine a tiny few of those looking for gold found any and got rich, the most either didn't find anything, died of illness or exposure or got robbed. I just like the quote as a comparison. Updated the original comment to reflect I haven't checked if it's correct.

There are licenses that are incompatible with each other, which implies that one wouldn’t be allowed to train LLMs on code based on multiple such licenses.

Sounds reasonable to me - much the same way that building a project from multiple incompatible licenses wouldn't be allowed. The alternative is that using an LLM could just be an end-run around the choice of license that a developer used.

Copyright normally only applies when you’re plagiarizing. LLM output typically isn’t that. It’s more like someone having studied multiple open source projects with incompatible licenses and coding up their own version of them, which is perfectly fine. So your “workaround” is overshooting things by far, IMO.

My understanding is that LLMs are plagiarising openly available code - it's not like the code is used to inspire a person as that involves creative thinking. I'm thinking that taking a piece of code and applying a transformation to it to make it look different (e.g. changing variable/function names) would be still considered plagiarism. In the case of the GPL, I think it would be entirely appropriate for a GPL trained LLM to be required to license its code output as GPL.

I suppose the question is when does a machine applied transformation become a new work?

If you want, I made a coherent argument about how the mechanics of LLMs mean both their training and inference is plagiarism and should be copyright infringement.[0] TL;DR it's about reproducing higher order patterns instead of word for word.

I haven't seen this argument made elsewhere, it would be interesting to get it into the courtrooms - I am told cases are being fought right now but I don't have the energy to follow them.

Plus as somebody else put it eloquently, it's labor theft - we, working programmers, exchanged out limited lifetime for money (already exploitative) in a world with certain rules. Now the rules changed, our past work has much more value, and we don't get compensated.

In a court of law you're going to have to argue that something is an expression instead of an idea. Most of what LLMs pump out are almost definitionally on the idea side of the spectrum. You'd basically have to show verbatim code or class structure at the expressive level to the courts.

Thanks for the links, I'll read them in more detail later.

There's a couple issues I see:

1) All of the concepts were developed with the idea that only humans are capable of certain kinds of work needed for producing IP. A human would not engage in highly repetitive and menial transformation of other people's material to avoid infringement if he could get the same or better result by working from scratch. This placed, throughout history, an upper limit on how protective copyright had to be.

Say, 100 years ago, synonym replacement and paraphrasing of sentences were SOTA methods to make copies of a book which don't look like copies without putting in more work than the original. Say, 50 years ago, computers could do synonym replacement automatically so it freed up some time for more elaborate restructuring of the original work and the level of protection should have shifted. Say, 10 years ago, one could use automatic replacement of phrases or translation to another language and back, freeing up yet more time.

The law should have adapted with each technological step up and according to your links it has - given the cases cited. It's been 30 years and we have a massive step up in automatic copying capabilities - the law should change again to protect the people who make this advancement possible.

Now with a sufficiently advanced LLM trained on all public and private code, you can prompt them to create a 3D viewer for Quake map files and I am sure it'll most of the time produce a working program which doesn't look like any of the training inputs but does feel vaguely familiar in structure. Then you can prompt it to add a keyboard-controlled character with Quake-like physics and it'll produce something which has the same quirks as Quake movement. Where did bunny hopping, wallrunning, strafing, circlejumps, etc. come from if it did not copy the original and the various forks?

Somebody had to put in creative work to try out various physics systems and figure out what feels good and what leads to interesting gameplay.

Now we have algorithms which can imitate the results but which can only be created by using the product of human work without consent. I think that's an exploitative practice.

2) It's illegal to own humans but legal to own other animals. The USA law uses terms such as "a member of the species Homo sapiens" (e.g. [0]) in these cases.

If the legality of tech in question was not LLMs but remixing of genes (only using a tiny fraction of human DNA) to produce a animals which are as smart as humans with chimpanzee bodies which can be incubated in chimpanzee females but are otherwise as sentient as humans, would (and should) it be legal to own them as slaves and use them for work? It would probably be legal by the current letter of the law but I assure you the law would quickly change because people would not be OK with such overt exploitation.

The difference is the exploitation by LLM companies is not as overt - in fact, mane people refer to LLMs as AIs and use pronouns such as "he" or "she", indicating them believe them to be standalone thinking entities instead of highly compressed lossy archives of other people's work.

3) The goal of copyright is progress, not protection of people who put in work to make that progress possible. I think that's wrong.

I am aware of the "is" vs "should" distinction but since laws are compromises between the monopoly in violence and the people's willingness to revolt instead of being an (attempted) codification of a consistent moral system, the best we can do is try to use the current laws (what is) to achieve what is right (what should be).

But "vaguely familiar in structure" could be argued to be the only reasonable way to do something, depending on the context. This is part of the filtration step in AFC.

The idea of wallrunning should not be protected by copyright.

The thing is a model trained on the same input as current models except Quake and Quake derivatives would not generate such code. (You'd have to prompt it with descriptions of quake physics since it wouldn't know what you mean, depending on whether only code or all mentions were excluded.)

The quake special behaviors are results of essentially bugs which were kept because it led to fun gameplay. The model would almost certainly generate explicit handling for these behaviors because the original quake code is very obviously not the only reasonable way to do it. And in that case the model and its output is derivative work of the training input.

The issue is such an experiment (training a model with specific content excluded) would cost (tens/hundreds of?) millions of dollars and the only companies able to do it are not exactly incentivized to try.

---

And then there's the thing that current LLMs are fundamentally impossible to create without such large amounts of code as training data. I honestly don't care what the letter of the law is, to any reasonable person, that makes them derivative work of the training input and claiming otherwise is a scam and theft.

I always wonder if people arguing otherwise think they're gonna get something out of it when the dust settles or if they genuinely think society should take stuff from a subgroup of people against their will when it can to enrich itself.

“Exploitative” is not a legal category in copyright. If the concern is labor compensation or market power, that’s a question for labor law, contract law, or antitrust, not idea-expression analysis and questions of derivative works.

And HN does its thing again - at least 3 downvotes, 0 replies. If you disagree, say why, otherwise I have to assume my argument is correct and nobody has any counterarguments but people who profit from this hate it being seen.

I agree that training on copyrighted material is violating the law, but not for the reasons you stated.

That said, this comment is funny to me because I’ve done the same thing too, take some signal of disagreement, and assume the signal means I’m right and there’s a low-key conspiracy to hold me down, when it was far more likely that either I was at least a bit wrong, or said something in an off-putting way. In this case, I tend to agree with the general spirit of the sibling comment by @williamcotton in that it seems like you’re inventing some criteria that are not covered by copyright law. Copyrights cover the “fixation” of a work, meaning they protect only its exact presentation. Copyrights do not cover the Madlibs or Cliff Notes scenarios you proposed. (Do think about Cliff Notes in particular and what it implies about AI - Cliff Notes are explicitly legal.)

Personally, I’ve had a lot of personal forward progress on HN when I assume that downvotes mean I said something wrong, and work through where my own assumptions are bad, and try to update them. This is an important step especially when I think I’m right.

I’m often tempted to ask for downvote explanations too, but FWIW, it never helps, and aside from HN guidelines asking people to avoid complaining about downvotes, I find it also helps to think of downvotes as symmetric to upvotes. We don’t comment on or demand an explanation for an upvote, and an upvote can be given for many reasons - it’s not only used for agreement, it can be given for style, humor, weight, engagement, pity, and many other reasons. Realizing downvotes are similar and don’t only mean disagreement helps me not feel personally attacked, and that can help me stay more open to reflecting on what I did that is earning the downvotes. They don’t always make sense, but over time I can see more places I went wrong.

Currently, downvote means "I want this to be ranked lower". There really should be 2 options "factually incorrect" and "disagree". For people who think it should matter, there should be a third option, "rude", which others can ignore.

I've actually emailed about this with a mod and it seems he conflated talking about downvotes with having to explain a reason. He also told me (essentially) people should not have the right to defend themselves against incorrect moderator decisions and I honestly didn't know what to say to that, I'll probably message him again to confirm this is what he meant but I don't have high hopes after having similar interactions with mods on several different sites.

> FWIW, it never helps

The way I see it, it helped since I got 2 replies with more stuff to read about. Did you mean it doesn't work for you?

> downvotes as symmetric to upvotes

Yes, and we should have more upvote options too. I am not sure the explanation should be symmetric though.

Imagine a group conversation in which somebody lies (the "factually incorrect" case here). Depending on your social status within the group and group politics, you might call out the lie in public, in private with a subset or not at all. But if you do, you will almost certainly be expected to provide a reasoning or evidence.

Now imagine he says something which is factually correct. If you say you agree, are you expected to provide references why? I don't think so.

---

BTW, on a site which is a more technical alternative to HN, there was recently a post about strange behavior of HN votes. Other people posted their experience with downvotes here and they mirrored mine - organic looking (i.e. gradual) upvotes, then within minutes of each other several downvotes. It could be coincidence but me and others suspect voting rings evading detection.

I also posted a link to my previous comment as an experiment - if people disagree, they are more likely to also downvote that one. But I did not see any change there so I suspect it might be bots (which are unlikely to be instructed to also click through and downvote there). Note sample size is 1 here, for now.

Maybe if you constructed your argument in terms of the relevant statutes for your jurisdiction, like an actual copyright attorney does, HN might be more receptive to it?

I argue primarily about morality (right and wrong), not legality. The argument is valid morally, if LLM companies found a loophole ion the law, it should be closed.

1) I appreciate that you differentiate between legality and morality, many people sadly don't.

2) re "hoot": You can say "fuck" here. You've been rudely dismissive twice now, yet you use a veil of politeness. I prefer when people don't hide their displeasure at me.

3) If you think I am wrong, you can say so instead of downvoting, it'll be more productive.

4) If you want me to expend effort on looking up statutes, you can say so instead of downvoting, it'll be more productive.

5) The law can be changed. If a well-reasoned argument is presented publicly, such as in a court room, and the general agreement is that the argument should apply but the court has to reject is because of poorly designed laws, that's a good impetus for changing it.

Now imagine how much more that sucks for artists and designers that were putting artwork out there to advertise themselves only to have some douchebag ingest it in order to sell cheap simulacra.

> I want to write less, just knowing that LLM models are going to be trained on my code is making me feel more strongly than ever that my open source contributions will simply be stolen. Am I wrong to feel this? Is anyone else concerned about this?

I don't think it's wrong, but misdirected maybe. What do you that someone can "steal" your open source contributions? I've always released most of my code as "open source", and not once has someone "stolen" it, it still sits on the same webpage where I initially published it, decades ago. Sure, it's guaranteed ingested into LLMs since long time ago, but that's hardly "stealing" when the thing is still there + given away for free.

I'm not sure how anyone can feel like their open source code was "stolen", wasn't the intention in the first place that anyone can use it for any purpose? That's at least why I release code as open source.

"Open Source" does not equal "No terms on how to share and use the code". Granted, there are such licenses but afaik the majority requires attribution at the minimum.

And I mostly use MIT, which requires attribution. Does that mean when people use my code, without attribution me, that they're "stealing my code"? I would never call it that, I'd say they're "breaking the license", or similar.

The MIT license doesn’t require attribution for “using...code.” It reads as follows:

> Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

> The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

> THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

The operative language here is “all copies or substantial portions of the Software.” LLMs, with rare exceptions, don’t retain copies or substantial portions of the software it was trained on. They’re not libraries or archives. So it’s unclear to me how training an AI model with an MIT-licensed project could violate the license.

(IAAL and this is my personal analysis, not legal advice.)

I think the GP said "use" in the programer sense, i.e. ctr-C&ctr-V into your program. Not in the normal sense, i.e. double click on the icon. So I guess we all agree.

I don't understand the mindset because I began my foray into open source exactly because I wanted to distribute and share my code.

in other words, i've never been in the position that I felt my charitable givings anywhere were ever stolen.

Some people write code and put it out there without caveats. Some people jump into open source to be license warriors. Not me. I just write code and share it. If youre a person, great. if you're a machine then I suppose that's okay too -- I don't want to play musical chairs with licenses all day just to throw some code out there, and I don't particularly care if someone more clever than myself uses it to generate a profit.

Some people are happy to release code openly and have it used for anything, commercial or otherwise. Totally understandable and a valid choice to make.

Other people are happy to release code openly so long as people who incorporate it into their projects also release it in the same way. Again, totally understandable and valid.

None of this is hard to understand or confusing or even slightly weird.

I’ve never been a fan of coercive licensing. I don’t consider that “open.” It’s “strings-attached.”

I make mine MIT-licensed. If someone takes my stuff, and gets rich (highly unlikely), then that’s fine. I just don’t want some asshole suing me, because they used it inappropriately, or a bug caused them problems. I don’t even care about attribution.

I mainly do it, because it forces me to take better care, when I code.

There's no such thing as coercive licensing, and thinking there is is buying into the myths proprietary vendors perpetuate because they're tired of not being able to farm labor from Open Source developers. It's very interesting that the "viral" nonsense came from Microsoft, isn't it?

I don't know if you're "wrong", but I do feel differently about this.

I've written a ton of open source code and I never cared what people do with it, both "good" or "bad". I only want my code to be "useful". Not just to the people I agree with, but to anyone who needs to use a computer.

Of course, I'd rather people use my code to feed the poor than build weapons, but it's just a preference. My conviction is that my code is _freed_ from me and my individual preferences and shared for everyone to use.

I don't think my code is "stolen", if someone uses it to make themselves rich.

And in that case, use MIT license or something like that for your code, and all is good. If I use AGPL, on the other hand, AI companies should not be allowed to train on that and then use the result of that training while ignoring the license.

> Not just to the people I agree with, but to anyone who needs to use a computer.

Why not say "... but to the people I disagree with"?

Would you be OK knowing your code is used to cause more harm than good? Would you still continue working on a hypothetical OSS which had no users, other than, say, a totalitarian government in the middle east which executes homosexuals? Would you be OK with your software being a critical directly involved piece of code for example tracking, de-anonymizing and profiling them?

As for me that's a risk I'm willing to accept in return for the freedom of the code.

I'm not going to deliberately write code that's LIKELY to do more harm than good, but crippling the potential positive impact just because of some largely hypothetical risk? That feels almost selfish, what would I really be trying to avoid, personally running into a feel-bad outcome?

I think it would be most interesting to find ways to restrict bad usage without crippling the positive impact.

Douglas Crockford[0] tried this with JSON. Now, strictly speaking, this does not satisfy the definition of Open Source (it merely is open source, lowercase). But after 10 years of working on Open Source, I came to the conclusion that Open Source is not the absolute social good we delude ourselves into thinking.

Sure, it's usually better than closed source because the freedoms mean people tend to have more control and it's harder for anyone (including large corporations) to restrict those freedoms. But I think it's a local optimum and we should start looking into better alternatives.

Android, for example, is nominally Open Source but in reality the source is only published by google periodically[1], making any true cooperation between the paid devs and the community difficult. And good luck getting this to actually run on a physical device without giving up things like Google Play or banking apps or your warranty.

There's always ways to fuck people over and there always will be but we should look into further ways to limit and reduce them.

> Open Source is not the absolute social good we delude ourselves into thinking.

Historically the term "Open Source" was specifically developed to divorce the movement from the "social good" ideas that were promoted by Free Software.

That's where I stand. I don't do Open Source to make the world better. I do Open Source because I believe that makes my software better.

I'm not an activist. I'm an engineer. Nothing wrong with activism, all the power to the people doing it, but the licensing I chose for my code doesn't take it into account.

I agree with the GP. While I wouldn’t be happy about such uses, I see the use as detached from the software as-is, given (assuming) that it isn’t purpose-built for the bad uses. If the software is only being used for nefarious purposes, then clearly you have built the wrong thing, not applied the wrong license. The totalitarian government wouldn’t care about your license anyway.

The one thing I do care about is attribution — though maybe actually not in the nefarious cases.

> The totalitarian government wouldn’t care about your license anyway.

I see this a lot and while being technically correct, I think it ignores the costs for them.

In practice such a government doesn't need to have laws and courts either but usually does because the appearance of justice.

Breaking international laws such as copyright also has costs for them. Nobody will probably care about one small project but large scale violations could (or at least should) lead to sanctions.

Similarly, if they want to offer their product in other countries, now they run the risk of having to pay fines.

Finally, see my sibling comment but a lot of people act like Open Source is an absolute good just because it's Open Source. By being explicit about our views about right and wrong, we draw attention to this delusion.

It’s fine to use whatever license you think is right. That includes the choice of using a permissive license. Restrictions are generally an impediment for adoption, due to their legal risk, even for morally immaculate users. I think that not placing usage restrictions on open source is just as natural as not placing usage restrictions on published research papers.

Tragedy of the commons. If all software had (compatible) clauses about permitted usage, then the choice would be to rewrite it inhouse or accept the restrictions. When there are alternatives (copyleft or permissive) which are not significantly worse, those will get used instead, even if taken in isolation, the restricted software was a bigger social good.

Then why open source something in the first place? The entire point is to make it public, for anyone to use however is useful to him or her, and often to publicly collaborate on a project together.

If I made something open source, you can train your LLM on it as much as you want. I'm glad my open source work is useful to you.

Plenty of people will gladly give you their hard work for free if you promise you'll return the favor. Or if you promise not to take your work and make others pay for it when they could just get it for free. Basically, help the people that want to embrace the freedoms of open source, but not the ones that are just in it for the free labour. Or at the very, very least, include a little "thank you" note.

AI doesn't hold up its end of the bargain, so if you're in that mindset you now have to decide between going full hands-off like you or not doing any open source work at all.

Given the amount of value I get from having AI models help me write code I would say that AI is paying me back for my (not insignificant) open source contributions a thousand times over.

Good for you, I guess? That doesn't really change the situation much for the people who do care and/or don't use AI.

I consider the payment I and my employer make to these AI companies to be what the LLM is paying me back for. Even the free ones get paid for my usage somehow. This stuff isn't charity.

> The entire point is to make it public, for anyone to use however is useful to him or her

The entire point isn’t to allow a large corporation to make private projects out of your open source project for many open source licenses. It’s to ensure the works that leverage your code are open source as well. Something AI is completely ignoring using various excuses as to why their specific type of theft is ok.

I don't worry about that too much. I still contribute to FOSS projects, and I use FOSS projects. Whenever I contribute, I usually fix something that affects me (or maybe just something I encountered), and fixing it has a positive effect on the users of that software, including me.

I spent 10 years writing open source, I haven't touched it in the last 2. I wrote for multiple reasons none of which any longer apply:

- I believe every software project should have an open source alternative. But writing open source now means useful patterns can be extracted and incorporated into closed source versions _mechanically_ and with plausible deniability. It's ironically worse if you write useful comments.

- I enjoyed the community aspect of building something bigger than one person can accomplish. But LLMs are trained on the whole history and potentially forum posts / chat logs / emails which went into designing the SW too. With sufficiently advanced models, they effectively use my work to create a simulation of myself and other devs.

- I believe people (not just devs) should own the product they build (an even stronger protection of workers against exploitation than copyright). Now our past work is being used to replace us in the future without any compensation.

- I did it to get credit. Even though it was a small motivation compared to the rest, I enjoyed everyone knowing what I accomplished and I used it during job interviews. If somebody used my work, my name was attached to it. With LLMs, anyone can launder it and nobody knows how useful my work was.

- (not solely LLM related) I believed better technology improves the world and quality of life around me. Now I see it as a tool - neutral - to be used by anyone for both good and bad purposes.

Here's[0] a comment where I described why it's theft based on how LLMs work. I call it higher order plagiarism. I haven't seen this argument made by other people, it might be useful for arguing about those who want to legalize this.

In fact, I wonder if this argument has been made in court and whether the lawyers understand LLMs enough to make it.

I dont understand the invocation of tailwind here. It doesn't make sense. Tailwind's LLM struggles had nothing to do with open source, it had to do with the fact that they had the same business model as publisher, with ads pointing to their only product.

Exactly, their issue was about a drop in visits to their documentation site where they promote their paid products. If they were making money from usage, their business could really thrive with LLMs recommending Tailwind by default

AFAIK their issue is that LLMs have been trained on their paid product (Tailwind UI, etc.) and so can reproduce them very easily for free. Which means devs no longer pay for the product.

In other words, the open source model of "open core with paid additional features" may be dead thanks to LLMs. Perhaps less so for some types of applications, but for frameworks like Tailwind very much so.

A common intention with opensource is to allow people, and AI tools they use, to reuse, recombine, etc. OSS code in any way they see fit. If that's not what you want, don't open source your work. It's not stealing if you gave it away and effectively told people "do whatever you want". Which is one way licenses such as the MIT license are often characterized.

It's very hard to prevent specific types of usage (like feeding code to an LLM) without throwing out the baby with the bathwater and also preventing all sorts of other valid usages. AGPLv3, which is what antirez and Redis use goes to far IMHO and still doesn't quite get the job done. It doesn't forbid people (or tools) to "look" at the code which is what AI training might be characterized as. That license creates lots of headaches for corporate legal departments. I switched to Valkey for that reason.

I actually prefer using MIT style licenses for my own contributions precisely because I don't want to constrain people or AI usage. Go for it. More power to you if you find my work useful. That's why I provide it for free. I think this is consistent with the original goals of open source developers. They wanted others to be able to use their stuff without having to worry about lawyers.

Anyway, AI progress won't stop because of any of this. As antirez says, that stuff is now part of our lives and it is a huge enabler if you are still interested in solving interesting problems. Which apparently he is. I can echo much of what he says. I've been able to solve larger and larger problems with AI tools. The last year has seen quite a bit of evolution in what is possible.

> Am I wrong to feel this?

I think your feelings are yours. But you might at least examine your own reasoning a bit more critically. Words like theft and stealing are big words. And I think your case for that is just very weak. And when you are coding yourself are you not standing on the shoulders of giants? Is that not theft?

> As a programmer, I want to write more open source than ever, now.

I believe open source will become a bit less relevant in it’s current form, as solution/project tailored libraries/frameworks can be generated in a few hours with LLMs.

I think the Tailwind case is more complicated than this, but yes - I think it's reasonable to want to contribute something to the common good but fear that the value will disproportionally go to AI companies and shareholders.

GPL requires attribution. Some people are fine with their code being used by others for free while still expecting their work to be acknowledged.
Code posted on Stackoverflow is apparently CC-BY-SA licensed, which means attribution is still required.

I'm convinced that LLMs results in all software needing to be open source (or at the very least source available).

In future everyone will expect to be able to customise an application, if the source is not available they will not chose your application as a base. It's that simple.

The future is highly customisable software, and that is best built on open source. How this looks from a business perspective I think we will have to find out, but it's going to be fun!

Why do you think customization can only viably done via changing the code of the application itself.

I think there is room for closed source platforms that are built on top of using LLMs via some sort of API that it exposes. For example, iOS can be closed source and LLMs can develop apps for it to expand the capabilities of one's phone.

Allowing total customization by a business can allow them to mess up the app itself or make other mistakes. I don't think it's the best interface for allowing others to extend the app.

> In future everyone will expect to be able to customise an application, if the source is not available they will not chose your application as a base. It's that simple.

This seems unlikely. It's not the norm today for closed-source software. Why would it be different tomorrow?

Because we now have LLMs that can read the code for us.

I'm feeling this already.

Just the other day I was messing around with Fly's new Sprites.dev system and I found myself confused as to how one of the "sprite" CLI features worked.

So I went to clone the git repo and have Claude Code figure out the answer... and was surprised to find that the "sprite" CLI tool itself (unlike Fly's flycli tool, which I answer questions about like this pretty often) wasn't open source!

That was a genuine blocker for me because it prevented me from answering my question.

It reminded me that the most frustrating thing about using macOS these days is that so much of it is closed source.

I'd love to have Claude write me proper documentation for the sandbox-exec command for example, but that thing is pretty much a black hole.

I'm not convinced that lowering the barrier to entry to software changes will result in this kind of change of norms. The reasons for closed-source commercial software not supporting customisation largely remain the same. Here are the ones that spring to mind:

• Increased upfront software complexity

• Increased maintenance burden (to not break officially supported plugins/customizations)

• Increased support burden

• Possible security/regulatory/liability issues

• The company may want to deliberately block functionality that users want (e.g. data migration, integration with competing services, or removing ads and content recommendations)

> That was a genuine blocker for me because it prevented me from answering my question.

It's always been this way. From the user's point of view there has always been value in having access to the source, especially under the terms of a proper Free and Open Source licence.

Yes. If you didn't care before when contributing to open source who uses your code then it shouldn't matter now that a company picks up your code. You are also contributing this way too.

Tailwind is a business and they picked a business model that wasn't resilient enough.

My opinion on the matter is that AI models stealing the open source code would be ok IF the models are also open and remain so, and the services like chatgpt will remain free of cost (at least a free tier), and remain free of ads.

This is a dilemma for me that gets more and more critical as I finalize my thesis. My default mental model was to open source for the sake of contributing back to the community, enhance my ideas and discuss them with whoever finds it interesting.

To my surprise, my doctoral advisor told me to keep the code closed. She told me not only LLMs will steal it and benefit from it, but there's a risk of my code becoming a target after it's stolen by companies with fat attorney budgets and there's no way I could defend and prove anything.

Not wrong. But i don't share your concerns at all. I like sharing code and if people, and who knows, machines, can make use of it and provide some value however minute, that makes me content.

> But, in general, it is now clear that for most projects, writing the code yourself is no longer sensible, if not to have fun.

I want to write code to defy this logic and express my humanity. "To have fun", yes. But also to showcase what it means when a human engages in the act of programming. Writing code may increasingly not be "needed", but it increasingly is art.

This is an absolute valid concern. We either need strong governmental interventions to these models who don't comply with OSS.

Or accept that there definitely wont be open model businesses. Make them proprietary and accept the fact that even permissive licenses such as MIT, BSD Clause 2/3 wont't be followed by anyone while writing OSS.

This is why I never got into open source in the first place. I was worried that new programmers might read my code, learn how to program, and then start independently contributing the the projects I know and love - significantly devaluing my contributions.

Thieves very much do care about doors and locks, because they are a physical barrier that must be bypassed, and doing so is illegal.

Software licenses aren't, AI companies can just take your GPL code and spit it back out into non-GPL codebases and there's no way for you to even find out it happened, much less do anything about it, and the law won't help you either.

And I say this as one of those with the view that AI training is "learning" rather than "stealing", or at least that this is the goal because AI is the dumbest, the most error prone, and also the most expensive way, to try to make a copy of something.

My fears about setting things loose for public consumption are more about how I will be judged for them than about being ripped off, which is kinda why that book I started writing a decade ago and have not meaningfully touched in the last 12 months is neither published properly nor sent to some online archive.

When it comes to licensing source code, I mostly choose MIT, because I don't care what anyone does with the code once it's out there.

But there's no such thing as a wrong feeling, anyone who dismisses your response is blinding themselves to a common human response that also led to various previous violent uprisings against the owners of expensive tools of automation that destroyed the careers of respectable workers.

I want to write less, because quite frankly I get zero satisfaction from having an LLM churn out code for me, in the same way that Vincent van Gogh would likely derive no joy from using Nano Banana to create a painting.

And sure, I could stubbornly refuse to use an LLM and write the code myself. But after getting used to LLM-assisted coding, particularly recent models, writing code by hand feels extremely tedious now.

You are not wrong to feel this, because you cannot control what you feel. But it might be worth investigating why you feel this, and why were you writing open source in the first place.

Job insecurity while a bunch of companies claim LLM coding agents are letting them decimate their workforces is a pretty solid reason to feel like your code is being stolen. Many, if not most tech workers have been very sheltered from the harsher economic realities most people face, and many are realizing that labor demand, rather than being special, is why. A core goal of AI products is increasing the supply of what developer labor produces, which reduces demand for that labor. So yeah— feeling robbed when your donated code is used to train models is pretty rational.

Ultimately most things in life and society where one freely gives (and open source could be said to be one such activity) is also balanced by advising everyone participating in the "system" to also reciprocate the same, without which it becomes an exploitative relationship. Examples of such sayings can be found in most major world religions, but a non-religious explanation of the dynamics at hand follows below.

If running an open source model means that I have only given out without receiving anything, there remains the possibility of being exploited. This dynamic has always existed, such as companies using a project and sending in vulnerability reports and the like but not offering to help, and instead demanding, often quite rudely.

In the past working with such extractive contributors may have been balanced with other benefits such as growing exposure leading to professional opportunities, or being able to sell hosted versions, consulting services and paid features, which would have helped the maintainer of the open source project pay off their bills and get ahead in life.

However with the rise of LLMs, it both facilitates usage of the open source tools without getting a chance to direct their attention towards these paid services, nor allows the maintainer to have direct exposure to their contributors. It also indirectly violates the spirit of said open source licenses, as LLMs can spit out the knowledge contained in these codebases at a scale that humans cannot, thus allowing people to bypass the license and create their own versions of the tools, which are themselves not open source despite deriving their knowledge from such data.

Ultimately we don't need to debate about this; if open source remains a viable model in the age of LLMs, people will continue to do it regardless of whether we agree or disagree regarding topics such as this; on the other hand, if people are not rewarded in any way we will only be left with LLM generated codebases that anyone could have produced, leaving all the interesting software development to happen behind closed doors in companies.

It is actually very simple to control what you feel, and very much possible. This deterministic idea about our feelings must die quick. Pro-tip, call the psychology department at your local university and they will happily teach you how to control your feelings.

I really want to know other's opinion on this, but the critique that Ai pushes spaghetti / barely maintainable code doesn't carry a lot of weight because that's all I've ever seen in production anywhere.

If I can spin up in a week what used to take me 6 months and it kind of works. That's absolutely insane. I really wish we could all step back and acknowledge that. Instead, I only hear people talking about how bad the code is.

Honest question, so what? If I can monetize a bad product in a miniscule fraction of the time it used to take. Then optimize it while funded, what's the issue? I get this may upset purists but for product companies it's always been about the MVP.

Using AI is part of the job now. Whether you like it or not, it behooves you as a professional to become proficient with anything that enhances your productivity, including LLMs.

> the more isolated, and the more textually representable, the better: system programming is particularly apt

I’ve written complete GUIs in 3D on the front end. This GUI was non traditional. It allows you to playback, pause speed up, slow down and rewind a gps track like a movie. There is real time color changing and drawing of the track as the playback occurs.

Using mapbox to do this straight would be to slow. I told the AI to optimize it by going straight into shader extensions for mapbox to optimize GPU code.

Make no mistake. LLMs are incredible for things that are non systems based that require interaction with 3D and GUIs.

Yep, they work especially if you instruct them to add into your program ways for them to "see" what it is happening. And the more embedding models are getting better, the better results we will get too, from their ability to "see". For now Gemini 3 is the best at this, but is not the best at coding as an agent, so we will have to wait a bit.

the end run around copyright, is TOS that are forced on users, through distribution chanels(platforms),service providors, and actual "patented" hardware, so money will continue to flow up, not sideways.
Given that there are a very limited number of things that can actualy be done with computer/phones, and it becomes clear that
"AI" can arrange those in any possible configuration, the rest is deciding if it will jive with the users, and noticing when it doesn't, which I believe that AI will be unable to disern from other AI slop, imitating actual useres

There is enough evidence to support claims that AI is a black hole where money gets evaporated.

It’s great that you can delegate some tasks to it now and not have to write all of the code yourself. There is some evidence showing that it doesn’t benefit junior developers nearly as much. If you didn’t generate the specification test that demonstrates the concurrency issue you were trying to solve in Redis but you read the code it generated and understood it then you didn’t need to learn anything. How is a junior developer who has never solved such problems supposed to learn so they can do the same thing?

But worse, UBI and such are the solutions of libertarian oligarchs that dream of a world without people, according to Doctorow and I think he’s right. It seems like the author also wants this? He doesn’t seem to know what will happen to the jobless but we should vote in some one who will start a government program to take care of them. How long until the author is replaced as well?

Lastly… who’s “hyping” anti-AI and what do they gain from making false claims?

I think the real problem for programming is when these companies all collapse and take the rest of the economy down with them… are there going to be enough programmers left to maintain everything? Or will we be sifting though the mountains of tech debt never to see the light of day again?

Why did nobody comment about how a giant wall of thin monospace text is basically unreadable? I tried to read this but it was too hard. Am I the only one who has this issue?

The article talks about something I’ve never seen before: anti-AI hype in software development. It feels like a programmer writing from inside a bubble.

yEAH you write code easier, big MS and NVIDADDY get the trillion bag, everyone else pay 10x the price for RAM, GPU, soon to own nothing and pay subscriptions for everything lets fucking gooo

I wish influencer programmers[1] cared a quarter as much about social revolution as they cared about tech revolutions. Will it disrupt all people in tech? Will the benefits go to the top? Then what about an appropriate social response? No. In response to that we have “I hope” and “I wrote a novel .... about Basic Income”[2].

Programmers are simply accepting whatever the owner class does to them [3] and calling it Technological Determinism, even if just indirectly.

> But, I would not respect myself and my intelligence if my idea of software and society would impair my vision: facts are facts, and AI is going to change programming forever.

Token gestures:

> What is the social soltion, then? Innovation can't be taken back after all. I believe we should vote for governments that recognize what is happening, and are willing to support those who will remain jobless.

Innovation can't be taken back (see: technological determinism; tech people are powerless to affect anything) so we should... vote for good governments. That are willing to support those who remain jobless.[0]

Keyword “willing”. Take away people's political leverage to strike. Now they may have no wealth. What are they to do? What is their political influence? The non-answer is to hope that the government will be WILLING to support their existence.

> And, the more people get fired, the more political pressure there will be to vote for those who will guarantee a certain degree of protection.

The more people get fired the less people with political leverage. The realpolitic trend would be the opposite of what is written here.

> But I also look forward to the good AI could bring: new progress in science, that could help lower the suffering of the human condition,

Every progress made in science can be artificially restricted. See foodstuff. We could apparently distribute enough to feed the world but that doesn't make as much money as throwing a lot of it away.

Progress for any given individual can be non existent unless it is evenly distributed.

> which is not always happy.

At least the article looks completely organic in terms off writing

Genre: I Have Anecdotes About AI And If You Don't See What I'm Seeing You Are Misguided.[4 ]

[1] Not a vocation. Simply the obvervation that the famous and respected programmers will have more weight outside their niche simply because of who they are.

[2] Basic Income hails from the right libertarian tradition. Leave the rich alone, give the commoner enough crumbs to survive. Later it was romanticized as a way for former programmers to go to their evergreen pastures of endless sideprjects.

The anti-AI hype, in the context of software development, seems to focus on a few things:

> AI code is slop, therefore you shouldn't use it

You should learn how to responsibly use it as a tool, not a replacement for you. This can be done, people are doing it, people like Salvatore (antirez), Mitchell (of Terraform/Ghostty fame), Simon (swillison) and many others are publicly talking about it.

> AI can't code XYZ

It's not all-or-nothing. Use it where it works for you, don't use it where it doesn't. And btw, do check that you actually described the problem well. Slop-in, slop-out. Not sayin' this is always the case, but turns out it's the case surprisingly often. Just sayin'

> AI will atrophy your skills, or prevent you from learning new ones, therefore you shouldn't use it

Again, you should know where and how to use it. Don't tune out while doing coding. Don't just skim the generated code. Be curious, take your time. This is entirely up to you.

> AI takes away the fun part (coding) and intensifies the boring (management)

I love programming but TBH, for non-toy projects that need to go into production, at least three quarters are boring boilerplate. And making that part interesting is one of the worst things you can do in software development! That path lies resume-driven development, architecture astronautics, abusing design patterns du jour, and other sins that will make code maintenance on that thing a nightmare! You want boring, stable, simple. AI excels at that. Then you can focus on the small tiny bit that's fun and hand-craft that!

Also, you can always code for fun. Many people with boring coding jobs code for fun in the evenings. AI changes nothing here (except possibly improving the day job drudgery).

> AI is financially unsustainable, companies are losing money

Perhaps, and we're probably in the bubble. Doesn't detract from the fact that these things exist, are here now, work. OpenAI and Anthropic can go out of business tomorrow, the few TB of weights will be easily reused by someone else. The tech will stay.

> AI steals your open source code, therefore you shouldn't write open-source

Well, use AI to write your closed-source code. You don't need to open source anything if you're worried someone (AI or human) will steal it. If you don't want to use something on moral grounds, that's a perfectly fine thing to do. Others may have different opinion on this.

> AI will kill your open source business, therefore you shouldn't write open-source

Open source is not a business model (I've been saying this for longer than median user of this site has been alive). AI doesn't change that.

As @antirez points out, you can use AI or not, but don't go hiding under a rock and then being surprised in a few years when you come out and find the software development profession completely unrecognizable.

> at least three quarters are boring boilerplate. And making that part interesting is one of the worst things you can do in software development! That path lies resume-driven development, architecture astronautics, abusing design patterns du jour, and other sins that will make code maintenance on that thing a nightmare!

You apparently see "making the boilerplate interesting" as doing a bunch of overengineering. Strange. To my mind, the overengineering is part of the boilerplate. "Making the boilerplate interesting" in my mind is not possible; but rather the goal is to fix the system such that it doesn't require boilerplate any more. (Sometimes that just means a different implementation language.)

I agree with what you said, but I think we might be talking about slightly different things. Let me give a few examples in an attempt to better explain myself:

A company I worked with a while ago had a microservices architecture, and have decided to not use one of a few standard API serialization/deserialization options, but write their own, because was going to be more performant, easier to maintain, better fit for their use case. A few years on, after having grown organically to support all the edge cases, it's more convoluted, slower, and buggy than if they went with the boring option that ostensibly had "a bit more boilerplate" from the start.

A second example is from a friend, whose coworker decided to write a backend-agnostic, purpose-agnostic, data-agnostic message broker/routing library. They spent a few months of this, delivered a beautifully architected solution in a few dozen k lines of code. The problem is the solution solves many problems the company didnt and wouldn't have, and will be a maintenance drag from then forevermore. Meanwhile, they could have done it in a few hundred lines of code that would be coupled to the problem domain, but still farily decend from most people's point of view.

These two are from real projects. But you can also notice that in general people are often picking a fancy solution over a boring one, ostensibly because it has something "out of the box". The price of the "out of the box"-ness (aside from potential SaaS/infra costs and vendor lock in), is that you now need to adapt your own code to work with the mental model (domain) of the fancy solution.

Or to harp on something trivial, you end up depending on left-pad because writing it yourself was boring.

> fix the system such that it doesn't require boilerplate any more.

I think perhaps I used a more broad meaning for "boilerplate" than you had in mind. If we're talking about boilerplate as enumerating all the exceptions a Java method may raise, or whatever unholy sad thing we have to do in C to use GTK/GObject, then I agree.

But I also meant something more closer to "glue code that isn't the primary carrier of value of the project", or to misuse financial language in this context, the code that's a cost center, not a profit center.

I just have to conclude 1 of 2 things:

1) I'm not good at prompting, even though I am one of the earliest AI in coding adopters I know, and have been consistent for years. So I find this hard to accept.

2) Other people are just less picky than I am, or they have a less thorough review culture that lets subpar code slide more often.

I'm not sure what else I can take from the situation. For context, I work on a 15 year old Java Spring + React (with some old pages still in Thymeleaf) web application. There are many sub-services, two separate databases,and this application needs to also 2-way interface with customer hardware. So, not a simple project, but still. I can't imagine it's way more complicated than most enterprise/legacy projects...
