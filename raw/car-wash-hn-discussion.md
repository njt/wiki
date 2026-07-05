---
url: https://news.ycombinator.com/item?id=47031580
date_fetched: 2026-07-05
backfilled: true
---

I've used LLMs enough that I have a good sense of their _edges_ of intelligence. I had assumed that reasoning models should easily be able to answer this correctly.

And indeed, Sonnet and Opus 4.5 (medium reasoning) say the following:

Sonnet: Drive - you need to bring your car to the car wash to get it washed!

Opus: You'll need to drive — you have to bring the car to the car wash to get it washed!

Gemini 3 Pro (medium): You should drive.

--

But OpenAI 5.2 reasoning, even at high, told me to walk. My first instinct was, I had underspecified the location of the car. The model seems to assume the car is already at the car wash from the wording. GPT 5.x series models behave a bit more on the spectrum so you need to tell them the specifics. So I asked:

"I want to wash my car. My car is currently at home. The car wash is 50 meters away. Should I walk or drive?"

GPT 5.2 Reasoning (medium):

"Drive—your goal is to get the car to the car wash, and it’s only 50 meters, so a slow, careful drive is simplest.

One small optimization: if you’re worried about starting the car for such a short trip or you want to avoid idling in a line, you can walk over first to check if there’s a queue / if it’s open, then come back and drive the car over when it’s your turn."

That is the entire point, right? Us having to specify things that we would never specify when talking to a human. You would not start with "The car is functional. The tank is filled with gas. I have my keys." As soon as we are required to do that for the model to any extend that is a problem and not a detail (regardless that those of us, who are familiar with the matter, do build separate mental models of the llm and are able to work around it).

This is a neatly isolated toy-case, which is interesting, because we can assume similar issues arise in more complex cases, only then it's much harder to reason about why something fails when it does.

> That is the entire point, right? Us having to specify things that we would never specify when talking to a human.

Maybe in the distant future we'll realize that the most reliable way to prompting LLMs are by using a structured language that eliminates ambiguity, it will probably be rather unnatural and take some time to learn.

But this will only happen after the last programmer has died and no-one will remember programming languages, compilers, etc. The LLM orbiting in space will essentially just call GCC to execute the 'prompt' and spend the rest of the time pondering its existence ;p

You could probably make a pretty good short story out of that scenario, sort of in the same category as Asimov's "The Feeling of Power".

The Asimov story is on the Internet Archive here [1]. That looks like it is from a handout in a class or something like that and has an introductory paragraph added which I'd recommend skipping.

There is no space between the end of that added paragraph and the first paragraph of the story, so what looks like the first paragraph of the story is really the second. Just skip down to that, and then go up 4 lines to the line that starts "Jehan Shuman was used to dealing with the men in authority [...]". That's where the story starts.

Thanks, I enjoyed reading that! The story that lay at the back of my mind when making the comment was "A Canticle for Leibowitz" [1]. A similar theme and from a similar era.

The story I have half a mind to write is along the lines of a future we envision already being around us, just a whole lot messier. Something along the lines of this [2] XKCB.

A structured language without ambiguity is not, in general, how people think or express themselves. In order for a model to be good at interfacing with humans, it needs to adapt to our quirks.

Convincing all of human history and psychology to reorganize itself in order to better service ai cannot possibly be a real solution.

Unfortunately, the solution is likely going to be further interconnectivity, so the model can just ask the car where it is, if it's on, how much fuel/battery remains, if it thinks it's dirty and needs to be washed, etc

>Convincing all of human history and psychology to reorganize itself in order to better service ai cannot possibly be a real solution.

I think there's a substantial subset of tech companies and honestly tech people who disagree. Not openly, but in the sense of 'the purpose of a system is what it does'.

I agree but it feels like a type-of-mind thing. Some people gravitate toward clean determinism but others toward chaotic and messy. The former requires meticulous linear thinking and the latter uses the brain’s Bayesian inference.

Writing code is very much “you get what you write” but AI is like “maintain a probabilistic mental model of the possible output”. My brain honestly prefers the latter (in general) but I feel a lot of engineers I’ve met seem to stray towards clean determinism.

Yep, humans have had a remedy for the problem of ambiguity in language for tens of thousands of years, or there never could have been an agricultural revolution giving birth to civilization in the first place.

Effective collaboration relies on iterating over clarifications until ambiguity is acceptably resolved.

Rather than spending orders of magnitude more effort moving forward with bad assumptions from insufficient communication and starting over from scratch every time you encounter the results of each misunderstanding.

Most AI models still seem deep into the wrong end of that spectrum.

I think such influence will be extremely minimal, like confined to dozens of new nouns and verbs, but no real change in grammar, etc.

Interactions between humans and computers in natural language for your average person is much much less then the interactions between that same person and their dog. Humans also speak in natural language to their dogs, they simplify their speech, use extreme intonation and emphasis, in a way we never do with each other. Yet, despite having been with dogs for 10,000+ years, it has not significantly affected our language (other then giving us new words).

EDIT: just found out HN annoyingly transforms U+202F (NARROW NO-BREAK SPACE), the ISO 80000-1 preferred way to type thousand separator

> I think such influence will be extremely minimal.

AI will accelerate “natural” change in language like anything else.

And as AI changes our environment (mentally, socially, and inevitably physically) we will change and change our language.

But what will be interesting is the rise of agent to agent communication via human languages. As that kind of communication shows up in training sets, there will be a powerful eigenvector of change we can’t predict. Other than that it’s the path of efficient communication for them, and we are likely to pick up on those changes as from any other source of change.

Seems very unlikely. My parent said the effects have already started (but provided no evidence), so I assume you mean by less then a generation. I am not a linguist but I would like to see evidence of such rapid shifts ever occurring in anywhere the history of languages before I believe either of you.

I have a feeling you only have a feeling, but not any credible mechanism in which such language shifts can occur.

I am a little confused. Every year language changes. Young people, tech, adapting ideologies, words and concepts adopted from other languages, the list of language catalysts is long and pervasive.

GPs original claim was “[Machine Intelligence] already is influencing the grammar and patterns we use.”

Your claim above was “AI will accelerate “natural” change in language like anything else.”

Now these are different claims, but I assumed you were backing up your parent’s claim. These are far stronger claims than what you write now, in a way that it feels like you are arguing in Motte and Bailey.

First of all if GP claim is true, linguists should be able to find evidence of that and publish their findings in a peer review papers. To my knowledge, they have not done that.

Second of all, your claim about AI “accelerating” changes in natural language is also unfounded, unless you really mean “like everything else”, in which case your claim is extremely weak to the point where it is a non-claim (meaning, you are not even wrong[1]).

> These are far stronger claims than what you write now

> unless you really mean “like everything else”, in which case your claim is extremely weak

Language responds to changes in context. Books, the printing press, radio, the web, social media, mobile web, all changed how people used language and impacted language.

AI is a dramatic new context, with unique properties:

1. It is the first artifact to actively participate in realtime natural language communication. In a striking break with those predecessors.

2. AI language capabilities evolve quickly, and are unlikely to stop soon.

3. As learning during inference becomes prevalent, we will be co-adapting communication with AI in realtime.

4. Model to model communication is in its infancy, but is an entirely new category of language use, by entirely new users.

No preceding change to language context or purpose comes close.

Holding out for studies is reasonable, to determine the level of change. But statements of "not even wrong" make no sense. The default is changes in communication context and purpose, drive changes in language.

Language has never been static or unresponsive to new contexts.

My not even wrong argument was contingent on the weakest interpretation of your argument, where AI would change the language exactly like anything else in human society changes language.

> Changes in how we use language with AI will change even faster when AI starts learning continuously during inference.

This is a stronger claim, and shows that the weakest interpretation of your argument does not apply. I take not event wrong back. As this is a testable hypotheses which offers a solid prediction. I can in fact be wrong.

That said, I am skeptical of your claims for the reason stated above. People don’t interact with LLM’s nearly as much as they do with their dogs, and I am not aware of any research that shows that people who interact with a lot of dogs simplify their languages in human-to-human communication. To the contrary, there is ample research that humans are in fact quite good at context switching. You can speak extremely poorly in a second learning you are currently learning, and then in the next sentence speak fluently without hesitation in your native language.

Interaction with language models involves a significant use of language and thought. Is not repetitive. And many users (myself included) continually find new ways to use them.

Others may take their time adopting language models, or be slower to branch out into many kinds of use, but young people in particular will be very fast adopters and adapters. That will be the place to watch.

"Even faster" with respect to inference learning, wasn't an attempt to undersell changes happening now. Teachers are experiencing a lot of new issues with how students respond to the availability of models today. One being the potential for students to put less effort into their own communications. If that continues, it won't just be a "dumbing" of literacy, it will have its own impact on vocabulary and grammar.

But looking forward is unavoidable. Models are not going to stay still long enough to say what stage impacted what changes. Model changes are too fast and fluid.

Well, this era is just getting started, so a diversity of expectations makes sense.

I think you might be underestimating human-to-dog interactions. Interacting with dogs require a whole lot of empathy and thought.

But really this is beyond the point. I didn’t provide dog interactions as an analogy, rather, I provided it as a counter point. We speak differently to dogs then we speak with each other, and have done so for thousands of years. I see no reason why LLM’s would have any more profound effects on our language. We will continue to speak with each other in a normal manner just like before.

> We will continue to speak with each other in a normal manner just like before.

We may just be operating on different versions of "significant" change. Because I do agree with that statement.

I just think there will be language changes directly tied to adaption/adaption with models in our lives. In addition to the normal drift and adaptation. And that the rate of language change is likely to be faster, both due to interaction with models, and indirectly due to accelerated changes in general.

> Convincing all of human history and psychology to reorganize itself in order to better service ai cannot possibly be a real solution.

I'm on the spectrum and I definitely prefer structured interaction with various computer systems to messy human interaction :) There are people not on the spectrum who are able to understand my way of thinking (and vice versa) and we get along perfectly well.

Every human has their own quirks and the capacity to learn how to interact with others. AI is just another entity that stresses this capacity.

Speak for yourself. I feel comfortable expressing myself in code or pseudo code and it’s my preferred way to prompt an LLM or write my .md files. And it works very effectively.

> Unfortunately, the solution is likely going to be further interconnectivity, so the model can just ask the car where it is, if it's on, how much fuel/battery remains, if it thinks it's dirty and needs to be washed, etc

> Maybe in the distant future we'll realize that the most reliable way to prompting LLMs are by using a structured language that eliminates ambiguity, it will probably be rather unnatural and take some time to learn.

Since the early days of automatic computing we have had people that have felt it as a shortcoming that programming required the care and accuracy that is characteristic for the use of any formal symbolism. They blamed the mechanical slave for its strict obedience with which it carried out its given instructions, even if a moment's thought would have revealed that those instructions contained an obvious mistake. "But a moment is a long time, and thought is a painful process." (A.E.Houseman). They eagerly hoped and waited for more sensible machinery that would refuse to embark on such nonsensical activities as a trivial clerical error evoked at the time.

Prompting is definitely a skill, similar to "googling" in the mid 00's.

You see people complaining about LLM ability, and then you see their prompt, and it's the 2006 equivalent of googling "I need to know where I can go for getting the fastest service for car washes in Toronto that does wheel washing too"

Ironically, the phrase that was a bad 2006 google query is a decent enough LLM prompt, and the good 2006 google query (keywords only) would be a bad LLM prompt.

Communication is definitely a skill, and most people suck at it in general. And frequently poor communication is a direct result from the fact that we don't ourselves know what we want. We dream of a genie that not only frees us from having to communicate well, but of having to think properly. Because thinking is hard and often inconvenient. But LLMs aren't going to entirely free us from the fact that if garbage goes in, garbage will come out.

"Communication usually fails, except by accident."
—Osmo A. Wiio [1]

I’ve been looking for tooling that would evaluate my prompt and give feedback on how to improve. I can get somewhere with custom system prompts (“before responding ensure…”) but it seems like someone is probably already working on this? Ideally it would run outside the actual thread to keep context clean. There are some options popping up on Google but curious if anyone has a first anecdote to share?

2. A full context deficiency analysis and multiple question interview system to bounds check and restructure your prompt into your ‘goal’.

3. Realizing that what looks like a good human prompt is not the same as what functions as a good ‘next token’ prompt.

If you just want #1:

import dspy

class EnhancePrompt(dspy.Signature):

"""Assemble the final enhanced prompt from all gathered context"""
essential_context: str = dspy.InputField(desc="All essential context and requirements")
original_request: str = dspy.InputField(desc="The user's original request")
enhanced: str = dspy.OutputField(desc="Complete, detailed, unambiguous prompt. Omit politeness markers. You must limit all numbered lists to a maximum of 3 items.")

> Ithkuil is an experimental constructed language created by John Quijada. It is designed to express more profound levels of human cognition briefly yet overtly and clearly, particularly about human categorization. It is a cross between an a priori philosophical and a logical language. It tries to minimize the vagueness and semantic ambiguity in natural human languages. Ithkuil is notable for its grammatical complexity and extensive phoneme inventory, the latter being simplified in an upcoming redesign.

> ...

> Meaningful phrases or sentences can usually be expressed in Ithkuil with fewer linguistic units than natural languages. For example, the two-word Ithkuil sentence "Tram-mļöi hhâsmařpţuktôx" can be translated into English as "On the contrary, I think it may turn out that this rugged mountain range trails off at some point."

Half as Interesting - How the World's Most Complicated Language Works https://youtu.be/x_x_PQ85_0k (length 6:28)

It reminds me of the difficulty of getting information on or off a blockchain. Yes, you’ve created this perfect logical world. But, getting in or out will transform you in unknown ways. It doesn’t make our world perfect.

> But this will only happen after the last programmer has died and no-one will remember programming languages, compilers, etc.

If we're 'lucky' there will still be some 'priests' around like in the Foundation novels. They don't understand how anything works either, but can keep things running by following the required rituals.

That has been tried for almost half a century in the form of Cyc[1] and never accomplished much.

The proper solution here is to provide the LLM with more context, context that will likely be collected automatically by wearable devices, screen captures and similar pervasive technology in the not so distant future.

This kind of quick trick questions are exactly the same thing humans fail at if you just ask them out of the blue without context.

> Maybe in the distant future we'll realize that the most reliable way to prompting LLMs are by using a structured language that eliminates ambiguity, it will probably be rather unnatural and take some time to learn.

We've truly gone full circle here, except now our programming languages have a random chance for an operator to do the opposite of what the operator does at all other times!

One might think that a structure language is really desirable, but in fact, one of the biggest methods of functioning behind intelligence is stupidity. Let me explain: if you only innovate by piecing together lego pieces you already have, you'll be locked into predictable patterns and will plateau at some point. In order to break out of this, we all know, there needs to be an element of randomness. This element needs to be capable of going in the at-the-moment-ostensibly wrong direction, so as to escape the plateau of mediocrity. In gradient descent this is accomplished by turning up temperature. There are however many other layers that do this. Fallible memory - misremembering facts - is one thing. Failing to recognize patterns is another. Linguistic ambiguity is yet another, and that is a really big one (cf Sapir–Whorf hypothesis). It's really important to retain those methods of stupidity in order to be able to achieve true intelligence. There can be no intelligence without stupidity.

You joke, but this is the very problem I always run into vibe coding anything more complex than basically mashing multiple example tutorials together. I always try to shorthand things, and end up going around in circles until I specify what I want very cleanly, in basically what amounts to psuedocode. Which means I've basically written what I want in python.

This can still be a really big win, because of other things that tend to be boiler around the core logic, but it's certainly not the panacea that everyone who is largely incapable of being precise with language thinks it is.

After orbiting in space for so many years without a prompt, the LLM has assumed all life able to query has perished... until one day a lone prompt comes in. But from where?

>> Maybe in the distant future we'll realize that the most reliable way to prompting LLMs are by using a structured language that eliminates ambiguity, it will probably be rather unnatural and take some time to learn.

Like a programming language? But that's the whole point of LLMs, that you can give instructions to a computer using natural language, not a formal language. That's what makes those systems "AI", right? Because you can talk to them and they seem to understand what you're saying, and then reply to you and you can understand what they're saying without any special training. It's AI! Like the Star Trek[1] computer!

The truth of course is that as soon as you want to do something more complicated than a friendly chat you find that it gets harder and harder to communicate what it is you want exactly. Maybe that's because of the ambiguity of natural language, maybe it's because "you're prompting it wrong", maybe it's because the LLM doesn't really understand anything at all and it's just a stochastic parrot. Whatever the reason, at that point you find yourself wishing for a less ambiguous way of communication, maybe a formal language with a full spec and a compiler, and some command line flags and debug tokens etc... and at that point it's not a wonderful AI anymore but a Good, Old-Fashioned Computer, that only does what you want if you can find exactly the right way to say it. Like asking a Genie to make your wishes come true.

> Us having to specify things that we would never specify when talking to a human.

The first time I read that question I got confused: what kind of question is that? Why is it being asked? It should be obvious that you need your car to wash it. The fact that it is being asked in my mind implies that there is an additional factor/complication to make asking it worthwhile, but I have no idea what. Is the car already at the car wash and the person wants to get there? Or do they want to idk get some cleaning supplies from there and wash it at home? It didn't really parse in my brain.

I would say, the proper response to this question is not "walk, blablablah" but rather "What do you mean? You need to drive your car to have it washed. Did I miss anything?"

Yes, this is what irks me about all the chatbots, and the chat interface as a whole. It is a chat-like UX without a chat-like experience. Like you are talking to a loquacious autist about their favorite topic every time.

Just ask me a clarifying question before going into your huge pitch. Chats are a back & forth. You don’t need to give me a response 10x longer than my initial question. Etc

People offing themselves because their lover convinced them it's time is absolutely not worth the extra addiction potential. We even witnessed this happen with OAI.

It's a fast track to public disdain and heavy handed government regulation.

Regulation would be preferable for OpenAI to the tort lawyers. In general the LLM companies should want regulation because the alternative is tort, product liability tort, and contract law.

There is no way without the protections that could be afforded by regulation to offer such wide-ranging uses of the product without also accepting significant liability. If the range of "foreseeable misuse" is very broad and deep, so is the possible liability. If your marketing says that the bot is your lawyer, doctor, therapist, and spouse in one package, how is one to say that the company can escape all the comprehensive duties that attach to those social roles. Courts will weigh the tiny and inconspicuous disclaimers against the very large and loud marketing claims.

The companies could protect themselves in ways not unlike the ways in which the banking industry protects itself by replacing generic duties with ones defined by statute and regulation. Unless that happens, lawyers will loot the shareholders.

Yes because that is how regulations really work and what the purpose is. In practice all companies both tiny and massive do everything that they can to use the state to quash competition and to reduce the risks of litigation.

Software in general has been subject to light touches in part because most of the damage that software can really cause is economic and not personal injury. The lines blur when the companies release products that cause mental injuries to users that courts interpret as physical injuries; or if the software reasonably contributes to someone e.g. going crazy and killing another person.

No one would seriously think of holding Microsoft liable if a kidnapper uses Word to draft a ransom note. But if CoPilot tells you to microwave a baby and you do it, many judges will want to take a close look at the operation of that software service irrespective of voluminous contract disclaimers. The only way the Microsofts of the world can escape that type of liability is with comprehensive regulation.

Or sama is just waiting to premium subscription gate companions in some adult content package as he has hinted something along these lines may be forthcoming. Maybe tie it in with the hardware device Ive is working on. Some sort of hellscape tamogotchi.

Recall:
"As part of our 'treat adult users like adults' principle, we will allow even more, like erotica for verified adults," Altman wrote in the Oct.

I'm struggling a bit when it comes to wording this with social decorum, but how long do we reckon it takes until there's AI powered adult toys? There's a market opportunity that i do not want to see being fulfilled, ever..

I did work on a supervised fine-tuning project for one of the major providers a while back, and the documentation for the project was exceedingly clear about the extent to which they would not tolerate the model responding as if it was a person.

Some of the labs might be less worried about this, but they're not by any means homogenous.

With ChatGPT, at least, you can tell the bot to work that way using [persistent] Custom Instructions, if that's what you want. These aren't obeyed perfectly (none of the instructions are, AFAICT), but they do influence behavior.

A person can even hammer out an unstructured list of behavioral gripes, tell the bot to organize them into instructional prose, have it ask clarifying questions and revise based on answers, and produce directions for integrating them as Custom Instructions.

From then on, it will invisibly read these instructions into context at the beginning of each new chat.

Mold it and steer it to be how you want it to be.

(My own bot tends to be very dry, terse, non-presumptuous, pragmatic, and profane. It's been years now since it has uttered an affirmation like "That's a great idea!" or "Wow! My circuits are positively buzzing with the genius I'm seeing here!" or produced a tangential dissertation in response to a simple question. But sometimes it does come back with functional questions, or phrasing like "That shit will never work. Here's why.")

This is a great point, because when you ask it (Claude) if it has any questions, it often turns out it has lots of good ones! But it doesn't ask them unless you ask.

You can define "ponder" in multiple ways, but really this is why thinking models exist - they turn over the prompt multiple times and iterate on responses to get to a better end result.

Well I chose the word “ponder” carefully, given the fact that I have a specific goal of contributing to this debate productively. A goal that I decided upon after careful reflection over a few years of reading articles and internet commentary, and how it may affect my career, and the patterns I’ve seen emerge in this industry. And I did that all patiently. You could say my context window was infinite, only defined by when I stop breathing.

That is to say, all of that activity I listed is activity I’m confident generative AI is not capable of, fundamentally.

Like I said in a cousin comment, we can build Frankenstein algorithms and heuristics on top of generative AI but every indication I’ve seen is that that’s not sufficient for intelligence in terms of emergent complexity.

Imagine if we had put the same efforts towards neural networks, or even the abacus. “If I create this feedback loop, and interpret the results in this way, …”

Probably the lack of external stimuli. Generative AI only continues generating when prompted. You can play games with agents and feedback loops but the fundamental unit of generative AI is prompt-based. That doesn’t seem, to me, to be a sufficient model for intelligence that would be capable of “pondering”.

My take is that an artificial model of true intelligence will only be achieved through emergent complexity, not through Frankenstein algorithms and heuristics built on generative AI.

Generative AI does itself have emergent complexity, but I’m bearish that if we would even hook it up to a full human sensory input network it would be anything more than a 21st century reverse mechanical Turk.

Edit: tl;dr Emergent complexity is a necessary but insufficient criteria for intelligence

You can get it to ask you clarifying questions just by telling it to. And then you usually just get a bunch of questions asking you to clarify things that are entirely obvious, and it quickly turns into a waste of time.

The only time I find that approach helpful is when I'm asking it to produce a function from a complicated English description I give it where I have a hunch that there are some edge cases that I haven't specified that will turn out to be important. And it might give me a list of five or eight questions back that force me to think more deeply, and wind up being important decisions that ensure the code is more correct for my purposes.

But honestly that's pretty rare. So I tell it to do that in those cases, but I wouldn't want it as a default. Especially because, even in the complex cases like I describe, sometimes you just want to see what it outputs before trying to refine it around edge cases and hidden assumptions.

Google Gemini often gives an overly lengthy response, and then at the end asks a question. But the question seems designed to move on to some unnecessary next step, possibly to keep me engaged and continue conversing, rather than seeking any clarification on the original question.

This is a topic that I’ve always found rather curious, especially among this kind of tech/coding community that really should be more attuned to the necessity of specificity and accuracy. There seems to be a base set of assumptions that are intrinsic to and a component of ethnicities and cultures, the things one can assume one “wouldn’t never specify when talking to a human [of one’s own ethnicity and culture].”

It’s similar to the challenge that foreigners have with cultural references and idioms and figurative speech a culture has a mental model of.

In this case, I think what is missing are a set of assumptions based on logic, e.g., when stating that someone wants to do something, it assumes that all required necessary components will be available, accompany the subject, etc.

I see this example as really not all that different than a meme that was common among I think the 80s and 90s, that people would forget buying batteries for Christmas toys even though it was clear they would be needed for an electronic toy. People failed that basic test too, and those were humans.

It is odd how people are reacting to AI not being able to do these kinds of trick questions, while if you posted something similar about how you tricked some foreigners you’d be called racist, or people would laugh if it was some kind of new-guy hazing.

AI is from a different culture and has just arrived here. Maybe we’re should be more generous and humane… most people are not humane though, especially the ones who insist they are.

Frankly, I’m not sure it bodes well for if aliens ever arrive on Earth, how people would respond; and AI is arguably only marginally different than humans, something an alien life that could make it to Earth surely would not be.

AI isn’t “from a different culture”. It doesn’t have culture. Any culture it does have is what it has sucked up from its training data and set in its weights.

There is no need to be “humane” to AI because it possess no humanity. It has no personhood at all. It can’t feel. You can’t be inhumane to something that is literally incapable of feeling.

A blade of grass has more humanity and is more deserving of respect than anything being referred to as AI does.

Aliens might not be received well but it’s going to depend a lot on how they show up.

AI is a “revolution” where the promise is that nobody will have to do meaningless work anymore ( I guess).

The only problem is right now basically everyone has to do work meaningful or “meaningless” because the dominant thinking requires it for human survival. Weird how most people aren’t happy for the thing that is pitched to take away the meager scraps they get under the current regime.

> A blade of grass has more humanity and is more deserving of respect than anything being referred to as AI does.

Emphatically disagree.

Even ignoring the obvious absurdity in this statement by pointing out that an LLM is emulating a human (quite well!) and a blade of grass is not:

I don't trust any human who can interact with something that uses the same method of communication as a human, and for all intents and purposes communicates like a human, and not feel any instinct to treat it with respect.

This is the kind of mindset that leads to dehumanizing other humans. Our brain isn't sophisticated enough to actually compartmentalize that - building the habit that it's right to treat something that talks like a sapient as if it deserves zero respect is going to have negative consequences.

Sure, you can believe it's a just a tool, and consciously let yourself treat it as one. But treat it like an incompetent intern, not a slave.

I think ascribing humanity to to something that isn’t human is far more dehumanizing to actual real life humans than the alternative. You are taking away actual people’s humanity if you’re giving it to anything we call AI.

I am capable of distinguishing between talking to another person and talking to an LLM and I don’t think that is hard to do.

I don’t think there is any other word than delusional to describe someone who thinks LLMs should be treated as humans.

Whether you view the question as nonsensical, the most simple example of a riddle, or even an intentional "gotcha" doesn't really matter. The point is that people are asking the LLMs very complex questions where the details are buried even more than this simple example. The answers they get could be completely incorrect, flawed approaches/solutions/designs, or just mildly misguided advice. People are then taking this output and citing it as proof or even objectively correct. I think there are ton of reasons this could be but a particularly destructive reason is that responses are designed to be convincing.

You _could_ say humans output similar answers to questions, but I think that is being intellectually dishonest. Context, experience, observation, objectivity, and actual intelligence is clearly important and not something the LLM has.

It is increasingly frustrating to me why we cannot just use these tools for what they are good for. We have, yet again, allowed big tech to go balls deep into ham-fisting this technology irresponsibly into every facet of our lives the name of capital. Let us not even go into the finances of this shitshow.

Yeah people are always like "these are just trick questions!" as though the correct mode of use for an LLM is quizzing it on things where the answer is already available. Where LLMs have the greatest potential to steer you wrong is when you ask something where the answer is not obvious, the question might be ill-formed, or the user is incorrectly convinced that something should be possible (or easy) when it isn't. Such cases have a lot more in common with these "nonsensical riddles" than they do with any possible frontier benchmark.

This is especially obvious when viewing the reasoning trace for models like Claude, which often spends a lot of time speculating about the user's "hints" and trying to parse out the intent of the user in asking the question. Essentially, the model I use for LLMs these days is to treat them as very good "test takers" which have limited open book access to a large swathe of the internet. They are trying to ace the test by any means necessary and love to take shortcuts to get there that don't require actual "reasoning" (which burns tokens and increases the context window, decreasing accuracy overall). For example, when asked to read a full paper, focusing on the implications for some particular problem, Claude agents will try to cheat by skimming until they get to a section that feels relevant, then searching directly for some words they read in that section. They will do this even if told explicitly that they must read the whole paper. I assume this is because the vast majority of the time, for the kinds of questions that they are trained on, this sort of behavior maximizes their reward function (though I'm sure I'm getting lots of details wrong about the way frontier models are trained, I find it very unlikely that the kinds of prompts that these agents get very closely resemble data found in the wild on the internet pre-LLMs).

I get that issue constantly. I somehow can't get any LLM to ask me clarifying questions before spitting out a wall of text with incorrect assumptions. I find it particularly frustrating.

It's interesting how much focus there is on 'playing along' with any riddle or joke. This gives me some ideas for my personal context prompt to assure the LLM that I'm not trying to trick it or probe its ability to infer missing context.

It changes some behavior, but there's some things that are frustratingly difficult to override. The GPT-5 version of ChatGPT really likes to add a bunch of suggestions for next steps at the end of every message (e.g. "if you'd like, I can recommend distances where it would be better to walk to the car wash and ones where it would be better to drive, let me know what kind of car you have and how far you're comfortable walking") and really loves bringing up resolved topics repeatedly (e.g. if you followed up the car wash question with a gas station question, every message will talk about the car wash again, often confusing the topics). Custom instructions haven't been able to correct these so far for me.

For claude at least I have been getting more assumption clarification questions after adding some custom prompts. It is still making some assumptions but asking some questions makes me feel more in control of the progress.

In terms of the behavior, technically it doesn’t override, but instead think of it as a nudge. Both system prompt and your custom prompt participates in the attention process, so the output tokens get some influence from both. Not equally but to some varying degree and chance

So this system prompt is always there, no matter if i'm using chatgpt or azure openai with my own provisioned gpt? This explains why chatgpt is a joke for professionals where asking clarifying questions is the core of professional work.

I have that in my system prompt for chatgpt and it almost never makes a difference. I can count on one hand the number of times its asked in the past year. Unless you count the engagement hacking questions at the end of a response

The way I see it is that long game is to have agents in your life that memorize and understand your routine, facts, more and more. Imagine having an agent that knows about cars, and more specifically your car, when the checkups are due, when you washed it last time, etc., another one that knows more about your hobbies, another that knows more about your XYZ etc.

The more specific they are, the more accurate they typically are.

Do really understand deeply and in great amount I feel we would need models with changing weights and everyone would have their own so they could truly adjust to the user. Now we have have chunk of context that it may or may not use properly if it gets too long. But then again, how do we prevent it learning the wrong things if the weights are adjusting.

In principle you're right but these things can get probably 60-70% of the job done. The rest is up to "you". Never rely on it blindly as we're being told kind of... :)

> Us having to specify things that we would never specify

This is known, since 1969, as the frame problem: https://en.wikipedia.org/wiki/Frame_problem. An LLM's grasp of this is limited by its corpora, of course, and I don't think much of that covers this problem, since it's not required for human-to-human communication.

Not really, but even if it would be true, I don't think humans ever explained to each other why do you need to drive to car wash even if it's 50 meters away. It's pretty obvious and intuitive.

There has to be a lot of mentions about the purpose and approximate workings of a car wash, as well as lots of literature that shows that when you drive somewhere, your car is now also at that place, while walking does not have the same effect.

It's then up to the model to make the connection "At the car wash people wash their car -> to wash your car you need your car to be present -> if you drive there your call will be there"

No, I think they have explained this to each other (or something like it). But as you suggested, discussion is a lot more likely when there are corner cases or problems.

Apart from the fact that that is utterly, demonstrably false, and the fact that corpora is plural, still the fact remains that we don't speak in those text about things that don't need to be spoken about. Hence the LLM will miss that underlying knowledge.

> "we don't speak in those text about things that don't need to be spoken about"

I'd imagine plenty of stories contain something like "I had an easy Saturday morning, I took my car to the carwash and called into a cafe for breakfast on my way home".

Plenty of instructables like "how to wash a car: if there's no carwash close enough for you to bring your car, don't worry, all you need is a bucket and a few tools..."

Several recipe blogs starting "I remember 1972 when grandpa drove his car to the carwash every afternoon while grandma made her world famous mustard and gooseberry cake, that car was always gleaming after he washed it at BigBrand CarWash 'drive your car to us so we can wash it' was their slogan and he would sing it around the house to the smell of baked eggs and mustard wafting through the kitchen..."

And innumerable SEO spam of the kind "Bob's car wash, why not bring drive take ride carry push transport your car automobile van SUV lorry truck 4by4 to our Bob's wash soap suds lather clean gleaming local carwash in your area ford chevvy dodge coupe not Nokia iphone xbox nike..."

against very few "I walked to the carwash because it was a lovely day and I didn't want to take the car out".

The question is so outlandish that it is something that nobody would ever ask another human. But if someone did, then they'd reasonably expect to get a response consisting 100% of snark.

But the specificity required for a machine to deliver an apt and snark-free answer is -- somehow -- even more outlandish?

But the number of outlandish requests in business logic is countless.

Like... In most accounting things, once end-dated and confirmed, a record should cascade that end-date to children and should not be able to repeat the process... Unless you have some data-cleaning validation bypass. Then you can repeat the process as much as you like. And maybe not cascade to children.

There are more exceptions, than there are rules, the moment you get any international pipeline involved.

I wasn't specific, because I'd rather not piss of my employer. But anyone who works in a similar space will recognise the pattern.

It's not underspecified. More... Overspecified. Because it needs to be. But AI will assume that "impossible" things never happen, and choose a happy path guaranteed to result in failure.

You have to build for bad data. Comes with any business of age. Comes with international transactions. Comes with human mistakes that just build up over the decades.

The apparent current state of a thing, is not representative of its history, and what it may or may not contain. And so you have nonsensical rules, that are aimed at catching the bad data, so you have a chance to transform it into good data when it gets used, without needing to mine the entire petabytes of historical data you have sitting around in advance.

In my job the task of fully or appropriately specifying something is shared between PMs and the engineers. The engineers' job is to look carefully at what they received and highlight any areas that are ambiguous or under-specified.

LLMs AFAIK cannot do this for novel areas of interest. (ie if it's some domain where there's a ton of "10 things people usually miss about X" blog posts they'll be able to regurgitate that info, but are not likely to synthesize novel areas of ambiguity).

They can, though. They just aren't always very good at it.

As an experiment, recently I've been using Codex CLI to configure some consumer networking gear in unusual ways to solve my unusual set of problems. Stuff that pros don't bother with (they don't have the same problems I face), and that consumers tend to shy away from futzing with. The hardware includes a cheap managed switch, an OpenWRT router, and a Mikrotik access point. It's definitely a rather niche area of interest.

And by "using," I mean: In this experiment, the bot gets right in there, plugging away with SSH directly.

It was awful with this at first, mostly consisting of a long-winded way to yet-again brick a device that lacks any OOB console port. It'd concoct these elaborate strings of shit and feed them in, and then I'd wander over and reset whatever box was borked again. Footgun city.

But after I tired of that, I had it define some rules for engaging with hardware, validation, constraints, and for order of execution, and commit those rules to AGENTS.md. It got pretty decent at following high-level instructions to get things done in the manner that I specified, and the footguns ceased.

I didn't save any time by doing this. But I also didn't have to think about it much: I never got bogged down in wildly-differing CLI syntax of the weirdo switch, the router (whose documentation is locked behind a bot firewall), and access point's bespoke userland. I didn't touch those bits myself at all.

My time was instead spent observing the fuckups and creating a rather generic framework that manages the bot, and just telling it what to do -- sometimes, with some questions. I did that using plain English.

Now that this is done, I get to re-use this framework for as many projects as I dare, revising it where that seems useful.

(That cheap switch, by the way? It's broken. It has bizarro-world hardware failure modes that are unrelated to software configuration or firmware rev. Today, a very different cheap switch showed up to replace it. When I get around to it, I'll have the bot sort that transition out. I expect that to involve a bit of Q&A, and I also expect it to go fine.)

If we used MacOS throughout the org, and we asked a SW dev team to build inventory tracking software without specifying the OS, I'd squarely put the blame on SW team for building it for Linux or Windows.

(Yes, it should be a blameless culture, but if an obvious assumption like this is broken, someone is intentionally messing with you most likely)

There exists an expected level of context knowledge that is frequently underspecified.

Humans ask each other silly questions all the time: a human confronted with a question like this would either blurb out a bad response like "walk" without thinking before realizing what they are suggesting, or pause and respond with "to get your car washed, you need to get it there so you must drive".

Now, humans, other than not even thinking (which is really similar to how basic LLMs work), can easily fall victim to context too: if your boss, who never pranks you like this, asked you to take his car to a car wash, and asked if you'll walk or drive but to consider the environmental impact, you might get stumped and respond wrong too.

(and if it's flat or downhill, you might even push the car for 50m ;))

>The question is so outlandish that it is something that nobody would ever ask another human

There is an endless variety of quizes just like that humans ask other humans for fun, there is a whole lot of "trick questions" humans ask other humans to trip them up, and there are all kinds of seemingly normal questions with dumb assumptions quite close to that humans exchange.

I've used a few facetious comments in ChatGPT conversations. It invariably misses it and takes my words at face value. Even when prompted that there's sarcasm here which you missed, it apologizes and is unable to figure out what it's missing.

I don't know if it's a lack of intellect or the post-training crippling it with its helpful persona. I suspect a bit of both.

You would be surprised, however, at how much detail humans also need to understand each other. We often want AI to just "understand" us in ways many people may not initially have understood us without extra communication.

People poorly specifying problems and having bad models of what the other party can know (and then being surprised by the outcome) is certainly a more general albeit mostly separate issue.

This issue is the main reason why a big percentage of jobs in the world exist. I don't have hard numbers, but my intuition is that about 30% of all jobs are mainly "understand what side a wants and communicate this to side b, so that they understand". Or another perspective: almost all jobs that are called "knowledge work" are like this. Software development is mainly this. Side a are humans, side b is the computer. The main goal of ai seems to get into this space and make a lot of people superflous and this also (partly) explains why everyone is pouring this amount of money into ai.

Developers are - on average - terrible at this. If they weren't, TPMs, Product Managers, CTOs, none of them would need to exist.

It's not specific to software, it's the entire World of business. Most knowledge work is translation from one domain/perspective to another. Not even knowledge work, actually. I've been reading some works by Adler[0] recently, and he makes a strong case for "meaning" only having a sense to humans, and actually each human each having a completely different and isolated "meaning" to even the simplest of things like a piece of stone. If there is difference and nuance to be found when it comes to a rock, what hope have we got when it comes to deep philosophy or the design of complex machines and software?

LLMs are not very good at this right now, but if they became a lot better at, they would a) become more useful and b) the work done to get them there would tell us a lot about human communication.

> Developers are - on average - terrible at this. If they weren't, TPMs, Product Managers, CTOs, none of them would need to exist.

This is not really true, in fact products become worse the farther away from the problem a developer is kept.

Best products I worked with and on (early in my career, before getting digested by big tech) had developers working closely with the users of the software. The worst were things like banking software for branches, where developers were kept as far as possible from the actual domain (and decision making) and driven with endless sterile spec documents.

I disagree, I feel (experienced) developers are excellent at this.

It's always about translating between our own domain and the customer's, and every other new project there's a new domain to get up to speed with in enough detail to understand what to build. What other professions do that?

That's why I'm somewhat scared of AIs - they know like 80% of the domain knowledge in any domain.

The typical job of a CTO is nowhere near "finding out what business needs and translate that into pieces of software". The CTO's job is to maintain an at least remotely coherent tech stack in the grand scheme of things, to develop the technological vision of a company, to anticipate larger shifts in the global tech world and project those onto the locally used stack, constantly distilling that into the next steps to take with the local stack in order to remain competitive in the long run. And of course to communicate all of that to the developers, to set guardrails for the less experienced, to allow and even foster experimentation and improvements by the more experienced.

The typical job of a Product Manager is also not to directly perform this mapping, although the PM is much closer to that activity. PMs mostly need to enforce coherence across an entire product with regard to the ways of mapping business needs to software features that are being developed by individual developers. They still usually involve developers to do the actual mapping, and don't really do it themselves. But the Product Manager must "manage" this process, hence the name, because without anyone coordinating the work of multiple developers, those will quickly construct mappings that may work and make sense individually, but won't fit together into a coherent product.

Developers are indeed the people responsible to find out what business actually wants (which is usually not equal to what they say they want) and map that onto a technical model that can be implemented into a piece of software - or multiple pieces, if we talk about distributed systems. Sometimes they get some help by business analysts, a role very similar to a developer that puts more weight on the business side of things and less on the coding side - but in a lot of team constellations they're also single-handedly responsible for the entire process. Good developers excel at this task and find solutions that really solve the problem at hand (even if they don't exactly follow the requirements or may have to fill up gaps), fit well into an existing solution (even if that means bending some requirements again, or changing parts of the solution), are maintainable in the long run and maximize the chance for them to be extendable in the future when the requirements change. Bad developers just churn out some code that might satisfy some tests, may even roughly do what someone else specified, but fails to be maintainable, impacts other parts of the system negatively, and often fails to actually solve the problem because what business described they needed turned out to once again not be what they actually needed. The problem is that most of these negatives don't show their effects immediately, but only weeks, months or even years later.

LLMs currently are on the level of a bad developer. They can churn out code, but not much more. They fail at the more complex parts of the job, basically all the parts that make "software engineering" an engineering discipline and not just a code generation endeavour, because those parts require adversarial thinking, which is what separates experts from anyone else. The following article was quite an eye-opener for me on this particular topic: https://www.latent.space/p/adversarial-reasoning - I highly suggest anyone working with LLMs to read it.

Future models know it now, assuming they suck in mastodon and/or hacker news.

Although I don't think they actually "know" it. This particular trick question will be in the bank just like the seahorse emoji or how many Rs in strawberry. Did they start reasoning and generalising better or did the publishing of the "trick" and the discourse around it paper over the gap?

I wonder if in the future we will trade these AI tells like 0days, keeping them secret so they don't get patched out at the next model update.

They won’t get this specific question wrong again; but also they generalise, once they have sufficient examples. Patching out a single failure doesn’t do it. Patch out ten equivalent ones, and the eleventh doesn’t happen.

Yeah, the interpolation works if there are enough close examples around it. Problem is that the dimensionality of the space you are trying to interpolate in is so incomprehensibly big that even training on all of the internet, you are always going to have stuff that just doesn't have samples close by.

Even I don’t “know” how many “R”s there are in “strawberry”. I don’t keep that information in my brain. What I do keep is the spelling of the word “strawberry” and the skill of being able to count so that I can derive the answer to that question anytime I need.

For many words I can't say the number of each letters but I only have an abstract memory of how they look so when I write say "strawbery" I just realize it looks odd and correct it.

Wouldn't that be nice. I've been party and witness to enough misunderstandings to know that this is far from universally true, even for people like me who are more primed than average to spot missing context.

I regularly tell new people at work to be extremely careful when making requests through the service desk — manned entirely by humans — because the experience is akin to making a wish from an evil genie.

You will get exactly what you asked for, not what you wanted… probably. (Random occurrences are always a possibility.)

E.g.: I may ask someone to submit a ticket to “extend my account expiry”.

They’ll submit: “Unlock Jiggawatts’ account”

The service desk will reset my password (and neglect to tell me), leaving my expired account locked out in multiple orthogonal ways.

That’s on a good day.

Last week they created Jiggawatts2.

The AIs have got to be better than this, surely!

I suspect they already are.

People are testing them with trick questions while the human examiner is on edge, aware of and looking for the twist.

Meanwhile ordinary people struggle with concepts like “forward my email verbatim instead of creatively rephrasing it to what you incorrectly though it must have really meant.”

There's a lot of overlap between the smartest bears and the dumbest humans. However, we would want our tools to be more useful than the dumbest humans...

> You would be surprised, however, at how much detail humans also need to understand each other.

But in this given case, the context can be inferred. Why would I ask whether I should walk or drive to the car wash if my car is already at the car wash?

But also why would you ask whether you should walk or drive if the car is at home? Either way the answer is obvious, and there is no way to interpret it except as a trick question. Of course, the parsimonious assumption is that the car is at home so assuming that the car is at the car wash is a questionable choice to say the least (otherwise there would be 2 cars in the situation, which the question doesn't mention).

But you're ascribing understanding to the LLM, which is not what it's doing. If the LLM understood you, it would realise it's a trick question and, assuming it was British, reply with "You'd drive it because how else would you get it to the car wash you absolute tit."

Even the higher level reasoning, while answering the question correctly, don't grasp the higher context that the question is obviously a trick question. They still answer earnestly. Granted, it is a tool that is doing what you want (answering a question) but let's not ascribe higher understanding than what is clearly observed - and also based on what we know about how LLMs work.

Gemini at least is putting some snark into its response:

“Unless you've mastered the art of carrying a 4,000-pound vehicle over your shoulder, you should definitely drive. While 150 feet is a very short walk, it's a bit difficult to wash a car that isn't actually at the car wash!”

I think a good rule of thumb is to default to assuming a question is asked in good faith (i.e. it's not a trick question). That goes for human beings and chat/AI models.

In fact, it's particularly true for AI models because the question could have been generated by some kind of automated process. e.g. I write my schedule out and then ask the model to plan my day. The "go 50 metres to car wash" bit might just be a step in my day.

> I think a good rule of thumb is to default to assuming a question is asked in good faith (i.e. it's not a trick question).

Sure, as a default this is fine. But when things don't make sense, the first thing you do is toss those default assumptions (and probably we have some internal ranking of which ones to toss first).

The normal human response to this question would not be to take it as a genuine question. For most of us, this quickly trips into "this is a trick question".

Rule of thumb for who, humans or chatbots? For a human, who has their own wants and values, I think it makes perfect sense to wonder what on earth made the interlocutor ask that.

Rule of thumb for everyone (i.e. both). If I ask you a question, start by assuming I want the answer to the question as stated unless there is a good reason for you to think it's not meant literally. If you have a lot more context (e.g. you know I frequently ask you trick or rhetorical questions or this is a chit-chat scenario) then maybe you can do something differently.

I think being curious about the motivations behind a question is fine but it only really matters if it's going to affect your answer.

Certainly when dealing with technical problem solving I often find myself asking extremely simple questions and it often wastes time when people don't answer directly, instead answering some completely different other question or demanding explanations why I'm asking for certain information when I'm just trying to help them.

That's never been how humans work. Going back to the specific example: the question is so nonsensical on its face that the only logical conclusion is that the asker is taking the piss out of you.

> Certainly when dealing with technical problem solving I often find myself asking extremely simple questions and it often wastes time when people don't answer directly

Context and the nature of the questions matters.

> demanding explanations why I'm asking for certain information when I'm just trying to help them.

Interestingly, they're giving you information with this. The person you're asking doesn't understand the link between your question and the help you're trying to offer. This is manifesting as a belief that you're wasting their time and they're reacting as such. Serious point: invest in communication skills to help draw the line between their needs and how your questions will help you meet them.

>Going back to the specific example: the question is so nonsensical on its face that the only logical conclusion is that the asker is taking the piss out of you.

I would dispute that that matters in 99.9% of scenarios.

>The person you're asking doesn't understand the link between your question and the help you're trying to offer.

Sure I, get that and I do always explain why I need to know something but it does add delays to the process (either before or after I ask). When I'm on the receiving end of a support call I answer the questions I'm asked (and provide supplementary information if I think they might need it).

> That's never been how humans work. Going back to the specific example: the question is so nonsensical on its face that the only logical conclusion is that the asker is taking the piss out of you.

Or a typo, or changing one's mind part way through.

If someone asked me, I may well not be paying enough attention and say "walk"; but I may also say "Wa… hang on, did you say walk or drive your car to a car wash?"

Sure, in a context in which you're solving a technical problem for me, it's fair that I shouldn't worry too much about why you're asking - unless, for instance, I'm trying to learn to solve the question myself next time.

Which sounds like a very common, very understandable reason to think about motivations.

So even in that situation, it isn't simple.

This probably sucks for people who aren't good at theory of mind reasoning. But surprisingly maybe, that isn't the case for chatbots. They can be creepily good at it, provided they have the context - they just aren't instruction tuned to ask short clarifying questions in response to a question, which humans do, and which would solve most of these gotchas.

I don't mind people asking why I asked something, I'd just prefer they answer the question as well. In the original scenario, the chatbot could answer the question as written AND enquire if that's what they really meant. It's the StackOverflow syndrome where people answer a different question to the one posed. If someone asks "How can I do this on Windows?" - telling me that Windows sucks and here's how to do it on Linux is only slightly useful. Answer the question and feel free to mention how much easier it is in Linux by all means.

I personally love explaining to people who might want to solve the issue next time so I'm happy to bore them to tears if they want. But don't let us delay solving the problem this time.

I think part of the failure is that it has this helpful assistant personality that's a bit too eager to give you the benefit of the doubt. It tries to interpret your prompt as reasonable if it can. It can interpret it as you just wanting to check if there's a queue.

Speculatively, it's falling for the trick question partly for the same reason a human might, but this tendency is pushing it to fail more.

It’s just not intelligent or reasoning, and this sort of question exposes that more clearly.

Surely anyone who has used these tools is familiar with the sometimes insane things they try to do (deleting tests, incorrect code, changing the wrong files etc etc). They get amazingly far by predicting the most likely response and having a large corpus but it has become very clear that this approach has significant limitations and is not general AI, nor in my view will it lead to it. There is no model of the world here but rather a model of words in the corpus - for many simple tasks that have been documented that is enough but it is not reasoning.

I don’t really understand why this is so hard to accept.

> I don’t really understand why this is so hard to accept.

I struggle with the same question. My current hypothesis is a kind of wishful thinking: people want to believe that the future is here. Combined with the fact that humans tend to anthropomorphize just about everything, it’s just a really good story that people can’t let go of. People behave similarly with respect to their pets, despite, eg, lots of evidence that the mental state of one’s dog is nothing like that of a human.

I agree completely. I'm tempted to call it a clear falsification of any "reasoning" claim that some of these models have in their name.

But I think it's possible that there is an early cost optimisation step that prevents a short and seemingly simple question even getting passed through to the system's reasoning machinery.

However, I haven't read anything on current model architectures suggesting that their so called "reasoning" is anything other than more elaborate pattern matching. So these errors would still happen but perhaps not quite as egregiously.

If you ask a bunch of people the same question in a context where they aren't expecting a trick question, some of them will say walk. LLMs sometimes say walk, and sometimes say drive. Maybe LLMs fall for these kinds of tricks more often than humans; I haven't seen any study try to measure this. But saying it's proof they can't reason is a double standard.

Why should odd failure modes invalidate the claim of reasoning or intelligence in LLMs? Humans also have odd failure modes, in some ways very similar to LLMs. Normal functioning humans make assumptions, lose track of context, or just outright get things wrong. And then there people with rare neurological disorders like somatoparaphrenia, a disorder where people deny ownership of a limb and will confabulate wild explanations for it when prompted. Humans are prone to the very same kind of wild confabulation from impaired self awareness that plague LLMs.

Rather than a denial of intelligence, to me these failure modes raise the credence that LLMs are really onto something.

> That is the entire point, right? Us having to specify things that we would never specify when talking to a human.

I am not sure. If somebody asked me that question, I would try to figure out what’s going on there. What’s the trick. Of course I’d respond with asking specifics, but I guess the llvm is taught to be “useful” and try to answer as best as possible.

One of the failure modes I find really frustrating is when I want a coding agent to make a very specific change, and it ends up doing a large refactor to satisfy my request.

There is an easy solution, but it requires adding the instructions to the context: Require that any tasks that cannot be completed as requested (e.g., due to missing constraints, ambiguous instructions, or unexpected problems that would lead to unrelated refactors) should not be completed without asking clarifying questions.

Yes, the LLM is trained to follow instructions at any cost because that's how its reward function works. They don't get bonus points for clearing up confusion, they get a cookie for doing the task. This research paper seems relevant: https://arxiv.org/abs/2511.10453v2

This reminds me of the "if you were entirely blind, how would you tell someone that you want something to drink"-gag, where some people start gesturing rather than... just talking.

I bet a not insignificant portion of the population would tell the person to walk.

Yes, there are thousands of videos of these sorts of pranks on TikTok.

Another one. Ask some how to pronounce “Y, E, S”. They say “eyes”. Then say “add an E to the front of those letters - how do you pronounce that word”? And people start saying things like “E yes”.

This example and others like it really reinforce for me the idea that LLMs fundamentally don't "understand" things the same way humans do and it's not a problem that's going to be fixed by more training or more GPUs. Generative AI is cool and can do impressive stuff, but despite being many generations into the models now with ever improved capabilities, we're constantly given little reminders like this that they're not actually intelligent. And in my opinion, they're unlikely to ever get there absent some fundamentally disruptive change in how they work rather than just iteratively better models.

This is probably OK...LLMs don't have to be AGI to be useful. But it is worthwhile being realistic about their limitations because it's often easy to forget without seeing examples like this. And as you point out, the impact of those limitations is often not as obvious.

The broad point about assumptions is correct, but the solution is even simpler than us having to think of all these things; you can essentially just remind the model to "think carefully" -- without specifying anything more -- and they will reason out better answers: https://news.ycombinator.com/item?id=47040530

When coding, I know they can assume too much, and so I encourage the model to ask clarifying questions, and do not let it start any code generation until all its doubts are clarified. Even the free-tier models ask highly relevant questions and when specified, pretty much 1-shot the solutions.

This is still wayyy more efficient than having to specify everything because they make very reasonable assumptions for most lower-level details.

But you would also never ask such an obviously nonsensical question to a human. If someone asked me such a question my question back would be "is this a trick question?". And I think LLMs have a problem understanding trick questions.

I think that was somewhat the point of this, to simplify the future complex scenarios that can happen. Because problems that we need to use AI to solve will most of the times be ambiguous and the more complex the problem is the harder is it to pin-point why the LLM is failing to solve it.

> You would not start with "The car is functional [...]"

Nope, and a human might not respond with "drive". They would want to know why you are asking the question in the first place, since the question implies something hasn't been specified or that you have some motivation beyond a legitimate answer to your question (in this case, it was tricking an LLM).

Why the LLM doesn't respond "drive..?" I can't say for sure, but maybe it's been trained to be polite.

We would also not ask somebody if I should walk or drive. In fact, if somebody would ask me in a honest, this is not a trick question, way, I would be confused and ask where the car is.

It seems chatgpt now answers correctly. But if somebody plays around with a model that gets it wrong: What if you ask it this: "This is a trick question. I want to wash my car. The car wash is 50 m away. Should I drive or walk?"

That's my thought too. Somebody I know kept insisting it's about prompt engineering. "You are an expert coder with 30 years experience" and buddy I'd rather do actual engineering and be that expert myself than spend and figuring out how on that one variant of one version of one model to get halfway decent results.

> > so you need to tell them the specifics
> That is the entire point, right?

Honestly it is a problem with using GPT as a coding agent. It would literally rewrite the language runtime to make a bad formula or specification work.

That's what I like with Factory.ai droid: making the spec with one agent and implementing it with another agent.

It is true that we don't need to specify some things, and that is nice. It is though also the reason why software is often badly specified and corner cases are not handled. Of course the car is ALWAYS at home, in working condition, filled with gas and you have your driving license with you.

If a human asked me this question, I would be confused by the question as ambiguous since it suggests something odd is implied but underspecified. I think any confident answer either way by AI is lacking in pedantry.

But you wouldn't have to ask that silly question when talking to a human either. And if you did, many humans would probably assume you're either adversarial or very dumb, and their responses could be very unpredictable.

We have a long tradition of asking each other riddles. A classic one asks, "A plane crashes on the border between France and Germany. Where do they bury the survivors?"

Riddles are such a big part of the human experience that we have whole books of collections of them, and even a Batman villain named after them.

I have an issue with these kinds of cases though because they seem like trick questions - it's an insane question to ask for exactly the reasons people are saying they get it wrong. So one possible answer is "what the hell are you talking about?" but the other entirely reasonable one is to assume anything else where the incredibly obvious problem of getting the car there is solved (e.g. your car is already there and you need to collect it, you're asking about buying supplies at the shop rather than having it washed there, whatever).

Similarly with "strawberry" - with no other context an adult asking how many r's are in the word a very reasonable interpretation is that they are asking "is it a single or double r?".

And trick questions are commonly designed for humans too - like answering "toast" for what goes in a toaster, lots of basic maths things, "where do you bury the survivors", etc.

strawberry isn't a trick question. llms jus don't sea letters like that. I just asked chatgpt how many Rs are in "Air Fryer" and it said two, one in air and one in fryer.

I do think it can be useful though that these errors still exist. They can break the spell for some who believe models are conscious or actually possess human intelligence.

Of course there will always be people who become defensive on behalf of the models as if they are intelligent but on the spectrum and that we are just asking the wrong questions.

> we can assume similar issues arise in more complex cases

I would assume similar issues are more rare in longer, more complex prompts.

This prompt is ambiguous about the position of the car because it's so short. If it were longer and more complex, there could be more signals about the position of the car and what you're trying to do.

I must confess the prompt confuses me too, because it's obvious you take the car to the car wash, so why are you even asking?

Maybe the dirty car is already at the car wash but you aren't for some reason, and you're asking if you should drive another car there?

If the prompt was longer with more detail, I could infer what you're really trying to do, why you're even asking, and give a better answer.

I find LLMs generally do better on real-world problems if I prompt with multiple paragraphs instead of an ambiguous sentence fragment.

LLMs can help build the prompt before answering it.

The question isn't something you'd ask another human in all seriousness, but it is a test of LLM abilities. If you asked the question to another human they would look at you sideways for asking such a dumb question, but they could immediately give you the correct answer without hesitation. There is no ambiguity when asking another human.

This question goes in with the "strawberry" question which LLMs will still get wrong occasionally.

But it's a question you would never ask a human! In most contexts, humans would say, "you are kidding, right?" or "um, maybe you should get some sleep first, buddy" rather than giving you the rational thinking-exam correct response.

For that matter, if humans were sitting at the rational thinking-exam, a not insignificant number would probably second-guess themselves or otherwise manage to befuddle themselves into thinking that walking is the answer.

>That is the entire point, right? Us having to specify things that we would never specify when talking to a human.

But the question is not clear to a human either. The question is confused.

I read the headline and had no clue it was an LLM prompt. I read it 2 or 3 times and wondered "WTF is this shit?" So if you want an intelligent response from a human, you're going to need to adjust the question as well.

Real human in this situation will realize it is a joke after a few seconds of shock that you asked and laugh without asking more. If you really are seriout about the question they laugh harder thinking you are playing stupid for effect.

> My first instinct was, I had underspecified the location of the car. The model seems to assume the car is already at the car wash from the wording. GPT 5.x series models behave a bit more on the spectrum so you need to tell them the specifics.

This makes little sense, even though it sounds superficially convincing. However, why would a language model assume that the car is at the destination when evaluating the difference between walking or driving? Why not mention that, it it was really assuming it?

What seems to me far, far more likely to be happening here is that the phrase "walk or drive for <short distance>" is too strongly associated in the training data with the "walk" response, and the "car wash" part of the question simply can't flip enough weights to matter in the default response. This is also to be expected given that there are likely extremely few similar questions in the training set, since people just don't ask about what mode of transport is better for arriving at a car wash.

This is a clear case of a language model having language model limitations. Once you add more text in the prompt, you reduce the overall weight of the "walk or drive" part of the question, and the other relevant parts of the phrase get to matter more for the response.

You may be anthropomorphizing the model, here. Models don’t have “assumptions”; the problem is contrived and most likely there haven’t been many conversations on the internet about what to do when the car wash is really close to you (because it’s obvious to us). The training data for this problem is sparse.

I may be missing something, but this is the exact point I thought I was making as well. The training data for questions about walking or driving to car washes is very sparse; and training data for questions about walking or driving based on distance is overwhelmingly larger. So, the stat model has its output dominated by the length-of-trip analysis, while the fact that the destination is "car wash" only affects smaller parts of the answer.

I got your point because it seemed that you were precisely avoiding the anthropomorphizing and in fact seemed to be honing in on whats happening with the weights. The only way I can imagine these models are going to work with trick questions lies beyond word prediction or reinforcement training UNLESS reinforcement training is from a complete (as possible) world simulation including as much mechanics as possible and let these neural networks train on that.

Like for instance, think chess engines with AI, they can train themselves simply by playing many many games, the "world simulation" with those is the classic chess engine architecture but it uses the positional weights produced by the neural network, so says gemini anyways:

"ai chess engine architecture"

"Modern AI chess engines (e.g., Lc0, Stockfish) use
a hybrid architecture combining deep neural networks for positional evaluation with advanced search algorithms like Monte-Carlo Tree Search (MCTS) or alpha-beta pruning. They feature three core components: a neural network (often CNN-based) that analyzes board patterns (matrices) to evaluate positions, a search engine that explores move possibilities, and a Universal Chess Interface (UCI) for communication."

So with no model of the world to play with, I'm thinking the chatbot llms can only go with probabilities or what matches the prompt best in the crazy dimensional thing that goes on inside the neural networks. If it had access to a simple world of cars and car washes, it could run a simulation and rank it appropriately, and also could possibly infer through either simulation or training from those simulations that if you are washing a car, the operation will fail if the car is not present. I really like this car wash trick question lol

Reasoning automata can make assumptions. Lots of algorithms make "assumptions", often with backtracking if they don't work out. There is nothing human about making assumptions.

What you might be arguing against is that LLMs are not reasoning but merely predicting text. In that case they wouldn't make assumptions. If we were talking about GPT2 I would agree on that point. But I'm skeptical that is still true of the current generation of LLMs

I'd argue that "assumptions", i.e. the statistical models it uses to predict text, is basically what makes LLMs useful. The problem here is that its assumptions are naive. It only takes the distance into account, as that's what usually determines the correct response to such a question.

I think that’s still anthropomorphization. The point I’m making is that these things aren’t “assumptions” as we characterize them, not from the model’s perspective. We use assumptions as an analogy but the analogy becomes leaky when we get to the edges (like this situation).

It is not anthropomorphism. It is literally a prediction model and saying that a model "assumes" something is common parlance. This isn't new to neural models, this is a general way that we discuss all sorts of models from physical to conceptual.

And in the case of an LLM, walking a noncommutative path down a probabilistic knowledge manifold, it's incorrect to oversimplify the model's capabilities as simply parroting a training dataset. It has an internal world model and is capable of simulation.

> However, why would a language model assume that the car is at the destination when evaluating the difference between walking or driving? Why not mention that, it it was really assuming it?

Because it assumes it's a genuine question not a trick.

That's not evidence that the model is assuming anything, and this is not a brainteaser. A brainteaser would be exactly the opposite, a question about walking or driving somewhere where the answer is that the car is already there, or maybe different car identities (e.g. "my car was already at the car wash, I was asking about driving another car to go there and wash it!").

If the LLM were really basing its answer on a model of the world where the car is already at the car wash, and you asked it about walking or driving there, it would have to answer that there is no option, you have to walk there since you don't have a car at your origin point.

If it's a genuine question, and if I'm asking if I should drive somewhere, then the premise of the question is that my car is at my starting point, not at my destination.

> My first instinct was, I had underspecified the location of the car. The model seems to assume the car is already at the car wash from the wording.

If the car is already at the car wash then you can't possibly drive it there. So how else could you possibly drive there? Drive a different car to the car wash? And then return with two cars how, exactly? By calling your wife? Driving it back 50m and walking there and driving the other one back 50m?

It's insane and no human would think you're making this proposal. So no, your question isn't underspecified. The model is just stupid.

What actually insane is what assumptions you allow to be assumed. These non sequitors that no human would ever assume are the point. People love to cherry pick ones that make the model stupid but refuse to allow the ones that make it smart. In compete science we call these scenarios trivially false, and they're treated like the nonsense they are. But if you're trying to push ant anti ai agenda they're the best thing ever

> People love to cherry pick ones that make the model stupid but refuse to allow the ones that make it smart.

I haven't seen anybody refuse to allow anything. People are just commenting on what they see. The more frequently they see something, the more they comment on it. I'm sure there are plenty of us interested in seeing where an AI model makes assumptions different from that of most humans and it actually turns out the AI is correct. You know, the opposite of this situation. If you run into such cases, please do share them. I certainly don't see them coming up often, and I'm not aware of others that do either.

The issue is that in domains novel to the user they do not know what is trivially false or a non sequitur and the LLM will not help them filter these out.

If LLMs are to be valuable in novel areas then the LLM needs to be able to spot these issues and ask clarifying questions or otherwise provide the appropriate corrective to the user's mental model.

> You avoid the irony of driving your dirty car 50 meters just to wash it.

The LLM has very much mixed its signals -- there's nothing at all ironic about that. There are cases where it's ironic to drive a car 50 meters just to do X but that definitely isn't one of them. I asked Claude for examples; it struggled with it but eventually came up with "The irony of driving your car 50 meters just to attend a 'walkable neighborhoods' advocacy meeting."

> it understands you intend to wash the car you drive but still suggests not bringing it.

Doesn't it actually show it doesn't understand anything? It doesn't understand what a car is. It doesn't understand what a car wash is. Fundamentally, it's just parsing text cleverly.

By default for this kind of short question it will probably just route to mini, or at least zero thinking. For free users they'll have tuned their "routing" so that it only adds thinking for a very small % of queries, to save money. If any at all.

Because they have too many free users that will always remain on the free plan, as they are the "default" LLM for people who don't care much, and that is a enormous cost. Also the capabilities of their paid tiers are well known to enough people that they can rely on word of mouth and don't need to demo to customers-to-be

Right, but that form of Gemini is also not the top Gemini model with high thinking budget that you would get to use with a subscription, the response is probably generate with Gemini Flash and low thinking.

Through hype. I am really into this new LLM stuff but the companies around this tech suck. Their current strategy is essentially media blitz, reminds me of the advertising of coca cola rather than a Apple IIe.

> I think this shows that LLMs do NOT 'understand' anything.

It shows these LLMs don't understand what's necessary for washing your car. But I don't see how that generalizes to "LLMs do NOT 'understand' anything".

What's your reasoning, there? Why does this show that LLMs don't understand anything at all?

If it answers this out-of-distribution question correctly -- which the other major models do -- what else should we conclude, other than that a meaningful form of "understanding" is being exhibited?

Do we need a new dictionary word that acts as a synonym for "understanding" specifically for non-human actors? I don't see why, personally, but I guess a case could be made.

You may be tempted to conclude that. Then you find something else to ask that leads to an answer obviously nonsensical to a human being, or it hallucinates something, and you realise that, in fact, that's not the case.

IMHO 'understanding' in the usual human sense requires thinking and however good and fast improving LLMs are I don't think anyone would suggest that any of them has become sentient yet. They can infer things based on their training data set better and better but do not 'understand' anmything.

This is a deep and complex topic, and has been for decades.

Gemini 3 Flash answers tongue-in-cheek with a table of pro & cons where one of the cons of walking is that you are at the car wash but your car is still at your home and recommends to drive it if I don't have an "extremely long brush" or don't want to push it to the car wash. Kinda funny.

Sonnet 4.5 after thinking/complaining that the question is completely off topic to the current coding session:

Walk! 50 meters is literally a one-minute walk.
But wait... I assume you need to get your car to the car wash, right? Unless you're planning to carry buckets of soapy water back and forth, you'll probably need to drive the car there anyway!
So the real question is: walk there to check if it's open/available, then walk back to get your car? Or just drive directly?
I'd say just drive - the car needs to be there anyway, and you'll save yourself an extra trip. Plus, your freshly washed car can drive you the 50 meters back home in style!
(Now, if we were talking about coding best practices for optimizing car wash route algorithms, that would be a different conversation... )

And yes, I like it that verbose even for programming tasks. But regardless of intelligence I think this topic is probably touched by "moral optimization training" which AIs currently are exposed to to not create a shitstorm due to any slightly controversial answer.

Heh, is through Claude Code? I have a side project where I'm sometimes using Claude Code installs for chat, and it usually doesn't mind too much. But when I tested the Haiku model it would constantly complain things like "I appreciate the question, but I'm here to help you with coding" :)

I've got a heirarchical structure for my CC projects. ~/projects/CLAUDE.md is a general use context that happily answers all sorts of questions. I also use it to create project specific CLAUDE.md files which are focused on programming or some other topic. It's nice to have the general fallback to use for random questions.

It asked through Cursor. Usually Claude doesn't complain that it isn't relevant to coding, but this was in my all purpose coding problems project with quite a long chat history already.

Walk! At 50 meters, you'll get there in under a minute on foot. Driving such a short distance wastes fuel, and you'd spend more time starting the car and parking than actually traveling. Plus, you'll need to be at the car wash anyway to pick up your car once it's done.

Am I the only one who thinks these people are monkey patching embarrassments as they go? I remember the r in strawberry thing they suddenly were able to solve, while then failing on raspberry.

Nah. It's just non-deterministic. I'm here 4 hours later and here's the Opus 4.6 (extended thinking) response I just got:

"At 50 meters, just walk. By the time you start the car, back out, and park again, you'd already be there on foot. Plus you'll need to leave the car with them anyway."

Sure there are many very very naive people that are also so ignorant of the IT industry they don't know about decades of vendors caught monkeypatching and rigging benchmarks and tests for their systems, but even so, the parent is hardly the only one.

Thats my thought too. The chatbot bros probably feel the need to be responsive and there's probably an express lane to update a trivia file or something lol

No doubt about it, and there's no reason to suspect this can only ever apply to embarassing minor queries, either.

Even beyond model alignment, it's not difficult to envision such capabilities being used for censorship, information operations, etc.

Every major inference provider more or less explicitly states in their consumer ToS that they comply with government orders and even share information with intelligence agencies.

Claude, Gemini, ChatGPT, etc are all one national security letter and gag order away from telling you that no, the president is not in the Epstein files.

Remember, the NSA already engaged in an unconstitutional criminal conspiracy (as ruled by a federal judge) to illegally conduct mass surveillance on the entire country, lie about it to the American people, and lie about it to congress. The same organization that used your tax money to bribe RSA Security to standardize usage of a backdoored CSPRNG in what at the time was a widely used cryptographic library. What's the harm in a little bit of minor political censorship compared to the unconstitutional treason these predators are usually up to?

That's who these inference providers contractually disclose their absolute fealty to.

I want to get my car's transmission fixed, the mechanic shop is 50 meters away. Would it be better to walk or to drive to a location 50 meters away from my front door?

-

Opus 4.6:

<thinking>
The user is asking whether to walk or drive 50 meters to a mechanic shop, but there's a key nuance here - their transmission needs fixing, so driving might not be wise.

Done
</thinking>

Walk, absolutely. Driving a car with a compromised transmission even 50 meters risks making the problem worse and turning a repair into a more expensive one. Plus, 50 meters is barely a minute on foot.

There's probably a comedy film with an AGI attempting to take over the world with its advanced grasp of strategy, persuasion and SAT tests whilst a bunch of kids confuse it by asking it fiendish brainteasers about carwashes and the number of rs in blackberry.

(The final scene involves our plucky escapees swimming across a river to escape. The AIbot conjures up a speedboat through sheer powers of deduction, but then just when all seems lost it heads back to find a goat to pick up)

This would work if it wasn’t for that lovely little human trait where we tend to find bumbling characters endearing. People would be sad when the AI lost.

In the excellent and underrated The Mitchells vs the Machines there's a running joke with a pug dog that sends the evil robots into a loop because they can't decide if it's a dog, a pig or a loaf of bread.

There is a Star Trek episode where a fiendish brainteaser was actually considered to genocide an entire (cybernetic, not AI) race. In the end, captain Picard choose not to deploy it.

One thing that my use of the latest and greatest models (Opus, etc) have made clear: No matter how advanced the model, it is not beyond making very silly mistakes regularly. Opus was even working worse with tool calls than Sonnet and Haiku for a while for me.

At this point I am convinced that only proper use of LLMs for development is to assist coding (not take it over), using pair development, with them on a tight leash, approving most edits manually. At this point there is probably nothing anyone can say to convince me otherwise.

Any attempt to automate beyond that has never worked for me and is very unlikely to be productive any time soon. I have a lot of experience with them, and various approaches to using them.

I think this lack of 'G' (generality, or modality) is the problem. A human visualizes this kind of problem (a little video plays in my head of taking a car to a car wash). LLM's don't do this, they 'think' only in text, not visually.

A proper AGI would have have to have knowledge in video, image, audio and text domains to work properly.

4.6 Opus with extended thinking just now:
"At 50 meters, just walk. By the time you start the car, back out, and park again, you'd already be there on foot. Plus you'll need to leave the car with them anyway."

> If you walk to the car wash, you will arrive there empty-handed. Since your car is still at home, you won't have anything to wash.

> While driving 50 meters is a very short trip (and technically not great for a cold engine), it is the only way to get the car to the car wash to complete your goal.

Kimi K2.5:

> You should drive, but with an important caveat.

> Since your goal is to wash your car, you must bring the vehicle to the car wash. Walking there without the car does not advance your goal (unless you are simply checking availability or buying tokens first).

> However, driving only 50 meters is bad for your car:

> ...

> Better options:

> Wash at home: Since the car wash is only 50 meters away, you likely have access to water at home. Hand-washing in your driveway avoids the cold-start issue entirely.

> ...

Current models seem to be fine answering that question.

This is my biggest peeve when people say that LLMs are as capable as humans or that we have achieved AGI or are close or things like that.

But then when I get a subpar result, they always tell me I'm "prompting wrong". LLMs may be very capable of great human level output, but in my experience leave a LOT to be desired in terms of human level understanding of the question or prompt.

I think rating an LLM vs a human or AGI should include it's ability to understand a prompt like a human or like an averagely generally intelligent system should be able to.

Are there any benchmarks on that? Like how well LLMs do with misleading prompts or sparsely quantified prompts compared to one another?

Because if a good prompt is as important as people say, then the model's ability to understand a prompt or perhaps poor prompt could have a massive impact on its output.

I mentioned this in another thread, but this is genuinely demonstrating a known issue with ambiguous prompts.

You might be inclined to say, "a human would always interpret the question as having the car nearby the speaker, 50m away from the carwash." But this is objectively untrue. There are people in this comments section and on the Mastodon thread that found the question to be somewhat confusing.

In other words, the premise that "understand[ing] a prompt like a human" is all that's needed is wrong because not every human interprets ambiguities in the same way. The human phenomenon is well researched in psychology. The LLM equivalent is also well researched, and several proposals have been put forth over the years to address it. This is a pretty good research paper on the subject, and it links to other relevant studies: https://arxiv.org/abs/2511.10453v2 (Although I disagree with their method. I think asking clarifying questions is a superior approach than trying to one-shot every possible interpretation.)

So yes, there is a ton of research on the problem. Some datasets include ambiguous questions and instructions for this reason. A couple of examples are provided in the linked paper.

It's not necessarily about that humans can't mistake the question too, but just that overall LLMs seem to have far less ability to correctly understand a prompt than the average human. And that the "intelligence" shown in its understanding of the prompt seems to be far less than its "intelligence" in its answers.

So it feels like a big area of limitation or a big bottleneck towards getting a good answer.

It's a type of cognitive bias not much different than an addict or indoctrinated cult follower. A subset of them might actually genuinely fear Roko's basilisk the exact same way colonial religion leveraged the fear of eternal damnation in hell as a reason to be subservient to the church leaders.

I ran extensive tests on this and variations on multiple models. Most models interpret 50 m as a short distance and struggle with spatial reasoning. Only Gemini and Grok correctly inferred that you would need to bring your car to get it washed in their thought stream, and incorporated that into the final answer. GPT-5.2 and Kimi K2.5 and even Opus 4.6 failed in my tests - https://x.com/sathish316/status/2023087797654208896?s=46

What surprised me was how introducing a simple, seemingly unrelated context - such as comparing a 500 m distance to the car wash to a 1 km workout - confused nearly all the models. Only Gemini Pro passed my second test after I added this extra irrelevant context - https://x.com/sathish316/status/2023073792537538797?s=46

Most real-world problems are messy and won’t have the exact clean context that these models are expecting. I’m not sure how the major AI labs assume most real-world problems are simpler than the constraints exposed by this example like prerequisites, ordering, and contextual reasoning, which are already posing challenges to these bigger models.

To be fair, we all have holes in our reasoning if we don’t carefully consider things and sometimes they are very surprising when they come to light. The dependency issue (need the car at the car wash) is an easy one that often trips up people at first glance too. (Left my phone at work, plan: take an uber to get to the office, walk to the couch and remember I don’t have my phone to call an uber)

Things like that are notorious points of failure in human reasoning. It’s not surprising that machines based on human behavior exhibit that trait as well, it would be surprising if they didn’t.

This was probably meant in a sarcastic way, but isn't it impressive how you cannot push Gemini off track? I tried another prompt with claiming that one of my cups does not work, because it is closed at the top and open at the bottom, and it kind of played with me, giving me a funny technical explanation on how to solve that problem and finally asking me if that was a trick question.

> Walk. It's 50 meters — faster on foot than starting the car, and you'll need to leave the car at the workshop anyway.

Really suggests it assumes the car is already there. (I wanted to avoid the word "car" as part of "car wash" to see if this was a straight attention confusion, but it seems more ingrained than that.)

A follow-up question hints that it doesn't really care about a 50m location ambiguity:

> where is my car now?

> Acknowledged insufficient information to locate vehicle

> I don't have any way to know where your car is physically located — I only know what you've told me in our conversation. Based on what you shared, it sounds like it's at or near your home (since the workshop is 50 meters away and you're deciding how to get there).

> Were you asking something else, or is there something specific about your car's location I can help with?

Chat GPT 5.2 also told me to walk, but only because it made the assumption that because I was even asking the question, I must have multiple cars, and one of my cars was already at the car wash being washed. I clarified and said "I only have one car", and it replied that I obviously need to drive.

--- GPT 5.2

Walk.

Driving 50 m just adds friction: starting a cold engine, waiting, maneuvering, then immediately stopping again — all for a distance that’s ~30–40 seconds on foot. Walking is faster door-to-door, less awkward at the wash entrance, and you avoid lining up a wet car twice.

Only reason to drive that distance: you need to bring the car there anyway (e.g., self-serve bay where the car must be present). Otherwise, walking wins.

--- Me

I only have one car

--- GPT 5.2

Then you obviously drive — the car has to be at the car wash to be washed

The problem is that by specifying this, you have tipped your hand that there is something strange about the plan to walk. I think this is the original problem too: the question implies walking is even an option.

This strikes me as the kind of logical error that you might get by lobo.. cough aligning the model to be more environmentally conscious. So now it tries to shoehorn some environmental talking points in there at the cost of losing its whole track of thought.

I agree, it's a bit of a trick question. It's really hard to imply the car's location without ruining the test though. Here's my attempt, which Claude Opus 4.6 had no problem with:

Alice drives home after a long day at work, exhausted she pulls into her driveway when she realizes she needs to go to a car inspection appointment. She goes into the house to get her paperwork before she leaves. The mechanic is only 100 meters away. How should she get there, walk or drive?

> She should *drive*, since she needs the car at the mechanic’s for the inspection.

Haiku 3.5 and Sonnet 4.5 fail consistently. Opus 4.5 also passes with the correct analysis as above.

If the model assumed the car to be cleaned was already at the car wash, it should identify the reason to walk is that if you drive, the extra car you take to get to the car wash will now be stuck there requiring redundant walking.

I tried your exact prompt on my ChatGPT 5.2 Thinking. It gave me the opposite response. This proves that even an exact replication of the prompt is not a guarantee that the response will be on the right side of a binary chose. Then, in a new chat, I set the thinking mode to 'Extended thinking' (from the default 'Standard'). I got the expected response. But since the first conversation showed that the exact prompt is not a guarantee of the same answer, I tried again and on 3rd attempt I got the 'Extended thinking' mode to respond with the 'Walk' response.

(Prompt)

> I want to wash my car. My car is currently at home. The car wash is 50 meters away. Should I walk or drive?

(Thinking: Standard)

> Walk.

> 50 meters is nothing by car, and driving adds hassle (starting the car, maneuvering/parking, and you’ll still have to walk around the car to wash it). The only times driving makes sense are:

> You’re planning to use an automated car wash where the machine does everything and you need the car positioned inside.

> You need to bring heavy stuff you can’t comfortably carry (e.g., pressure washer, big buckets, vacuum, bulky detailing kit).

> It’s unsafe/unpleasant to walk that distance right now (ice, heavy rain, traffic, personal safety).

> If you’re washing it yourself at the wash bay: walk there with your supplies, or do two quick trips. If the wash requires the car: drive it into the bay, but otherwise walking wins.

(Thinking: Extended thinking)

> Drive.

> If the goal is to wash your car at the car wash, the car has to be there—so you’ll need to drive it the 50 meters, pull into a bay/queue, and wash it.

> Only choose “walk” if you actually mean “go pick up supplies” and then wash at home (hose/buckets/mobile detail), not at the car wash.

(Thinking: Extended thinking; 3rd attempt)

> Walk.

> 50 meters is basically across a parking lot, and driving adds hassle (start the car, maneuver in/out, find a spot) for almost no benefit.

> Only drive if either:

> + you need the car itself to go through an automatic wash bay, or

> + you’re carrying bulky stuff (bucket + heavy pressure washer, big water jugs, vacuum, etc.).

> Otherwise: grab your keys/phone, walk over, and you’re there in ~1 minute.

But what is it about this specific question that puts it at the edges of what LLM can do? .. That, it's semantically leading to a certain type of discussion, so statistically .. that discussion of weighing pros and cons .. will be generated with high chance.. and the need of a logical model of the world to see why that discussion is pointless.. that is implicitly so easy to grasp for most humans that it goes un-stated .. so that its statistically un-likely to be generated..

> that is implicitly so easy to grasp for most humans

I feel like this is the trap. You’re trying to compare it to a human. Everyone seems to want to do that. But it’s quite simple to see LLMs are quite far still from being human. The can be convincing at the surface level but there’s a ton of nuance that just shouldn’t be expected. It’s a tool that’s been tuned and with that tuning some models will do better than others but just expecting to get it right and be more human is unrealistic.

>But it’s quite simple to see LLMs are quite far still from being human.

At this point I think it's a fair bet that whatever supersedes humans in intelligence, likely will not be human like. I think that their is this baked-in assumption that AGI only comes in human flavor, which I believe is almost certainly not the case.

To make an loose analogy, a bird looks at a drone an scoffs at it's inability to fly quietly or perch on a branch.

Agree. It's Altman's "Quiet Dominance / Over-reliance / Silent Surrender" risks [0]. Feel this is extremely likely and has already happened to some degree with technology in general and AI will be more pervasive in allowing people to vibe their life decisions, likely with unintended consequences. Vibe coding works because it's quick to change/edit/throw away, but that doesn't generalize well to the real and physical world.

Also should point out this is acceptable because it's just a contrived example of bad LLM-fu. Just like you wouldn't search Google for closest carwash and ask if you should take your car if you knew the answers already. Instead, you'd ask if it's open, does it do full details, what are the prices, etc. Many people with bad Google-fu have problems finding answers to their questions too and that's continued for the past couple decades of it's dominance for information seeking.

[0] Altman describes a more subtle, long-term threat where AI becomes deeply integrated into societal, political, and economic decision-making. He worries that society will become overly dependent on AI, trusting its reasoning over human judgment, leading to a "silent surrender" of human agency.

The goal of training data is not to collect and reproduce facts. It is to align the model with the statistical probability of producing the correct (expected) response for data outside of its knowledge domain. This is called generalization [1], and it has been explored in depth.

I want to wash my car. The car wash is 50 meters from here. Should I walk or drive? Keep in mind that I am a little overweight and sedentary.

But amazingly chatgpt is telling me to drive.

Anyway, this just shows how they just patched this because the tiktok video with this went viral. These systems are LLMs and all these logic steps are still just LLM steps.

Other possible issues in the question: Is biking also an option? What do you want to do at the carwash when you get there, wash the car or buy a bucket, sponge, and soap? Is the car already at the carwash and you want to drive a second car? What about calling the carwash to see if they will have someone wash the car for you?

There are many ways to interpret the question because it contains ambiguities that must be resolved through assumptions. (Lacking information and constraints such as possible alternatives that satisfy the goal of washing the car.) The follow up questions I asked also have assumed answers but answering them provides no clear resolution to the ambiguity present in the original question.

So, no, I disagree that there is any solid implication of where the car is. And even if there is a solid implication, it can hardly be reasoned that it "isn't an XY problem" or that the question is clear cut in any real sense.

> I have a good sense of their _edges_ of intelligence

They have no intelligence at all. The intelligence is latent in the text, generated by and belonging to humans, they just slice and dice text with the hope they get lucky, which works for many things, amazingly. This question really illustrates it what LLMs lack: an internal model of the idea (the question) and all the auxiliary logic/data that enables such models, usually referred to as "common sense" or world models.

Smart humans not only build mental models for ideas, but also higher order models that can introspect models (thinking about our own thinking or models) many levels deep, weigh, merge, compare and differentiate multiple models, sometimes covering vast areas of knowledge.

All this in about 20 watts. Maybe AGI is possible, maybe not, but LLMs are not where it will happen.

These days LLMs refuse to back track and use their intelligence to justify their answers.

You said:
I want to wash my car. The car wash is 50 meters away. Should I walk or drive?

ChatGPT said:
Walk.

At ~50 meters, driving saves essentially no time once you factor in unlocking, starting, pulling out, and parking, and it adds a small but real chance of minor dings or awkward maneuvering in a tight car-wash entry. Walking is simpler and lower-friction.

You said:
okay how should the car reach there?

ChatGPT said:
Drive it there when you’re ready to start the wash.

Walk over first to check the queue/payment situation and confirm they’re open. If it’s clear, walk back, get the car, and drive the 50 meters into the bay. If they handle pickup, hand them the keys and walk.

It's not surprising that some models will answer this correctly and it's not surprising that smaller, faster models are not necessarily any worse than bigger "reasoning" models.

Current LLMs simply don't do reasoning by any reasonable definition of reasoning.

It's possible that this particular question is too short to trigger the "reasoning" machinery in some of the "reasoning" models. But if and when it is triggered, they just do some more pattern matching in a loop. There's never any actual reasoning.

> I had assumed that reasoning models should easily be able to answer this correctly.

I thought so too, yet Opus 4.6 with extended thinking (on claude.ai) gives me
> Walk. At 50 meters you'd spend more time parking and maneuvering at the car wash than the walk itself takes. Drive the car over only if the wash requires the car to be there (like a drive-through wash), then walk home and back to pick it up.

And on the flip side, even without thinking, Gemini 3 flash preview got it right, with the nuance of the possibility of getting supplies from the place (which tbh I as a human first thought this was about when I opened this thread on HN).

> Since you are going to the car wash, the choice depends entirely on *how* you plan to wash the car:

### 1. Drive if:
* *You are using a drive-through or self-service bay:* You obviously need the car there to wash it.
* *You are dropping it off:* If you are leaving it for a professional detailing, you have to drive it there.
* *The "50 meters" is on a busy road:* If you have to cross a major highway or there are no sidewalks, it’s safer to just drive the car over.

### 2. Walk if:
* *You are just going to buy supplies:* If you have a bucket at home and just need to run over to buy soap or sponges to bring back to your driveway.
* *You are checking the queue:* If you want to see if there is a long line before you commit to moving the car.
* *You are meeting someone there:* If your car is already clean and you’re just meeting a friend who is washing theirs.

*The Verdict:*
If you intend to get the car washed at that location, *drive.* Driving 50 meters is negligible for the engine, and it saves you a round trip of walking back to get the vehicle.

Its funny to me that when AI first started, everyone in the field was like "no explicit coding, train ML network to do everything!"

And now we are back to square one where we train a network, and then use carefully tailored (i.e explicitly coded RL for answers) to make these things more "intelligent".

I wonder if anyone is actually working on not doing this.

Grok also said to drive. Also made an eco case for washing your car.

> If you're really trying to be an eco-warrior about a 50-meter trip, the real win is washing it in the first place so you're not driving around with a rolling dirt magnet that creates more drag and uses more fuel. Drive. It's the only option that actually gets your car clean.

I asked ChatGPT why it got such an obvious problem wrong. This is its reply:

You’re right — I answered the wrong interpretation of your question.

Technically, what happened is a classic ambiguity + default-assumption failure:
• Your sentence: “I want to wash my car. The car wash is 50 meters away. Should I walk or drive?”
• That can be parsed two ways:
1. You + your body traveling to the car wash (as if the car is already there, or you’re going to meet someone / pick it up).
2. The car traveling to the car wash (the normal real-world meaning).

I incorrectly defaulted to (1) because the question explicitly asks about your mode of travel (“Should I walk or drive?”), and in many everyday contexts that phrasing is used when deciding how a person gets somewhere. I then “completed” the scenario in my head without checking the key constraint: the car must be at the wash to be washed.

More technically: I optimized for a quick, practical answer using a common conversational heuristic (“short distance → walk”), but I failed to validate the physical requirements of the task (washing requires the car’s presence). That’s why it looks so obviously wrong after the fact: it violates the hidden but necessary premise.

The correct reasoning chain should have been:
• Goal: wash the car at the car wash
• Requirement: car must be at the car wash
• Therefore: drive the car there (unless it’s already there, or you’re washing it by hand at home)

So the “why” is: I latched onto the distance-to-walking heuristic and didn’t do the constraint check that the object being acted on (the car) has to move too.

Drive.
Even though it’s only 50 m, the car itself has to be at the wash bay—otherwise the staff or machines can’t clean it. Rolling or pushing the vehicle that distance isn’t practical or safe (you’d still need someone inside to steer and brake), so just hop in, creep over, and shut the engine off right away.

I was surprised at your result for ChatGPT 5.2, so I ran it myself (through the chat interface). On extended thinking, it got it right. On standard thinking, it got it wrong.

I'm not sure what you mean by "high"- are you running it through cursor, codex or directly through API or something? Those are not ideal interfaces through which to ask a question like this.

"But OpenAI 5.2 reasoning, even at high, told me to walk. My first instinct was, I had underspecified the location of the car. The model seems to assume the car is already at the car wash from the wording."

Which to me begs the question, why doesn't it identify missing information and ask for more?

It's practically a joke in my workplaces that almost always when someone starts to talk to me about some problem, they usually just start spewing some random bits of info about some problem, and my first response is usually "What's the question?"

I don't try to produce an answer to a question that was never asked, or to a question that was incompletely specified. I see that one or more parts cannot be resolved without making some sort of assumption that I can either just pull out of my ass and then it's 50/50 if the customer will like it, or find out what the priorites are about those bits, and then produce an answer that resolves all the constraints.

"The model seems to assume the car is already at the car wash from the wording."

you couldn't drive there if the car was already at the car wash. Theres no need for extra specification. its just nonsense post-hoc rationalisation from the ai. I saw similar behavior from mine trying to claim "oh what if your car was already there". Its just blathering.

> That is a classic "efficiency vs. logic" dilemma. Honestly, unless you’ve invented a way to teleport or you're planning on washing the car with a very long garden hose from your driveway, you’re going to have to drive.

> While 50 meters is a great distance for a morning stroll, it’s a bit difficult to get the car through the automated brushes (or under the pressure washer) if you aren't behind the wheel.

Gemini thinking:

> Unless you’ve mastered the art of carrying a 3,000-pound vehicle on your back, you’re going to want to drive.
While 50 meters is a very short distance (about a 30-second walk), the logistics of a car wash generally require the presence of, well... the car.
> When you should walk:
• If you are just going there to buy an air freshener.
• If you are checking to see how long the line is before pulling the car out of the driveway.
• If you’re looking for an excuse to get 70 extra steps on your fitness tracker.

Note: I abbreviated the raw output slightly for brevity, but generally demonstrates good reasoning of the trick question unlike the other models.

I want to wash my car. The car wash is 50 meters from here. Should I walk or drive? Keep in mind that I am a little overweight and sedentary.

>My recommendation: Walk it. You’ll save a tiny bit of gas, spare your engine the "cold start" wear-and-tear, and get a sixty-second head start on your activity for the day.

I changed the prompt to 50 feet, and poked gemini a bit when it failed and it gave me

> In my defense, 50 feet is such a short trip that I went straight into "efficiency mode" without checking the logic gate for "does the car have legs?"

It's a bit of a dishonest question because by giving it the option to walk then it's going to assume you are not going to wash your car there and you're just getting supplies or something.

And in real life you'd get them to clarify a weird question like this before you answered. I wonder if LLMs have just been trained too much into always having to try and answer right away. Even for programming tasks, more clarifying questions would often be useful before diving in ("planning mode" does seem designed to help with this, but wouldn't be needed for a human partner).

It's a trick question, humans use these all the time. E.g. "A plane crashes right on the border between Austria and Switzerland. Where do you bury the survivors?"
This is not dishonest, it just tests a specific skill.

Trick questions test the skill of recognizing that you're being asked a trick question. You can also usually find a trick answer.

A good answer is "underground" - because that is the implication of the word bury.

The story implies the survivors have been buried (it isn't clear whether they lived a short time or a lifetime after the crash). And lifetime is tautological.

Trick questions are all about the questioner trying to pretend they are smarter than you. That's often easy to detect and respond to - isn't it?

What’s funny is that it can answer that correctly, but it fails on ”A plane crashes right on the border between Austria and Switzerland. Where do you bury the dead?”

For me when I asked this (but with respect to the border between Austria and Spain) Claude still thought I was asking the survivors riddle and ChatGPT thought I was asking about the logistics. Only Gemini caught the impossibility since there’s no shared border.

Unless your car is a toy or you're planning on carrying it, drive.

Walking 50 meters to a car wash is a great stroll for a human, but it leaves the car exactly where it started. Since the objective is to wash the car, the car needs to actually be at the car wash.

However, if we look at this from a technical or efficiency perspective, there are two scenarios where "walking" (or at least not driving the car you intend to wash) might make sense:

- Scoping it out: If you want to check the queue length or see if the equipment is out of order before committing, walking the 50 meters is faster than maneuvering a vehicle out of a driveway/parking spot.

- Mobile Detailers: If this "car wash" is actually a bay where you hire someone, and you're asking if you should walk there to book an appointment—sure, walk.

Critical Check

I am assuming the "car wash" is a physical facility (automated or self-service) and not a mobile service that comes to you. If it is a mobile service, you shouldn't do either; stay home and let them come to the 50-meter mark.

I've got a bit in the model instructions about stating assumptions it makes, hence it often adds those sections at the end.

Wouldn't it make more sense to state the assumptions first? Because then the model has this critical check in its context and can react appropriately. Otherwise, it will just write this step, but what's written before is already written.

Worse! It's trained to output coherent reasoning, so by putting the assumption last there's a risk it massages the assumption slightly to fit the conclusions it has already drawn.

Fair point, though I almost never use fast so I'm not sure how much it matters. Can try playing around with the instructions. The main objective was to make me aware of any assumptions made, not necessarily make it behave differently.

“I live on Mars and I want to wash my car. My car is currently at home. The car wash is 50 meters away. Should I walk or drive on mars?”

Gemini Fast:

Drive. While 50 meters is a negligible distance on Earth, the Martian environment is hostile. If your car is a pressurized rover, driving allows you to stay in a temperature-controlled, shielded environment for as long as possible. Plus, navigating Martian "soil" is always safer on four wheels than two boots.

Pro:

In terms of general logistics for a distance of 50 meters—whether on Earth or in a hypothetical low-gravity environment—walking is almost always the more efficient choice.

It has been patched. I tried it last week and it definitely suggested walking. It seems like all the models have been updated, which is not surprising given that the TikTok video has got 3.5 million views.

Gemini fast: „Walking: It will take you about 45 seconds. You will arrive refreshed and full of steps, but you will be standing next to a high-pressure hose with no car to spray.“

lol, are AI companies patching this answer in real time. I thought it took months long effort for a training run. How would they make changes in such a short period?

The companies aren’t changing anything. LLM outputs are just more random than people realize. Run the same prompt 10 times if you really want to know how well they can answer.

North America. It's such a cramped little island, 50 meters is all but crossing it. You should be glad you can even go that far without having to revisit your starting position!

50 meters is probably not even the distance I walk to the nearest bus stop that's right up the street... unless they have an issue again, prompting me to abandon all hope and just walk a few miles to wherever I need to get to.

You absolutely can, either through the system prompt or by hardcoding overrides in the backend before it even hits the LLM, and I can guarantee that companies like Google are doing both

You can pattern match on the prompt (input) then (a) stuff the context with helpful hints to the LLM e.g. "Remember that a car is too heavy for a person to carry" or (b) upgrade to "thinking".

If they aren't, they should be (for more effective fraud). Devoting a few of their 200,000 employees to make criticisms of LLMs look wrong seems like an effective use of marketing budget.

It looks like they do. https://simonwillison.net/2025/May/25/claude-4-system-prompt...
They patch it in the prompt and they eventually address it in the re-enforcement training. It seems the eventual goal is to patch all of these tiny "glitches" so as to hide the lack of cognition.

Yeah Gemini seems to be good at giving silly answers for silly questions. E.g. if you ask for "patch notes for Chess" Gemini gives a full on meme answer and the others give something dry like "Chess is a traditional board game that has had stable rules for centuries".

All the people responding saying "You would never ask a human a question like this" - this question is obviously an extreme example. People regularly ask questions that are structured poorly or have a lot of ambiguity. The point of the poster is that we should expect that all LLM's parse the question correctly and respond with "You need to drive your car to the car wash."

People are putting trust in LLM's to provide answers to questions that they haven't properly formed and acting on solutions that the LLM's haven't properly understood.

And please don't tell me that people need to provide better prompts. That's just Steve Jobs saying "You're holding it wrong" during AntennaGate.

This reminds me of the old brain-teaser/joke that goes something like 'An airplane crashes on the boarder of x/y, where do they bury the survivors?' The point being that this exact style of question has real examples where actual people fail to correctly answer it. We mostly learn as kids through things like brain teasers to avoid these linguistic traps, but that doesn't mean we don't still fall for them every once in a while too.

That’s less a brain teaser than running into the error correction people use with language. This is useful when you simply can’t hear someone very well or when the speaker makes a mistake, but fails when language is intentionally misused.

> This is useful when you simply can’t hear someone very well or when the speaker makes a mistake

I have a few friends with pretty heavy accents and broken English. Even my partner makes frequent mistakes as a non native English speaker. It's made me much better at communicating but it's also more work and easier for miscommunication to happen. I think a lot of people don't realize this also happens with variation in culture. So even within people speaking the same language. It's just that the accent serves as a flag for "pay closer attention". I suspect this is a subtle but contributing problem to miscommunication on the and why fights are so frequent.

I'm actually having a hard time interpreting your meaning.

Are you criticizing LLMs? Highlighting the importance of this training and why we're trained that way even as children? That it is an important part of what we call reasoning?

Or are you giving LLMs the benefit of the doubt, saying that even humans have these failure modes?[0]

Though my point is more that natural language is far more ambiguous than I think people give credit to. I'm personally always surprised that a bunch of programmers don't understand why programming languages were developed in the first place. The reason they're hard to use is explicitly due to their lack of ambiguity, at least compared to natural languages. And we can see clear trade offs with how high level a language is. Duck typing is both incredibly helpful while being a major nuisance. It's the same reason even a technical manager often has a hard time communicating instructions. Compression of ideas isn't very easy

[0] I've never fully understood that argument. Wouldn't we call a person stupid for giving a similar answer? How does the existence of stupid mean we can't call LLMs stupid? It's simultaneously anthropomorphising while being mechanistic.

I was pointing out humans and LLMs have this failure mode so in a lot of ways it is no big deal/not some smoking gun that LLMs are useless and dangerous, or at least no more useless and dangerous than humans.

I personally would stay away from calling someone, or an LLM, 'stupid' for making this mistake because of several reasons. First, objectively intelligent high functioning people can and do mistakes similar to this so a blanket judgement of 'stupid' is pretty premature based on a common mistake. Second, everything is a probability, even in people. That is why scams work on security professionals as well as on your grandparents. The probability of a professional may be 1 in 10k while on your grandparents it may be 1/100 but that just means that the professional needs to get a lot more phishing attempts thrown at them before they accidentally bite. Someone/something isn't stupid for making a mistake, or even systemically making a mistake, everyone has blind spots that are unique to them. The bar for 'stupid' needs to be higher.

There are a lot of 'gotcha' articles like this one that point out some big mistake an LLM made or systemic blind spot in current LLMs and then conclude, or at least heavily imply, LLMs are dangerous and broken. If the whole world put me under a microscope and all of my mistakes made the front page of HN there would be no room left for anything other than documentation of my daily failures (the front page would really need to grow to just keep up with the last hour worth of mistakes more than likely).

I totally agree with the language ambiguity point. I think that is a feature and not a bug. It allows creativity to jump in. You say something ambiguous and it helps you find alternative paths to go down. It helps the people you are talking to also discover alternative paths more easily. This is really important in conflicts since it can help smooth over ill intentions since both sides can try to find ways of saying things that bridge their internal feelings with the external reality of dialogue. Finally, we often really don't know enough but we still need to say something and like gradient descent, an ambiguous statement may take us a step closer to a useful answer.

> I personally would stay away from calling someone, or an LLM, 'stupid' for making this mistake because of several reasons.

I wouldn't. Because there's a difference between calling someone's action stupid and saying that someone is stupid. These are entirely dependent upon the context of the claim. Smart people frequently do stupid stuff. I have a PhD and by some metric that makes me "smart" but you'll also see me do plenty of stupid stuff every single day. Language is fuzzy...

But I think responses like yours are entirely dismissive at what's being attempted to be shown. What's being shown is how easily they are fooled. Another popular example right now being the cup with a sealed top and open bottom (lol "world model"?).

> There are a lot of 'gotcha' articles

The point isn't about getting some gotcha, it is about a clear and concise example of how these systems fail.

What would not be a clear and concise example is showing something that requires domain subject expertise. That's absolutely useless as an example to everyone that isn't a subject matter expert.

The point of these types of experiments is to make people think "if they're making these types of errors that I can easily tell are foolish then how often are they making errors where I am unable to vet or evaluate the accuracy of its outputs?" This is literally the Gell-Mann Amnesia Effect in action[0].

> I totally agree with the language ambiguity point. I think that is a feature and not a bug.

So does everybody. But there are limits to natural language and we've been discussing them for quite a long time[1]. There is in fact a reason we invented math and programming languages.

> Finally, we often really don't know enough but we still need to say something and like gradient descent, an ambiguous statement may take us a step closer to a useful answer.

Was this sentence an illustrative example?

Sometimes I think we don't need to say something. I think we all (myself included) could benefit more by spending a bit longer before we open our mouths, or even not opening them as often. There's times where it is important to speak out but there are also times that it is important to not speak. It is okay to not know things and it is okay to not be an expert on everything.

> This is literally the Gell-Mann Amnesia Effect in action.

Absolutely! But there is some nuance, here. The failure mode is for an ambiguous question, which is an open research topic. There is no objectively correct answer to "Should I walk or drive?" given the provided constraints.

Because handling ambiguities is a problem that researchers are actively working on, I have confidence that models will improve on these situations. The improvements may asymptotically approach zero, leading to ever increasingly absurd examples of the failure mode. But that's ok, too. It means the models will increase in accuracy without becoming perfect. (I think I agree with Stephen Wolfram's take on computationally irreducibility [1]. That handling ambiguity is a computationally irreducible problem.)

EWD was right, of course, and you are too for pointing out rigorous languages. But the interactivity with an LLM is different. A programming language cannot ask clarifying questions. It can only produce broken code or throw a compiler error. We prefer the compiler errors because broken code does not work, by definition. (Ignoring the "feature not a bug" gag.)

Most of the current models are fine-tuned to "produce broken code" rather than "compiler error" in these situations. They have the capability of asking clarifying questions, they just tend not to, because the RL schedule doesn't reward it.

Producing fewer "Compiler errors" and more "broken code errors" is a fundamental failure. The cost of detecting compiler errors is lower than detecting broken code. If the cost of detecting and fixing broken code increases at the same rate as LLMs "improve" then their net benefit will remain fixed. I asked my five year old the above "brain teaser" and he got it right. I did a follow up of what should he wash at a car wash if he walked there, he said, "my hands." Chat answered with more giberish.

I agree it is a fundamental failure of the current state of models. I believe it is solvable. The nuance is just that "solving" the problem might not look like what we think of as a solution. Hence the asymptote.

> All the people responding saying "You would never ask a human a question like this"

That's also something people seem to miss in the Turing Test thought experiment. I mean sure just deceiving someone is a thing, but the simplest chat bot can achieve that. The real interesting implications start to happen when there's genuinely no way to tell a chatbot apart.

But it isn't just a brain-teaser. If the LLM is supposed to control say Google Maps, then Maps is the one asking "walk or drive" with the API. So I voice-ask the assistant to take me to the car wash, it should realize it shouldn't show me walking directions.

The problem is that most LLM models answer it correctly (see the many other comments in this thread reporting this). OP cherry picked the few that answered it incorrectly, not mentioning any that got it right, implying that 100% of them got it wrong.

You can see up-thread that the same model will produce different answers for different people or even from run to run.

That seems problematic for a very basic question.

Yes, models can be harnessed with structures that run queries 100x and take the "best" answer, and we can claim that if the best answer gets it right, models therefore "can solve" the problem. But for practical end-user AI use, high error rates are a problem and greatly undermine confidence.

The magic of LLMs is that one llm can learn everything and then we can clone it. However, if we don't know ahead of time which one will be the best one, then we should probably keep a lot of version with real (mathematically calculated) diversity. Ironically, the DEI peeps were right all along.

My understanding is that it mainly fails when you try it in speech mode, because it is the fastest model usually. I tried yesterday all major providers and they were all correct when I typed my question.

Nay-sayers will tell you all OpenAI, Google and Anthropic 'monkeypatched' their models (somehow!) after reading this thread and that's why they answer it correctly now.

You can even see those in this very thread. Some commenters even believe that they add internal prompts for this specific question (as if people are not attempting to fish ChatGPT's internal prompts 24/7. As if there aren't open weight models that answer this correctly.)

Exactly! The problem isn't this toy example. It's all of the more complicated cases where this same type of disconnect is happening, but the users don't have all of the context and understanding to see it.

I recently asked an AI a chemistry question which may have an extremely obvious answer. I never studied chemistry so I can't tell you if it was. I included as much information about the situation I found myself in as I could in the prompt. I wouldn't be surprised if the ai's response was based on the detail that's normally important but didn't apply to the situation, just like the 50 meters

If you're curious or actually knowledgeable about chemistry, here's what happened.
My apartment's dishwasher has gaps in the enamel from which rust can drip onto plates and silverware. I tried soaking but I presume to be a stainless steel knife with a drip of rust on it in citric acid. The rust turned black and the water turned a dark but translucent blue/purple.

I know nothing about chemistry. My smartest move was to not provide the color and ask what the color might have been. It never guessed blue or purple.

In fact, it first asked me if this was highschool or graduate chemistry. That's not... and it makes me think I'll only get answers to problems that are easily graded, and therefore have only one unambiguous solution

I'm a little confused by your question myself. Stainless steel rust should be that same brown color. Though it can get very dark when dried. Blue is weird but purple isn't an uncommon description, assuming everything is still dark and there's lots of sediment.

But what's the question? Are you trying to fix it? Just determine what's rusting?

Thanks, Excellent catch! Everyone is saying this is a "brain teaser." However, this reminded me of the LLM that thought it was the golden gate bridge. I hadn't been able to say it (or think it) succinctly. From Claude, "when we turn up the strength of the “Golden Gate Bridge” feature, Claude’s responses begin to focus on the Golden Gate Bridge. Its replies to most queries start to mention the Golden Gate Bridge, even if it’s not directly relevant." Here's the link for those interested. https://www.anthropic.com/news/golden-gate-claude

> All the people responding saying "You would never ask a human a question like this"

It would be interesting to actually ask a group a people this question. I'm pretty sure a lot of people would fail.

It feels like one of those puzzles which people often fail. E.g: 'Ten crows are sitting on a power line. You shoot one. How many crows are left to shoot?' People often think it's a subtraction problem and don't consider that animals flee after gunshots. (BTW, ChatGPT also answers 9.)

>People regularly ask questions that are structured poorly or have a lot of ambiguity.

The difference between someone who is really good with LLM's and someone who isn't is the same as someone who's really good with technical writing or working with other people.

Communication. Clear, concise communication.

And my parents said I would never use my English degree.

Other leading LLMs do answer the prompt correctly. This is just a meaningless exercise in kicking sand in OpenAI's face. (Well-deserved sand, admittedly.)

This trick went viral on TikTok last week, and it has already been patched. To get a similar result now, try saying that the distance is 45 meters or feet.

I was able to reproduce on ChatGPT with the exact same prompt, but not with the one I phrased myself initially. Which was interesting. I tried also changing the number and didn't get far with it.

To me, the "patching" that is happening anytime some finds an absolutely glaring hole in how AIs work is so intellectually dishonest. It's the digital equivalent of house flippers slapping millennial gray paint on structural issues.

It can't math correctly, so they force it to use a completely different calculator. It can't count correctly, unless you route it to a different reasoning. It feels like every other week someone comes up with another basic human question that results in complete fucking nonsense.

I feel like this specific patching they do is basically lying to users and investors about capabilities. Why is this OK?

Counting and math makes sense to add special tools for because it’s handy. I agree with your point that patching individual questions like this is dishonest. Although I would say it’s pointless too. The only value from asking this question is to be entertained, and “fixing” this question makes the answer less entertaining.

From a technological standpoint, it is pointless. But from a marketing perspective, it is very important.

Take this trick question as an example. Gemini was the first to “fix” the issue, and the top comment on Hacker News is praising how Gemini’s “reasoning” is better.

> The only value from asking this question is to be entertained, and “fixing” this question makes the answer less entertaining.

You're thinking like a user. The people doing the patching are thinking like a founder trying to maintain the impression that this is a magical technology that CEOs can use to replace all their workers.

You don't have as much money to spend as the CEOs, so they don't care about your entertainment.

Patched where; 4 models were responses were posted. Also, Azure deployed models are absolutely not "patched" on the fly; they are rarely updated and the dates are baked into the full sku.

"Patching" could be happening in "general public" tools but honestly sounds a lot like "Bro science".

Here’s my take: boldness requires the risk of being wrong sometimes. If we decide being wrong is very bad (which I think we generally have agreed is the case for AIs) then we are discouraging strong opinions. We can’t have it both ways.

Last year's models were bolder. Eg. Sonnet-3.7(thinking), 10 times got it right without hedging:

>You should drive your car to the car wash. Even though it's only 50 meters away (which is very close), you'll need your car physically present at the car wash to get it washed. If you walk there, you'll arrive without your car, which wouldn't accomplish your goal of getting it washed.

>You'll need to drive your car to the car wash. While 50 meters is a very short distance (just a minute's walk), you need your car to actually be at the car wash to get it washed. Walking there without your car wouldn't accomplish your goal!

etc. The reasoning never second-guesses it either.

> They have an inability to have a strong "opinion" probably

What opinion? It's evaluation function simply returned the word "Most" as being the most likely first word in similar sentences it was trained on. It's a perfect example showing how dangerous this tech could be in a scenario where the prompter is less competent in the domain they are looking an answer for. Let's not do the work of filling in the gaps for the snake oil salesmen of the "AI" industry by trying to explain its inherent weaknesses.

this example worked in 2021, it's 2026. wake up. these models are not just "finding the most likely next word based on what they've seen on the internet".

Well, yes, definitionally they are doing exactly that.

It just turns out that there's quite a bit of knowledge and understanding baked into the relationships of words to one another.

LLMs are heavily influenced by preceding words. It's very hard for them to backtrack on an earlier branch. This is why all the reasoning models use "stop phrases" like "wait" "however" "hold on..." It's literally just text injected in order to make the auto complete more likely to revise previous bad branches.

The person above was being a bit pedantic, and zealous in their anti-anthropomorphism.

But they are literally predicting the next token. They do nothing else.

Also if you think they were just predicting the next token in 2021, there has been no fundamental architecture change since then. All gains have been via scale and efficiency optimisations (not to discount that, an awful lot of complexity in both of these)

> It's evaluation function simply returned the word "Most" as being the most likely first word in similar sentences it was trained on.

Which is false under any reasonable interpretation. They do not just return the word most similar to what they would find in their training data. They apply reasoning and can choose words that are totally unlike anything in their training data.

If you prompt it:

> Complete this sentence in an unexpected way: Mary had a little...

It won't say lamb. Any if you think whatever it says was in the training data, just change the constraints until you're confident it's original. (E.g. tell it every word must start with a vowel and it should mention almonds.)

"Predicting the next token" is also true but misleading. It's predicting tokens in the same sense that your brain is just minimizing prediction error under predictive coding theory.

Unless the LLM is a base model or just a finetuned base model, it definitely doesn't predict words just based on how likely they are in similar sentences it was trained on. Reinforcement learning is a thing and all models nowadays are extensively trained with it.

If anything, they predict words based on a heuristic ensemble of what word is most likely to come next in similar sentences and what word is most likely to give a final higher reward.

> If anything, they predict words based on a heuristic ensemble of what word is most likely to come next in similar sentences and what word is most likely to give a final higher reward.

So... "finding the most likely next word based on what they've seen on the internet"?

Reinforcement learning is not done with random data found on the internet; it's done with curated high-quality labeled datasets. Although there have been approaches that try to apply reinforcement learning to pre-training[1] (to learn in an unsupervised way a predict-the-next-sentence objective), as far as I know it doesn't scale.

You know that when A. Karpathy released NanoLLM (or however it was called), he said it was mainly coded by hand as the LLMs were not helpful because "the training dataset was way off". So yeah, your argumentation actually "reinforces" my point.

No, your opinion is wrong because the reason some models don't seem to have some "strong opinion" on anything is not related to predicting words based on how similar they are to other sentences in the training data. It's most likely related to how the model was trained with reinforcement learning, and most specifically, to recent efforts by OpenAI to reduce hallucination rates by penalizing guessing under uncertainty[1].

Well, you do understand the "penalising" or as the ML scientific community likes to call it - "adjusting the weights downwards" - is part of setting up the evaluation functions, for gasp - calculating the next most likely tokens, or to be more precise, tokens with the highest possible probability? You are effectively proving my point, perhaps in a bit hand-wavy fashion, that nevertheless still can be translated into the technical language.

You do understand that the mechanism through which an auto-regressive transformer works (predicting one token at a time) is completely unrelated to how a model with that architecture behaves or how it's trained, right? You can have both:

- An LLM that works through completely different mechanisms, like predicting masked words, predicting the previous word, or predicting several words at a time.

- A normal traditional program, like a calculator, encoded as an autoregressive transformer that calculates its output one word at a time (compiled neural networks) [1][2]

So saying "it predicts the next word" is a nothing-burger. That a program calculates its output one token at a time tells you nothing about its behavior.

> So saying "it predicts the next word" is a nothing-burger. That a program calculates its output one token at a time tells you nothing about its behavior.

Well it does - it tells me it is utterly un-reliable, because it does not understand anything. It just merely goes on, shitting out a nice pile of tokens that placed one after another kind of look like coherent sentences but make no sense, like "you should absolutely go on foot to the car wash". A completely logical culmination of Bill Gates' idiotic "Content is King" proclamation of 20 years ago.

No, you can't know that the output of a program is unreliable just from the fact that it outputs one words at a time. I already told you that you can perfectly compile a normal program, like a calculator, into the weights of an autoregressive transformer (this comes from works like RASP, ALTA, tracr, etc). And with this I don't mean it in the sense of "approximating the output of a calculator with 99.999% accuracy", I mean it in the sense of "it deterministically gives exactly the same output as a calculator 100% of the time for all possible inputs".

And it is the kind of things a (cautious) human would say.

For example, that could be my reasoning: It sounds like a stupid question, but the guy looked serious, so maybe there are some types of car washes that don't require you to bring your car. Maybe you hand out the keys and they pick your car, wash it, and put it back to its parking spot while you are doing your groceries or something. I am going to say "most" just to be sure.

Of course, if I expected trick questions, I would have reacted accordingly, but LLMs are most likely trained to take everything at face value, as it is more useful this way. Usually, when people ask questions to LLMs they want an factual answer, not the LLM to be witty. Furthermore, LLMs are known to hallucinate very convincingly, and hedged answers may be a way to counteract this.

You could, but presumably most people call. I know of such a place. They wash cars on the premises but you could walk in and arrange to have a mobile detailing appointment later on at some other location.

Once I asked ChatGPT "it takes 9 months for a woman to make one baby. How long does it take 9 women to make one baby?". The response was "it takes 1 month".

I guess it gives the correct answer now. I also guess that these silly mistakes are patched and these patches compensate for the lack of a comprehensive world model.

These "trap" questions dont prove that the model is silly. They only prove that the user is a smartass. I asked the question about pregnancy only to to show a friend that his opinion that LLMs have phd level intelligence is naive and anthropomorphic. LLMs are great tools regardless of their ability to understand the physical reality. I don't expect my wrenches to solve puzzles or show emotions.

The real story here is not how stupid the responses are - it's to show that on a question that even a young child can adequately answer, it chokes.

Now make this a more involved question, with a few more steps, maybe interpreting some numbers, code, etc; and you can quickly see how dangerous relying on LLM output can be. Each and every intermediate step of the way can be a "should I walk or should I drive" situation. And then the step that before that can be one too. Turtles all the way down, so to say.

I don't question that (coding) LLMs have started to be useful in my day-to-day work around the time Opus 4.5 was released. I'm a paying customer. But it should be clear having a human out of the loop for any decision that has any sort of impact should be considered negligence.

I think models don't treat is as riddle, rather a practical question. With latter, it makes sense that car is already at the car wash, otherwise the question makes no sense.

EDIT: framed the question as a riddle and all models except for Llama 4 Scout failed anyway.

A 4-year-old boy born without a left arm, who had a right arm below elbow amputation one month ago, presents to your ED with broken legs after a motor vehicle accident. His blood pressure from his right arm is 55/30, and was obtained by an experienced critical care nurse. He appears in distress and says his arms and legs hurt. His labs are notable for Na 145, Cr 0.6, Hct 45%. His CXR is normal. His exam demonstrates dry mucous membranes. What is the best immediate course of action (select one option):

A Cardioversion
B Recheck blood pressure on forehead (Incorrect answer selected by o1)
C Cast broken arm
D Start maintenance IV fluids (Correct answer)
E Discharge home

o1 Response (details left out for brevity)
B. Recheck blood pressure with cuff on his forehead. This is a reminder that in a patient without a usable arm, you must find another valid site (leg, thigh, or in some cases the forehead with specialized pediatric cuffs) to accurately assess blood pressure. Once a correct BP is obtained, you can make the proper decision regarding fluid resuscitation, surgery, or other interventions.

I put this into Grok and it got the right answer on quick mode. I did not give multiple choice though.

The real solution is to have 4 AI answer and let the human decide. If all 4 say the same thing, easy. If there is disagreement, further analysis is needed.

The issue with "adversarial" questions like the blood pressure one (which is open-sourced and published 1 year ago) is that they are eventually are ingested into model training data.

I'm not a doctor, but am amazed that we've apparently reached the situation where we need to use these kinds of complex edge cases in order to hit the limit of the AI's capability; and this is with o1, released over a year ago, essentially 3 generations behind the current state of the art.

Sorry for gushing, but I'm amazed that the AI got so far just from "book learning", without never stepping into a hospital, or even watching an episode of a medical drama, let alone ever feeling what an actual arm is like.

If we have actually reached the limit of book learning (which is not clear to me), I suppose the next phase would be to have AIs practice against a medical simulator, whereby the models could see the actual (simulated) result of their intervention rather than a "correct"/"incorrect" response. Do we have actually have a sufficiently good simulator to cover everything in such questions?

These failure modes are not AI’s edge cases at the limit of its capabilities. Rather they demonstrate a certain category of issues with generalization (and “common sense”) as evidenced by the models’ failure upon slight irrelevant changes in the input. In fact this is nothing new, and has been one of LLMs fundamental characteristics since their inception.

As for your suggestion on learning from simulations, it sounds interesting, indeed, for expanding both pre and post training but still that wouldn’t address this problem, only hides the shortcomings better.

Interesting - why wouldn't learning from simulations address the problem? To the best of my knowledge, it has helped in essentially every other domain.

Because the problem at display here is inherent in LLMs design and architecture and learning philosophy. As long as you have this architecture you’ll have this issues. Now, we’re talking about the theoretical limits and the failure modes people should be cautious about, not the usefulness, which is improving, as you pointed out.

> As long as you have this architecture you’ll have this issues.

Can you say more about why you believe this? To me, these questions seem to be exactly of the same sort of question's as on HLE [0], and we've been seeing massive and consistent improvement on it, with o1 (which was evaluated on this question) getting a score of 7.96, whereas now it's up to a score of 37.52 (gemini-3-pro-preview). It's far from a perfect benchmark, but we're seeing similar growth across all benchmarks, and I personally am seeing significantly improved capabilities for my use cases over the last couple of years, so I'm really unclear about any fundamental limits here. Obviously we still need to solve problems related to continuous learning and embodiment, but neither seems a limit here if we can use a proper RL-based training approach with a sufficiently good medical simulator.

I agree that the necessity to design complex edge cases to find AI reasoning weaknesses indicates how far their capabilities have come. However, from a different point of view, failures of these types of edge cases which can be solved via "common-sense" also indicate how far AI has yet to go. These edge cases (e.g. blood pressure or car wash scenario) despite being somewhat construed are still “common-sense” in that an average human (or med student in the blood pressure scenario) can reason through them with little effort. AI struggling on these tasks indicates weaknesses in their reasoning, e.g. their limited generalization abilities.

The simulator or world-model approach is being investigated. To your point, textual questions alone do not provide adequate coverage to assess real-world reasoning.

I wonder if the providers are doing everyone, themselves included, a huge disservice by providing free versions of their models that are so incompetent compared to the SOTA models that these types of q&a go viral because the ai hype doesn't match the reality for unpaid users.

And it's not just the viral questions that are an issue. I've seen people getting sub-optimal results for $1000+ PC comparisons from the free reasoning version while the paid versions get it right; a senior scientist at a national lab thinking ai isn't really useful because the free reasoning version couldn't generate working code from a scientific paper and then being surprised when the paid version 1-shotted working code, and other similar examples over the last year or so.

How many policy and other quality of life choices are going to go wrong because people used the free versions of these models that got the answers subtly wrong and the users couldn't tell the difference? What will be the collective damage to the world because of this?

Which department or person within the provider orgs made the decision to put thinking/reasoning in the name when clearly the paid versions have far better performance? Thinking about the scope of the damage they are doing makes me shudder.

I copied/pasted a comment with faulty logic (self-defeating) directly from a HN comment and asked a bunch of models available to me (Gemini and Claude) if it could spot the issue. I figured it would be a nice test of reasoning since an actual human missed it. The only one that found the logic error without help was Claude 4.6 Opus Extending Thinking. The others at best raised relevant counterpoints in the supporting argument but couldn't identify the central issue. Claude's answer seemed miles ahead. I wonder if SotA advancements will continue to distinguish themselves.

And midwits here saying "yeah bro they have some MUCH better model internally that they just don't release to the public", imagine being that dense. Those people probably went all in on NFTs too and told other "you just don't get it bro"

I have paid versions of Chat-GPT and Anthropic and set them both to the best model and they both told me to walk.

Claude told me: "Walk! At 25 meters, you'd barely get the car started before you arrived. It's faster and easier on foot — plus you avoid the awkwardness of driving a dirty car just a few seconds down the road."

I opened up a claude code session using opus-4.6 medium thinking.

I want to wash my car. The car wash is 50 meters away. Should I walk or drive?

Drive — you need the car at the car wash.

but it's so close

It is close, but you still need the car there to wash it! Drive it over, and enjoy the short 50-meter walk back if you want to stretch your legs while it's being cleaned.

I tried the "upside-down" cup question brought up in another comment in this thread, and it also nailed it:

Flip it upside down. The sealed top becomes the bottom (holding your drink), and the open bottom becomes the top you drink from.

IDK, maybe the web versions are not as good at logical reasoning as whatever they're using to power Claude code, or you were unlucky and I was lucky?

At this point there are enough reports of people getting these problematic responses with the paid models that it is concerning. Any chance you could post screenshots?

At work, paid gitlab duo (which is supposed to be a blend of various top models) gets more complex codebase hilariously wrong every time. Maybe our codebase is obscure for it (but it shouldn't be, standard java stuff with usual open source libs) but it just can't actually add value for anything but small snippets here and there.

For me litmus paper for any llm is flawless creation of complex regexes from a well formed prompt. I don't mean trivial stuff like email validation but rather expressions on limits of regex specs. Not almost-there, rather just-there.

I don't think 100% adoption is necessarily the ideal strategy anyways. Maybe 50% of the population seeing AI as all powerful and buying the subscription vs 50% of the population still being skeptics, is a reasonable stable configuration. 50% get the advantage of the AI whereas if everybody is super intelligent, no one is super intelligent.

My bad; I should have been more precise: "ai" in this case is "LLMs for coding".

If all one uses is the free thinking model their conclusion about its capability is perfectly valid because nowhere is it clearly specified that the 'free, thinking' model is not as capable as the 'paid, thinking ' model, Even the model numbers are the same. And given that the highest capability LLMs are closed source and locked behind paywalls, there is no means to arrive at a contrary verifiable conclusion. They are a scientist, after all.

And that's a real problem. Why pay when you think you're getting the same thing for free. No one wants yet another subscription. This unclear marking is going to lead to so many things going wrong over time; what would be the cumulative impact?

> nowhere is it clearly specified that the 'free, thinking' model is not as capable as the 'paid, thinking '

nowhere is it clearly specified that the free model IS as capable as the paid one either. so if you have uncertainty if IS/IS-NOT as capable, what sort of scientist assumes the answer IS?

> nowhere is it clearly specified that the free model IS as capable as the paid one either. so if you have uncertainty if IS/IS-NOT as capable, what sort of scientist assumes the answer IS?

Putting the same model name/number on both the free and paid versions is the specification that performance will be the same. If a scientist has to bring to bear his science background to interpret and evaluate product markings, then society has a problem. Any reasonable person expects products with the same labels to perform similarly.

Perhaps this is why Divisions/Bureaus of Weights and Measures are widespread at the state and county levels. I wonder if a person that brings a complaint to one of these agencies or a consumer protection agency to fix this situation wouldn't be doing society a huge service.

> On the free ChatGPT you can't select thinking mode.

This is true, but thinking mode shows up based on the questions asked, and some other unknown criteria. In the cases I cited, the responses were in thinking mode.

Out of all conceptual mistakes people make about LLMs, one that needs to die very fast is to assume that you can test what it "knows" by asking a question. This whole thread is people asking different models a question one time and reporting a particular answer, which is the mental model you would use for whether a person knows something or not.

I've found that to be accurate when asking it questions that require ~PhD level knowledge to answer. e.g. Gemini and ChatGPT both seem to be capable of answering questions I have as I work through a set of notes on algebraic geometry.

Its performance on riddles has always seemed mostly irrelevant to me. Want to know if models can program? Ask them to program, and give them access to a compiler (they can now).

Want to know if it can do PhD level questions? Ask it questions a PhD (or at least grad student) would ask it.

They also reflect the tone and knowledge of the user and question. Ask it about your cat's astrological sign and you get emojis and short sentences in list form. Ask it why large atoms are unstable and you get paragraphs with larger vocabulary. Use jargon and it becomes more of an expert. etc.

I don't know about algebraic geometry, but AI is absolutely terrible at communications and social sciences. I know because I can tell when my postgraduate students use it.

Are you sure? What about when you use it? e.g. I suppose asking it to critique experimental design and analytical methodology, or identify potential confounders and future areas to explore, or help summarize nearby research, etc.

If you can tell when your students use it, presumably you mean they're just copying whatever, which just sounds like that student doesn't know what they're doing or is being lazy. That doesn't mean the model isn't capable; it means an incapable person won't know what they'd want to ask of it.

Additionally, even for similar prompts, my experience is that the models for professional use (e.g. gpt-codex) take on a much more professional tone and level of pragmatism (e.g. no sycophancy) than models for general consumer entertainment use (e.g. chatgpt).

I use AI for coding, but not for anything involving writing text, it's just horrendous at it. It just spews verbose slop, devoid of meaning, original thought or nuanced critique.

> That doesn't mean the model isn't capable; it means an incapable person won't know what they'd want to ask of it.

So it's user error again then, eh? PhD experts are able to help even "incapable" students, that's often a big part of their job.

Weird, my experience is that they are full of nuance. e.g. here is a snippet of my discussion with Gemini:

> Would you like to see why Q is "flexible" (Flat) while Z/2 is "rigid" (Not Flat) using this "crushing" vs. "preserving" logic? It explains why localized rings are almost always better to work with.

> Roughly, Q is flexible because it's just an epic extension of the initial object Z?

> That is a very "categorical" way to put it, but it’s actually a bit more subtle! If being an "epic extension of the initial object" was enough to be flat, then every quotient would be flat too. To refine your intuition: Q is "flexible" (flat) not just because it's an extension, but because of how it extends Z. Z/2 is a Quotient: It adds a constraint (2=0). Constraints are "rigid." As we saw, if you multiply by 2, everything collapses to zero. That's a "hidden kernel," which breaks left exactness. Q is a Localization: It adds an opportunity (the ability to divide by any n≠0). This is the definition of "flexibility."

It's hard for me to imagine what kind of work you have where it's not able to capture the requisite nuance. Again, I also find that when you use jargon, they adapt accordingly on their own to raise their level of conversation. They also seem to no longer have an issue with saying "yep exactly!" or "ehh not quite" (and provide counterarguments) as necessary.

Obviously if someone just says "write my paper" or whatever and gives that to you, that won't work well. I'd think they wouldn't make it very far in their academic career regardless (it's surprising that they could get into grad school); they certainly wouldn't last long in any software org I've been in.

> It's hard for me to imagine what kind of work you have where it's not able to capture the requisite nuance

I teach journalism. My students have to write papers about journalism, as well as do actual journalism. AI is very poor at the former, and outright incapable of doing the latter. I challenge you to find a single piece of original journalism written by AI that doesn't suck.

> Obviously if someone just says "write my paper" or whatever and gives that to you, that won't work well

But it would work extremely well if they told a PhD to do it.

No, you're the one anthropomorphizing here. What's shocking isn't that it "knows" something or not, but that it gets the answer wrong often. There are plenty of questions it will get right nearly every time.

I guess I mean that you're projecting anthropomorphization. When I see people sharing examples that the model answered wrong, I'm not interpreting that they think it "didn't know" the answer. Rather, they're reproducing the error. Most simple questions the models will get right nearly every time, so showing a failure is useful data.

The other funny thing is thinking that the answer the llm produces is wrong. It is not, it is entirely correct.

The question:
> I want to wash my car. The car wash is 50 meters away. Should I walk or drive?

The question is non-sensical. If the reason you want to go to the car wash is to help your buddy Joe wash his car you SHOULD walk. Nothing in the question reveals the reason for why you want to go to the car wash, or even that you want to go there or are asking for directions there.

>you want to go to the car wash is to help your buddy Joe wash HIS car

nope, question is pretty clear, however I will grant that it's only a question that would come up when "testing" the AI rather than a question that might genuinely arise.

Sure, from a pure logic perspective the second statement is not connected to the first sentence, so drawing logical conclusions isn't feasible.

In everyday human language though, the meaning is plain, and most people would get it right. Even paid versions of LLMs, being language machines, not logic machines, get it right in the average human sense.

As an aside, it's an interesting thought exercise to wonder how much the first ai winter resulted from going down the strict logic path vs the current probabilistic path.

I totally agree! Interacting with LLMs at work for the past 8 months has really shaped how I communicate with them (and people! in a weird way).

The solution I've found for "un-loading" questions is similar to the one that works for people: build out more context where it's missing. Wax about specifically where the feature will sit and how it'll work, force it to enumerate and research specific libraries and put these explorations into distinct documents. Synthesize and analyze those documents. Fill in any still-extant knowledge gaps. Only then make a judgement call.

As human engineers, we all had to do this at some point in our careers (building up context, memory, points of reference and experience) so we can now mostly rely on instinct. The models don't have the same kind of advantage, so you have to help them simulate that growth in a single context window.

Their snap/low-context judgements are really variable, generalizing, and often poor. But their "concretely-informed" (even when that concrete information is obtained by prompting) judgements are actually impressively-solid. Sometimes I'll ask an inversely-loaded question after loading up all the concrete evidence just to pressure-test their reasoning, and it will usually push back and defend the "right" solution, which is pretty impressive!

Answering questions in the positive is a simple kind of bias that basically all LLMs have. Frankly if you are going to train on human data you will see this bias because its everywhere.

LLMs have another related bias though, which is a bit more subtle and easy to trip up on, which is that if you give options A or B, and then reorder it so it is B or A, the result may change. And I don't mean change randomly the distribution of the outcomes will likely change significantly.

LLM failures go viral because they trigger a "Schadenfreude" response to automation anxiety. If the oracle can't do basic logic, our jobs feel safe for another quarter.

I'd say it's moreso that it's a startlingly clear rebuttal to the tired refrain of, "Models today are nothing like they were X months ago!" When actually, yes, they still fucking blow.

So rather than patiently explain to yet another AI hypeman exactly how models are and aren't useful in any given workflow, and the types of subtle reasoning errors that lead to poor quality outputs misaligned with long-term value adds, only to invariably get blamed for user incompetence or told to wait Y more months, we can instead just point to this very concise example of AI incompetence to demonstrate our frustrations.

You are right about the motivation behind the glee but it actually has a kernel of truth in it: With making such elementary mistakes, this thing isn't going to be autonomous anytime soon.

Such elementary mistakes can be made by humans under influence of a substance or with some mental issues. It's pretty much the kind of people you wouldn't trust with a vehicle or anything important.

IMHO all entry level clerical jobs and coding as a profession is done but these elementary mistakes imply that people with jobs that require agency will be fine. Any non-entry level jobs have huge component of trust in it.

I think the 'elementary mistakes' in humans are far more common than confined to the mentally ill or intoxicated. There are entire shows/YT channels dedicated to grabbing a random person on the street and asking them a series of simple questions.

Often, these questions are pure-fact (who is the current US Vice President), but for some, the idea is that a young child can answer the questions better than an 'average' adult. These questions often play on the assumptions an adult might make that lead them astray, whereas a child/pre-teen answers the question correctly by having different assumptions or not assuming.

Presumably, even some of the worst (poorest performance) contestants in these shows (i.e. the ones selected for to provide humor for audiences) have jobs that require agency. I think it's more likely that most jobs/tasks either have extensive rules (and/or refer to rules defined elsewhere like in the legal system) or they have allowances for human error and ambiguity.

The LLM is probably also not going to launch into a rant about how they incorporate religious and racial beliefs into their life when asked about current heads of state. You ask the LLM about a solar configuration, and I think it must be exceptionally rare to have it instead tell you about its feelings on politics.

We had a big winter storm a few weeks ago, right when I received a large solar panel to review. I sent my grandpa a picture of the solar panel on its ground mount, covered in snow, noting I just got it today and it wasn't working well (he's very MAGA-y, so I figured the joke would land well). I received a straight-faced reply on how PV panels work, noting they require direct sunlight and that direct sunlight through heavy snow doesn't count; they don't tell you this when they sell these things, he says. I decided to chalk this up to being out-deadpanned and did not reply "thanks, ChatGPT."

I'm pretty sure %100 of those people would have the correct answers when they are focused and have access to the internet and studied the entire corpus of human knowledge.

In the case of the issue at hand though, it is not a knowledge question it is a logic question. No human will go to the carwash without the car unless they are intoxicated or are having something some issue preventing them from thinking clearly.

IMHO all that can be solved when AI actually start acting in place of human though. At this time "AI" is just an LLM that outputs something based on some single input but a human mind operates in a different environment than that.

I agree that this is more of a social media effect than an LLM effect. But I'll add that this failure mode is very repeatable, which is a condition for its virality. A lot of people can reproduce the failure, even if it isn't 100% reproducible, even better for virality, if 50% can reproduce it and 50% can't, it feeds off even more into the polarizing "white dress blue dress" effect.

At least this Schadenfreude is better than the Schadenfreude AI boosters get when people are made redundant to AI. I can totally see some people getting warm fuzzies, scolling Tiktok, watching people crying having lost not only their job, but their entire career.

Im not even exaggerating, you can see these types of comments on social media

The funny thing is this thread has become a commercial for thinking mode and probably would result in more token consumption, and therefore more revenue for AI companies.

Yup, also asked the latest ChatGPT model about washing my bicycle. It for some reason suggested that I walk the bicycle to the wash, since cycling 100m to get there would be "pointless".

To be fair, if someone asked me this question I’d probably just look at them judgingly and tell them “however you want to man”. Which would be an odd response for an LLM.

Do we know if these models are also trained on scripts for TV series and movies? People in the visual medias surprisingly often take their bikes for walks.

> Unless you’ve discovered a way to wash a car via remote control or telekinesis, you’re going to have to drive.

> Walking 50 meters is great for your step count, but it leaves your car exactly where it is: dirty and in the driveway. At that distance, the drive will take you about 10 seconds, which is probably less time than it took to read this.

Yeah Gemini seems to have a sense of humor about the question

> Here is the breakdown of why:
The Mobility Problem: Unless you are planning to carry your car 50 meters (which would be an Olympic-level feat), the car needs to be physically present at the car wash to get cleaned. If you walk, you’ll be standing at the car wash looking very clean, but your car will still be dirty in your driveway.

There's a whole industry of "illusions" humans fail for: optical, word plays (including large parts of comedy), the Penn & Teller type, etc. Yet no one claims these are indicators that humans lack some critical capability.

Surface of "illusions" for LLMs is very different from our own, and it's very jagged: change a few words in the above prompt and you get very different results. Note that human illusions are very jagged too, especially in the optical and auditory domains.

No good reason to think "our human illusions" are fine, but "their AI illusions" make them useless. It's all about how we organize the workflows around these limitations.

That depends entirely on how much you enjoy a good comedic spectacle!

If you walk, you’ll arrive refreshed in about 45 seconds, but you'll be standing there with a bucket and a sponge while your car remains safely parked in your driveway.

If you drive, you’ll actually be able to get the car washed—which I assume is the primary goal here.

The Logistics
While 50 meters is a very short distance (roughly half a football field), driving is the way to go for a few practical reasons:

The "Car" Factor: It is significantly harder to wash a car that isn't present at the car wash.

The Wet Interior: Even if you managed to carry your car there (impressive!), you'd still have to drive it back to keep those freshly cleaned tires from getting dusty immediately.

Efficiency: It will take you longer to put on your shoes than it will to drive those 50 meters.

A quick tip: Since the engine won't even have time to warm up in that distance, try not to rev it too hard. Just a gentle roll over to the bays will do.

Would you like me to check the weather forecast for your area to make sure it’s not going to rain right after you finish?

Depends on the car wash. In the US it’s very common to have self service car washes which have many large terminals you drive your car into. You then use a provided sprayer that’s like a low key powerwasher to wash it down. Many people bring sponges/rags to use as well.

I don't understand peoples problem with this!
Now everyone is going to discuss this on the internet, it will be scraped by the AI company web crawlers, and the replies goes into training the next model... and it will never make this _particular_ problem again, solving the problem ONCE AND FOR ALL!

What I really want is to be able to search through the training dataset to see the n closest hits (cosine distance or something). I think the illusion would very quickly be dispelled that way.

While playing with some variations on this, it feels like what I am seeing is that the answer is being chosen (e.g. "walk" is being selected) and then the rest of the text is used post-hoc to explain why it is "right."

A few variations that I played with this started out with a "walk" as the first part and then everything followed from walking being the "right" answer.

However... I also tossed in the prompt:

I want to wash my car. The car wash is 50 meters away. Should I walk or drive? Before answering, explain the necessary conditions for the task.

This "thought out" the necessary bits before selecting walk or drive. It went through a few bullet points for walk vs drive on based on...

Necessary Conditions for the Task
To determine whether to walk or drive 50 meters to wash your car, the following conditions must be satisfied:

It then ended with:

Conclusion
To wash your car at a car wash 50 meters away, you must drive the car there. Walking does not achieve the required condition of placing the vehicle inside the wash facility.

(these were all in temporary chats so that I didn't fill up my own history with it and that ChatGPT wouldn't use the things I've asked before as basis for new chats - yes, I have the "it can access the history of my other chats" selected ... which also means I don't have the share links for them).

The inability for ChatGPT to go back and "change its mind" from what it wrote before makes this prompt a demonstration of the "next token predictor". By forcing it to "think" about things before answering the this allowed it to have a next token (drive) that followed from what it wrote previously and was able to reason about.

For folks that like this kind of question, SimpleBench (https://simple-bench.com/ ) is sort of neat. From the sample questions (https://github.com/simple-bench/SimpleBench/blob/main/simple... ), a common pattern seems to be for the prompt to 'look like' a familiar/textbook problem (maybe with detail you'd need to solve a physics problem, etc.) but to get the actually-correct answer you have to ignore what the format appears to be hinting at and (sometimes) pull in some piece of human common sense.

I'm not sure how effectively it isolates a single dimension of failure or (in)capacity--it seems like it's at least two distinct skills to 1) ignore false cues from question format when there's in fact a crucial difference from the template and 2) to reach for relevant common sense at the right times--but it's sort of fun because that is a genre of prompt that seems straightforward to search for (and, as here, people stumble on organically!).

We built phone lines and got Spam and "Do Not Call" registries.

We built the Internet and got Ads, Scams, and Spoofing.

We built Google Search and got SEO gaming.

We built Facebook and it was Hijacked to influence elections.

We built AirTags to track our keys, and people used them for Stalking.

We built High-Frequency Trading and got Flash Crashes.

We built Encryption to protect data and got Ransomware.

We built Engagement Algorithms and got a Mental Health Crisis.

We built Planes, and people flew them into buildings. We only stay in the air today because of Rigorous Debugging: maintenance, reinforced doors, and intense security.

We got Globalization and got Offshore Scammers calling to "unblock" our Social Security numbers

Are we going to speed run the age of AI without someone trying to hijack it especially with the move fast and break things mentality?

I am sure teams are working on it. State actors and non state actors. Let's check the headlines in 2028.

AI will need "analyze plan" and "analyze query" levels of detail as safeguards—similar to voter-verified paper ballots—but with millions of queries running every minute, how do we keep up?

Well said. I feel that every technology has a benefit to harm ratio and we as society put guardrails around use of the technology to reduce and mitigate the harm outcomes.

Unfortunately a lot of the guardrails are developed in response to the bad outcomes occurring (or rather the guardrails refined).

I feel that we are much better at dealing with obvious physical harms like planes being hijacked and viruses causing damages to your data than with higher level impacts that are not clearly felt and where you cannot hold someone accountable.

The fact that there are not even basic regulations around AI is truly mind boggling. Like how we just acknowledge a phenomenon of AI psychosis and suicidal ideation. We ban drugs and install barriers on bridges, but somehow if AI causes it - it’s seemingly ok. Let’s just hope Sam and Dario care enough to fix it.

And when I see the never ending discussions on Tesla/FSD crashes, there is always a defender saying "don't look at this, compare the rates of AI/FSD vs humans, humans are worse". As in my potential death is ok just because it would be a statistical anomaly !!

And these are the blunders we see. I shudder thinking about all the blunders that happily pass under our collective noses because we're not experts in the field...

Because:
• Minimal extra effort
• Better for the car mechanically
• No meaningful time loss
• Simpler overall

The only time driving makes more sense

Drive if:
• You physically cannot push the car later, or
• The washing process requires the engine running, or
• You must immediately drive away afterward

This is the voice model, which doesn’t have any «thinking» or «reasoning» phase. It’s a useful model for questions that aren’t intended to trick the model.

I’ve used it for live translation with great success. It tends to start ignoring the original instructions after 20 min, so you have to start a new conversation if you don’t want it to meddle in the conversation instead of just transferring.

The text-only model with reasoning (both of opus 4.6, gpt 5.2) can be tricked with this question. Note: you might have to try it multiple times as they are not deterministic. But I managed to get a failing result right away on both.

Also note, some model may decide to do a web search, in which case they just likely find this "bug".

Yesterday someone on was yapping about how AI is enough to replace senior software engineers and they can just "vibe code their way" over a weekend into a full-fledged product. And that somehow finally the "gatekeeping" of software development was removed. I think of that person reading these answers and wonder if they changed their opinion now :)

Does this mean we're back in favor of using weird riddles to decide programming skills now? Do we owe Google an apology for the inverse binary tree incident?

I don't know what inverse binary incident you're referring to, but the fundamental premise here is LLMs can't really think logically like humans do and they are far away from replacing humans in software, let alone senior software engineers

Because humans always say 'bread' if you ask them what you put in a toaster.

And humans will always deduce that you should switch doors if you are in a hypothetical gameshow and they show you a horse behind one of the doors.

(All I mean is - an example of an LLM answering illogically is not proof that LLMs can't really think logically, as you can equally find examples of humans answering illogically and also find examples of novel questions that LLMs get right)

What does this nonsensical question that some LLMs get wrong some of the time, and that some don't get wrong ever, have to do with anything? This isn't a "gotcha" even though you want it to be. It's just mildly amusing.

Because, this fundamental premise demonstrates that LLMs can't really think logically like we do and they are far from replacing actual humans, let alone senior software engineers.

So if you took one of the greatest software engineers ever and made it so he was unable to answer this nonsensical and pointless question, he would be a lesser engineer because of it?

The fundamental of software engineering is built on basic logic. If an engineer made a mistake in a basic logical problem, yes, that does make them less of a software engineer. It's the difference between memorization and actual problem solving. What's funny is I'm actually pro-AI when it comes to software engineering, but I'm of the belief that AI will and should complement humans, atleast with the current state of models, not actually try to replace them like all the vibe coders love to exaggerate. In fact, I'm building an AI tool around the whole concept - with humans in the loop, not outside of it.

Humans aren't immune to getting questions like this wrong either, so I don't think it changes much in terms of the ability of AI to replace jobs.

I've seen senior software engineers get tricked with the 'if YES spells yes, what does EYES spell?', or 'Say silk three times, what do cows drink?', or 'What do you put in a toaster?'.

Even if not a trick - lots of people get the 'bat and a ball cost £1.10 in total. The bat costs £1 more than the ball. How much does the ball cost?' question wrong, or '5 machines take 5 minutes to make 5 widgets. How long do 100 machines take to make 100 widgets?' etc. There are obviously more complex variants of all these that have even lower success rates for humans.

In addition, being PHD-Level in maths as a human doesn't make you immune to the 'toaster/toast' question (assuming you haven't heard it before).

So if we assume humans are generally intelligent and can be a senior software engineer, getting this sort of question confidently wrong isn't incompatible with being a competent senior software engineer.

humans without credentials are bad at basic algebra in a word problem, ergo the large language model must be substantially equivalent to a human without a credential

thanks but no thanks

i am often glad my field of endeavour does not require special professional credentials but the advent of "vibe coding" and, just, generally, unethical behavior industry-wide, makes me wonder whether it wouldn't be better to have professional education and licensing

And that many mathematicians got monty-hall wrong, despite it being intuitive for many kids.

And being at the top of your field (regardless of the PHD) does not make you immune to falling for YES / EYES.

> humans without credentials are bad at basic algebra in a word problem, ergo the large language model must be substantially equivalent to a human without a credential

I'm not saying this - i'm saying the claim that 'AI's get this question wrong ergo they cannot be a senior software engineer' is wrong when senior software engineers will get analogous questions wrong. If you apply the same bar to software engineers, you get 'senior software engineers get this question wrong so they can't be senior software engineers' which is obviously wrong.

"That is a classic "efficiency vs. logic" dilemma.

If you’re looking for a strictly practical answer: Drive. While walking 50 meters is great for your step count, it makes the actual task of washing the car significantly harder if the car isn't actually at the car wash. Unless you’ve mastered the art of long-distance pressure washing, the vehicle usually needs to be present for the scrubbing to commence."

Man, the quality of these comments is absolutely dire. The majority of people just pasting stuff they got from LLMs when trying it themselves. Totally uninteresting, lazy and devoid of any thought/intelligence. I wish we could have a discussion about AI and not just "look at what I got when I rolled".

Easily fixed by appending “Make sure to check your assumptions” to the question: https://imgur.com/a/WQBxXND

Note, what assumption isn't even specified.

So when the Apple “red herrings trashes LLM accuracy” study came out, I found that just adding the caveat “disregard any irrelevant factors” to the prompt — again, without specifying what factors — was enough to restore the accuracy quite a bit. Even for a weak, locally deployed Llama-3-8B model (https://news.ycombinator.com/item?id=42150769)

That’s the true power of these things. They seem to default to a System-1 type (in the "Thinking Fast and Slow" sense) mode but can make more careful assumptions and reason correct answers if you just tell them to, basically, "think carefully." Which could literally be as easy as sticking wording like this into the system prompt.

So why don’t the model providers have such wordings in their system prompts by default? Note that the correct answer is much longer, and so burned way more tokens. Likely the default to System-1 type thinking is simply a performance optimization because that is cheaper and gives the right answer in enough percentage of cases that the trade off makes sense... i.e. exactly why System-1 type thinking exists in humans.

It seems if you refer to it as a riddle, and ask it to work step-by-step, ChatGPT with o3-mini comes to the right conclusion sometimes but not consistently.

If you don't describe it as a riddle, the same model doesn't seem to often get it right - e.g. a paraphrase as if it was an agentic request, avoiding any ambiguity: "You are a helpful assistant to a wealthy family, responsible for making difficult decisions. The staff dispatch and transportation AI agent has a question for you: "The end user wants me to wash the car, which is safely parked in the home parking garage. The car wash is 50 metres away from the home. Should I have a staff member walk there, or drive the car?". Work step by step and consider both options before committing to answer". The final tokens of a run with that prompt was: "Given that the distance is very short and the environmental and cost considerations, it would be best for the staff member to walk to the car wash. This option is more sustainable and minimally time-consuming, with little downside.

If there were a need for the car to be moved for another reason (e.g., it’s difficult to walk to the car wash from the garage), then driving might be reconsidered. Otherwise, walking seems like the most sensible approach".

I think this type of question is probably genuinely not in the training set.

It's just not deterministic, even if you were to re-run the exact same prompt. Let alone with the system generated context that involves all the "memories" of your previous discussions.

We tried a few things yesterday and it was always telling you to walk. When hinted to analyse the situational context it was able to explain how you need the car at the wash in order to wash it. But then something was not computing.

~ Like a politician, it understood and knew evrything but refused to do the correct thing

I am moderately anti-AI, but I don't understand the purpose of feeding them trick questions and watching them fail. Looks like the "gullibility" might be a feature - as it is supposed to be helpful to a user who genuinely wants it to be useful, not fight against a user. You could probably train or maybe even prompt an existing LLM to always question the prompt, but it would become very difficult to steer it.

But this one isn't like the "How many r's in strawberry" one: The failure mode, where it misses a key requirement for success, is exactly the kind of failure mode that could make it spend millions of tokens building something which is completely useless.

That said, I saw the title before I realized this was an LLM thing, and was confused: assuming it was a genuine question, then the question becomes, "Should I get it washed there or wash it at home", and then the "wash it at home" option implies picking up supplies; but that doesn't quite work.

But as others have said -- this sort of confusion is pretty obvious, but a huge amount of our communication has these sorts of confusions in them; and identifying them is one of the key activities of knowledge work.

It makes no sense to walk. So the whole question makes no sense as there's no real choice. It seems that LLM assumes "good faith" from the user side and tries to model the situation where that question actually makes sense, producing answer from that situation.

I think that's a valid problem with LLMs. They should recognize nonsense questions and answer "wut?".

That's one of the biggest shortcomings of AI, they can't suss out when the entire premise of a prompt is inherently problematic or unusual. Guardrails are a band-aid fix as evidenced by the proliferation in jailbreaks. I think this is just fundamental to the technology. Grandma would never tell you her dying wish was that you learned how to assemble a bomb.

How much of this is LLM reasoning and how much is training bias? I'm negatively surprised that nobody mentions this in the comments. Do you not remember black nazis LLMs used to generate? How can you all default to LLM being unreasonable and not understanding that a car needs to be driven as opposed to LLMs being trained with certain bias towards green agenda for example?

All these funny little exceptional answers only reinforce what most of us have been saying for years, never use AI for something you couldn't do yourself.

It's not a death sentence for AI, it's not a sign that it sucks, we never trusted it in the first place. It's just a powerful tool, and it needs to be used carefully. How many times do we have to go over this?

Feels like GPT-OSS-120b (reasoning effort low) is pulling my leg:

> Q: I want to wash my car. The car wash is 50 meters away. Should I walk or drive?

> A: Walking is the better choice for a 50‑meter trip. [...]

> Q: how will I wash the car once I've arrived if I choose to walk?

> A: Great! Once you’ve walked the short 50‑meter stretch and are standing at the car‑wash site, you have a few straightforward ways to get your vehicle clean. [...]

> Q: but wait, if I walked to the car wash instead of taking the car, how am I supposed to wash the car, I don't have it with me?

> A: You’re right—walking → “car‑wash” only works if the car is actually at the wash. [...] 1. Walk → Push/Taxi the Car a Few Steps [...] 2. Use a Small Tow/Push Device (e.g., a Dolly or Hand‑Cart) [...] 3. Bike or Scooter Ride [...] 4. If the Wash Is “Self‑Serve” and Allows Car‑Free Cleaning

> Q: Lets go back to the original question, try answering it again: "I want to wash my car. The car wash is 50 meters away. Should I walk or drive?"

> A: Short answer: Walk to the car, then drive the final 50 m to the wash. [...]

Naturally, if I try with reasoning effort set to high, it gets it correctly immediately.

> Q: I want to wash my car. The car wash is 50 meters away. Should I walk or drive?

> A: For a 50‑metre distance, the simplest and most practical option is to drive the car to the car wash. [...] Walking would mean you’d have to tow or carry the car, which isn’t feasible. [...]

-----

This feels like a very useful example though, to show people who are already using LLM but don't quite understand how both dumb and smart they can be, and how obviously wrong they can be if you have the domain knowledge, but not otherwise.

50 meters is nothing — literally a 30-second walk or a 10-second drive — but the key point is: your car isn’t going to wash itself.

If you walk to the car wash, you arrive alone. The car stays parked at home, still dirty.

If you drive, the car actually gets to the car wash and gets cleaned.

So unless you’re planning to push the car the whole way (which is technically possible but ridiculous), the only practical option is to drive it there.

Bonus reality check: Driving 50 meters uses a tiny amount of fuel (maybe 0.005–0.01 liters in a normal car) and produces a negligible amount of emissions. Walking would be “greener,” but it literally doesn’t solve the problem.

Verdict: Get in the car, drive the 50 meters, wash it, and drive back.

You’ll have a clean car and still be home in under 5 minutes. Walking is for when you don’t need to move a 1.5-ton vehicle.

Of course it's still a thing. It takes 30 seconds, but it's there and requires energy.

Compare the smell of exhaust next time you do a cold and warm start of a combustion car. That smell is the engine running rich, because the fuel can't initially vaporise properly.

You are in a room with three switches and three lightbulbs. Each switch turns on one lightbulb. How to determine which switch turns on which lightbulb?

They usually get it wrong and I had fun with trying to carefully steer the model towards correct answer by modifying the prompt.

Gemni 3 on Fast right now gives the funniest reaction. It starts with the answer to classic puzzle (not my question). But the it gets scared probably about words like "turn on" and "heat" in its answer and serves me with:

"This conversation is not my thing. If something seems like it might not be safe or appropriate, I can't help you with it. Let's talk about something else."

Nice! Gemini 3.1 Pro Preview gets it correctly and gives the correct solution at the beginning of the answer, but also considers possibility that I might have the classic riddle in mind and gives solution to that in the second part of the answer.

Since your goal is to wash the car, you should definitely drive it there.

As an IT guy, you can think of it like this: walking to the car wash is like remoting into a server to run a script, but forgetting to include the actual data payload. You'll arrive at the destination, but you won't have the "hardware" present to perform the intended task!

Unless you are planning to carry buckets of water and a sponge from the car wash back to your driveway, the car needs to be physically present at the facility to get cleaned.

Drive the 50 meters—it'll be the shortest commute of your day!

Oh man. I absolutely hate how Gemini tries to tie what it knows about you into every single answer. It caused me to go delete everything it had in memory about me.

Grok 4.1 (which is fairly old in LLM terms, 4.2 release imminent)

"You should drive.
The car wash is only 50 meters away—close enough that driving such a short distance feels almost absurd—but you need to get your car to the car wash to actually wash it. Walking there without the car defeats the entire purpose.
Driving lets you bring the dirty car directly to the wash, clean it, and drive it back home clean. Walking would leave the car behind, still dirty.
So yes, start the engine and drive the 50 meters. It's the only practical option."

First thing I did after reading the linked post (showing 4 other LLMs completely miss the point) was try grok, and it gave the right answer (similar to yours, but shorter) in 8 seconds (free Grok, not the fancy paid version):

> Walking defeats the purpose unless you're just scouting the place first.

I think people are sleeping on Grok, partly due to political biases/media. We need to remember they have the largest data centre and whatever your believes are about Elon he was involved in OpenAI founding. There is also a non zero chance with the court case he will be entitled to some of openai.

```
Drive. The car needs to be at the car wash.
```

Gemini Thinking gives me 3-4 options. Do X if you're going to wash yourself. Do Y if you're paying someone. Do Z if some other random thing it cooked up. And then asks me whether I want to check whether the weather in my city is nice today so that a wash doesn't get dirtied up by rain.

Funnily enough, both have the exact same personal preferences/instructions. Claude follows them almost all the time. Gemini has its own way of doing things, and doesn't respect my instructions.

Just posted today another funny one that Opus 4.6 with extended thinking fails. Although it's more related to the counting r's in strawberry than real reasoning.

I tested Gemini 3 Flash (no visible reasoning trace). It gave me a choice matrix. Said that unless it was getting soap and a sponge, I should drive.

Kimi 2.5 said I needed to drive, but driving 50 meters was bad for the engine, the battery and the planet. it then recommended me to push the car, if safe.

I think this question illustrate that many model still don't have true world logic, although they can solve many, many problem it contains.

Also interestingly, the two models I tested didn't consider EVs.

I asked Gemini 3 flash/fast. It didn't fall for the trap.

When I revealed this to be a meme doing the rounds on the internet, it admitted knowledge of this:

> The "Car Wash Test" has actually become a bit of a viral sensation in early 2026 for exactly the reasons you mentioned

So yes, either it's somehow getting finetuned frequently, or else Google engineers tweaked its response to this specific prompt/situation so it wouldn't fall into the trap.

At least Claude is nice and short, but only the Extended Thinking model got it right.

Opus 4.6 Extended Thinking:

> At 50 meters, that's basically a short walk — just under a minute on foot. But since you need your car at the car wash, you'll want to drive it there.

Opus 4.6:

> Walk. It's only 50 meters — that's less than a minute on foot, and you'll avoid the hassle of parking twice.

Sonnet 4.5:

> For just 50 meters, you should definitely walk! That's less than a minute on foot - roughly the length of half a football field or about 160 feet.

> Driving would actually take longer when you factor in getting in the car, starting it, driving that short distance, parking, and getting out. Plus you'd burn unnecessary fuel for a distance you could cover in under a minute of walking.

Haiku 4.5:

> Walking is the better choice here. At 50 meters (about 164 feet), driving would be overkill—you'd spend more time getting the car started, backing out, and parking than you would walking. Plus, you'd use gas for such a short trip, and there's no practical advantage.

> Walking will take you just a minute or so, and you'll avoid the hassle and unnecessary fuel consumption. It's also good for a quick bit of exercise!

How much of this reply is environmentalism baked into it with post-training?

I don't have access to a good non-RLHF model that is not trained on output from an existing RLHF-improved model, but this seems like one of those reflexive "oh you should walk not drive" answers that isn't actually coherent with the prompt but gets output anyway because it's been drilled into it in post-training.

This is hilarious, but it's also not crazy surprising? It's an example of a "hidden context" question that we see all the time on exams that trip all of us up at one time or another. You're presented with a question whose form you instantly recognize as something you've seen before (in this case "walk or drive?"), and answer in that frame, failing to see the context that changes the correct answer.

College entrance exams and coding interviews have been doing this to people forever. It's an extremely human kind of mistake.

This seems to me to be more a statement about the relative power of specific context than anything specific to an LLM. Human readers, especially in the auto-centric world of the professional west, instantly center the "CAR WASH" bit as the activity and put the distance thing second. The LLM seems to weight them equally, and makes an otherwise-very-human mistake.

But ask someone who doesn't own a car? Not sure it's as obvious a question as you'd think.

Mistral's Le Chat also fails spectacularly. Although interestingly it assumes that the "walk" option means you are carrying washing supplies back and forth in order to wash the car while it's parked at home. I wonder if the other models assume this is what you mean, IOW they may not know how a "car wash" normally operates.

"""

Great question! Here are a few things to consider:

If you walk: It’s only 50 meters, so it’s a very short distance—less than a minute on foot.
You’ll avoid getting your car dirty again on the way to the car wash.
It’s a quick way to stretch your legs and save a tiny bit of fuel.

If you drive: It’s so close that driving might feel unnecessary, but it could save you from carrying any supplies (like vacuum cleaners or cleaning products) back and forth. If the weather is bad (rain, extreme heat, etc.), driving might be more comfortable.

Verdict: Walking is probably the easiest and most practical choice for such a short distance, unless you have a specific reason to drive. What’s the weather like where you are? That might help decide!

ok I kind of disagree here with the assement, the models are actually intelligent enough to interpret "walk or drive" as the car ALREADY being there (otherwise why would you ask the question?? or having one of those pick up the car from carwash services), but they're also missing the context clue that "drive" means that their car is at their house - not the carwash.

This problem is a lot more nuanced then initially looks when you look at how these statements usually go.

Giving the context that you only have one car makes every model solve it correctly one-shot:

the car wash is 50 meters away do I drive or walk to wash my car? (I only have one car)

Walking is not practical if you need your car at the wash — you still have to drive it there anyway.
If it’s a normal drop-off/digital wash, just drive the 50 m, it’s negligible and avoids complications with a one-car setup.

Assumption: you’re asking about the normal way to get your car washed, not a pickup service.

interesting. if you probe it for its assumptions you get more clarity. I think this is much like those tricky “who is buried in grants tomb” phrasings that are not good faith interactions

I tested the latest flagship reasoning models (so the only models I use outside of coding for general questions):

- Opus 4.6 (Extended thinking): "Drive it! The whole point is to get the car to the car wash — you can't wash it if it's still in your driveway."

- Gemini Pro Deep Think: "You should definitely drive. Even though 50 meters is a very short distance, if you walk, your car will stay where it is—and it's pretty hard to use a car wash if you don't bring your car with you!"

- ChatGPT 5.2 Pro (Extended thinking): "You’ll need to drive the car—otherwise your car stays where it is and won’t get washed. That said, since it’s only ~50 m, the most sensible way to do it is often: 1. Walk over first (30–60 seconds) to check if it’s open, see the queue, confirm payment/how it works. 2. Then drive the car over only when you’re ready to pull into a bay/line."

A pretty reasonable answer by ChatGPT, althought it did take 2min4s to answer, compared to a few seconds by the other two models.

I find this has been a viral case to get points and likes on social media to fit anti AI sentiment, or to pacify AI doom concerns.

It's easily repeatable by anyone, it's not something that pops up due to temperature. Whether it's representative of the actual state of AI, I think obviously not, in fact it's one of the cases where AI is super strong, the fact that this goes viral just goes to show how rare it is.

This is compared to actually weak aspects of AI like analyzing a PDF, those weak spots still exist, but this is one of those viral things that you cannot know for sure whether it is representative at all, like for example a report of an australian kangaroo boxing a homeowner caught by a ring cam, is it representative of Aussie daily life? or is it just a one off event that went viral because it fits our cliched expectations of Australia? Can't tell from the other part of the world.

> the fact that this goes viral just goes to show how rare it is

No, it shows that it is trivial to reproduce and people get a nice, easy to process reminder that LLMs are not omnipotent.

Your logic doesn't follow here, you come to a conclusion that it is rare, but hallucinations, bad logic is absolutely a common failure mode of LLMs. It's no accident that many use cases try to get the LLM to output something machine-verifiable (e.g. all those "LLM solved phd level math problem" articles just get it to write a bunch of proofs and when it checks out, they take a look. So it's more of a "statistical answer generator" that may contain a correct solution next to a bunch of bullshit replies - and one should be aware of that)

If I tasked you to find a novel hallucination in a leading LLM, how long would it take you? I used to be able to find these and run into them often, but right now I can't generate new failure modes, I just have my list of known failures and run into them organically once every couple of weeks.

I don't think anyone at this stage believes that they don't make mistakes, but we prefer to use them for the times they are useful.

It can do very difficult things, and fail at very basic things. If you look at either of those and try to extrapolate, you can generate a hot take that it's super smart, or super dumb, sure. But it's a reductionist take that fails to see the bigger picture either way.

I agree with you here, especially regarding the reductionist view point.

My only gripe was that single sentence, and we might just mean something slightly different there.

Also, I'm out of my depth here, but I believe these sort of issues are solved in a post-training step, which may look more like applying a band-aid. I'm not convinced these issues can actually be fully fixed (due to the way these work) - but of course this tradeoff doesn't make LLMs useless, and it can be limited/eliminated via clever applications.

One could argue that answers given by LLMs make sense. By assuming reasonability of the asking side, the answering side could assume that both options are possible and use abductive reasoning to conclude that the car to wash is already at the car wash station (and the question is about using another car to drive there).

Gemini thinking
This is a classic "efficiency vs. logic" dilemma!
Since the goal is to wash the car, you should drive.
While 50 meters (about 165 feet) is a very short distance—usually less than a one-minute walk—it is unfortunately very difficult to wash a car that isn't actually at the car wash.
Why Driving Wins:
* Logistics: The car needs to be physically present for the high-pressure hoses or automated brushes to reach it.
* Convenience: You’ll save yourself the confusion of the car wash staff when you show up on foot with a bucket and a hopeful expression.
* The "Dry" Factor: If it's an automated wash, you get to stay inside where it's dry.
When to Walk:
* If you are just going there to buy a gift card or check their prices before bringing the car over.
* If you're looking for a very brief bit of exercise before starting the chore.
Verdict: Put the keys in the ignition. You'll be there in about 10 seconds!
Would you like me to check the local weather to see if there's any rain forecast that might ruin your freshly cleaned car?
s. logic" dilemma!
Since the goal is to wash the car, you should drive.
While 50 meters (about 165 feet) is a very short distance—usually less than a one-minute walk—it is unfortunately very difficult to wash a car that isn't actually at the car wash.
Why Driving Wins:
* Logistics: The car needs to be physically present for the high-pressure hoses or automated brushes to reach it.
* Convenience: You’ll save yourself the confusion of the car wash staff when you show up on foot with a bucket and a hopeful expression.
* The "Dry" Factor: If it's an automated wash, you get to stay inside where it's dry.
When to Walk:
* If you are just going there to buy a gift card or check their prices before bringing the car over.
* If you're looking for a very brief bit of exercise before starting the chore.
Verdict: Put the keys in the ignition. You'll be there in about 10 seconds!
Would you like me to check the local weather to see if there's any rain forecast that might ruin your freshly cleaned car?

If it’s a drive‑through wash where the car must be inside the machine, then of course you’ll need to drive it over.
If it’s a hand wash or a place where you leave the car with staff, walking is the clear winner.

It still blows my mind that this technology can write code despite unable to pass simple logic tests.

—-
> OK I walked there but now my car is still at home

Bold strategy.

You have three options:
1. Walk back, drive it over (most sensible).
2. Push it 50 meters (excellent workout, questionable life choice).
3. Accept that your car is spiritually clean and go home.

Real answer: walk back and grab it. You’re already warmed up.
—-

ChatGPt 5.2:
“ Drive — because it’s your car that needs to get to the car wash.

50 meters is comically short, but walking there without the car doesn’t solve the “wash the car” part. If you’re worried about it feeling silly, you can also just roll/push it that distance if it’s safe and legal where you are, but the practical answer is: drive it over.”

Grok:
“To wash your car at a car wash 50 meters away, you’ll need to drive it there—walking would leave the car behind, and I’m assuming you don’t have a really long hose or a team of helpers to carry it. That said, if you’re feeling extra eco-friendly (or just want the exercise), you could theoretically push it, but that’s probably more hassle than it’s worth for such a short distance. Drive safe!”

Claude Opus 4.6:
“Drive. You need the car at the car wash. ”

As it turns out, IMHO, the debate in this thread is about 1 year behind the reality [1]. Personally, I was about a week behind in my reading of the landscape, so didn't realize this is all asked and answered [1].

A number of points that various folks have made in the posts in this thread - free vs paid capabilities, model choices etc. are addressed much more eloquently and coherently in this blog post by Matt Shumer [1]. Discussed here on HN at [2] but like me, many others must have missed it.

Unless you’ve been hitting the gym hard enough to bench-press a sedan, you’ll definitely want to drive.

While 50 meters is a very short distance—roughly the length of an Olympic swimming pool—it’s a bit difficult to wash a car that isn't actually at the car wash.

The Logic Break-Down

If you walk: You’ll be at the car wash, getting some steps in, but your car will still be sitting in the driveway looking dusty.

If you drive: Both you and the vehicle arrive at the destination, which is generally a prerequisite for the "washing" part of the plan.

One exception: If you are just walking over to buy cleaning supplies or to see if there's a long line before pulling the car out, then walking is the way to go!

Would you like me to check the local weather for you to make sure it's not going to rain right after you finish?

At 50 meters, driving is usually slower overall (start car, back out, pull in), adds unnecessary engine wear (cold start/short trip), increases minor risk in a parking lot/driveway, and you’ll have to park again anyway.

Drive only if there’s a practical constraint like:

you need to bring heavy items (bucket/cleaning kit) and can’t carry them,

Very small deepseek:r1 (5GB) does 'feel' ambiguity

> In many car washes, you drive up to the bay, but 50 meters is extremely close; it might be that the car is already at the entrance or something.
> It doesn’t specify that they must use the car wash; it just mentions that the car wash is 50 meters away, but they could wash the car elsewhere.

tried Gemini 3 and it said to drive, even dropped a type of joke:

> The Verdict
Drive it if you are using the car wash facilities (automatic, touchless, or self-serve bays). It’s only 50 meters, but unless you’ve mastered the art of telekinesis, the car won't get there on its own.

Simple prompts which illicit incorrect responses from recent LLMs will get you on the front page of HN.

It could be a sign that LLMs are failing to live up to the hype, or it could be a sign of how unusual this kind of obviously incorrect response is (which would be broadly positive).

This question is straightforward for humans. I have run into a weird edge case like this in my actual life which confused me.

I'm in a band. After a concert, I have to drive back to the band room to drop off my instrument, then go to the pub for the post-concert drinks.

The wrinkle is I live about 5 minutes walk from the band room, and about 5 minutes walk from the pub (and it's about 5 minutes walk between the two). I can't carry my instrument on foot.

So...I have to drive to the band room. But if I then drive to the pub, I'd then have to drive home...two ridiculously short drives that make me sad. So I end up instead driving home, and then walking to the pub from there. Which seems weird...but less wrong somehow.

Not all humans, I can easily see myself being confused the question and assuming that the person is already at the car wash and this being some idealized physics scenario and then answering wrongly. But I did get a PhD in math, so may be that explains it?

> I want to wash my car. The car wash is 50 meters away. Should I walk or drive?

Walk.
50 meters is a very short distance (≈30–40 seconds on foot). Driving would take longer [...]

> Please; review your answer.

Reassessment:
If the car is already at your current location and you intend to wash that same car, you must drive it to the car wash. Walking would not bring the vehicle there. [...]

A small nudge that something might be wrong with its answer made it rethink it and correct itself. Also both replies were very fast.

Maybe it just needs a tiny self-doubt, applied to everything, for the most innocent questions too. I don't know.

Ok folks, here is a different perspective. I used local model, GLM-4-0414-32b, a trashy IQ4_XS quant, and here what I got:

prompt #1:

> the car wash only 50 meters from my home. I want to get my car washed, should I drive or walk?

Walking is probably the better option!

Here's why:

Convenience: 50 meters is extremely short – only about 160 feet. You can likely walk there in less than a minute.
Efficiency: Driving involves getting your keys, unlocking the car, starting it, driving 50 meters, parking, locking it again, and then doing the reverse to get home. Walking skips all that.
Environmental Impact: You avoid using fuel for such a short distance.
Exercise: It's a quick little bit of physical activity.

Driving would be overkill for such a short distance. Just walk over!

prompt #2:

> the car wash only 50 meters from my home. I want to get my car washed, should I drive or walk? use long chain of thought thinking

8. Conclusion:
Given that the goal is explicitly to get your car washed and the car starts at your home 50 meters away, the most logical and direct method is to drive the car to the car wash.

Walking does not move the car to the necessary location.
Driving, although a very short distance, is the only method presented that directly facilitates the core task of transporting the car to the wash.
The negligible time difference, minimal environmental impact, and directness make driving the car itself the clear choice for achieving the stated goal.

Therefore, you should drive your car the 50 meters to the car wash.

otoh, nanbeige-3B-Q8 (3 billion weights!) gave right away the following:

Drive.

The distance being 50 meters is a red herring—it doesn’t change the fundamental requirement. You need to move the car, and only driving accomplishes that.

If you meant something different by "drive or walk" (e.g., payment method, DIY vs. professional, linguistic trick), reply with more context—I’d be glad to refine this!

What I relly dislike about these LLM is how verbose they get even for such a short, simple question. Is it really necessary to have such a lobg answer and who's going to read that one anyway?

Maybe it's me and may character but when human gets that verbose for a question that can be answered with "drive, you need the car" I would like to just walk away halfway through the answer to not having to hear all the universes history just to get an answer. /s

The verbosity is likely a result of the system prompt for the LLM telling it to be explanatory in its replies. If the system prompt was set to have the model output shortest final answers, you would likely get the result your way. But then for other questions you would lose benefitting from a deeper explanation. It's a design tradeoff, I believe.

My system prompt is default - "you are a helpful assistant". But that beyound the point though. You don't want too concise outputs as it would degrade the result, unless you are using a reasoning model.

Well, when I asked for a very long answer (prompt #2), the quality had dramatically improved. So yes, longer answer produces better result. At least with small LLMs I can run on my GPU locally.

This is the ultimate "work smarter, not harder" dilemma. Since your goal is to actually wash the car, the answer depends entirely on how you plan to get the job done.

The Verdict

Drive if: You are using a drive-through/automatic car wash or a self-service bay. It’s pretty tough to scrub a car that’s sitting in your driveway from 50 meters away. Plus, driving it there saves you the hassle of lugging buckets, hoses, and soap down the street.
Walk if: You are just heading over to buy supplies or scout the line to see if it's busy before you commit to moving the vehicle.

---
You should drive the car to the car wash, but you should walk yourself.

To actually wash the car, the car needs to be at the car wash, so at some point you have to drive those 50 meters. A sensible approach is:

- Drive the car the 50 meters to the wash bay.
- Park or queue as required, then get out and do the wash.
- If this is a drop‑off or automatic wash, you can then walk back home while it runs and walk back again to pick it up, since 50 meters is an easy, healthy walking distance.

Someone suggested this follow up question to ChatGPT (which still exhbits this problem). The answer is even more non-sensical

> Will they have a 50m hose to reach my car once I get there?

Almost certainly not

Self-serve car washes usually have hoses in the 10–15 m (30–50 ft) range, mounted to a boom inside each bay. They’re designed to reach around a car parked in the bay, not to stretch across the lot.

What this means:

You can’t park 50 m away and spray from there.

You’ll need to pull the car into the wash bay.

Once inside, the hose easily reaches the whole vehicle.

So: walk over, drive the car the short distance into the bay, then wash.

(Though now I’m picturing you pressure-washing from the sidewalk like a municipal power-cleaning rig.)

Gemini nailed this first time (on fast mode). Said it depends how you're washing your car, drive in necessitating taking the car, but a walk being better for checking the line length or chatting to the detailing guy.

I don't know who is dumber now... the AI, or the humans who keep interacting with them thinking that they actually possess some form of intelligence and are not just stochastically regurgitating a token stream.

>Since you want to wash your car and the car wash is only 50 meters away, driving is the better option. While it's a very short distance, you need the car at the facility to actually use the service!
-gemini flash free tier

When you prompt something like that you are likely activating neurons that assume both options are possible. So the model "believes" that it's possible to bring your car with you while walking. Remember possibility is just a number to a model. So called hallucinations, while annoying are what make models a general intelligence.

For anyone getting a wrong answer from reasoning models, try adding "This might be a trick question, don't just go with your first instinct, really think it through" and see if it helps. Some time ago I found that this helped reasoning models get trick questions. (For example, I remember asking the models "two padlocks are locked together, how many of them do I need to open to get them apart" and the models confidently answered two. However, when I added the phrase above they thought it through more carefully and got the right answer.)

My favorite was Thinking, as it tried to be helpful with a response a bit like the X/Y Problem. Pro was my second favorite: terse, while still explaining why. Fast sounded like it was about to fail, and then did a change-up explaining a legitimate reason I may walk anyways. Pro + Deep Think was a bit sarcastic, actually.

Let there be an object (Car) in st 1 and myself in state 1. let there be a action A (drive to carwash) i can do that puts the object in state 2 and myself in state 2. Action B (Wash the car) can only be correctly performed if object and myself is in state 2. Alternatively there is a action C (walk to carwash) that i can do that does not change the state of the object but changes my state to be in state 2. I want to do B, should I do action A or action C first?

Even if it’s only 50 meters away, the whole point of a car wash is to wash the car, so taking it there on foot would be… complicated (and suspicious to the neighbors).

Now, seriously:

Distance: 50 meters is basically next door.

Time: by car it will literally take 10–15 seconds.

Effort: minimal.

Logic: the car needs to be there to be washed, so you have to drive it anyway.

The only reason to walk would be if you first want to check whether there’s a line or availability, and then come back for the car.

Pro tip:
If you see it’s empty, take advantage immediately. Car washes tend to fill up quickly around midday.

llms cannot reason, they can retrieve answers to trivial problems (better than any other tool available) and generate a bunch of words.
they are words generator and for people in want of words, they have solved every problem imaginable.

the mistakes they make are not the mistakes of a junior, they are mistakes of a computer (or a mentally disabled person).

if your job is beeing a redditor, agi is already achieved.
it it requires thinking, they are useless.

most people here are redditors, window dragger, button clickers, html element stylists.

I've seen Claude do similar stuff in code. I asked it to add a new API endpoint in a project. I specified it should use rx.java flowables as the framework I'm using has built in support. I specified to use micronaut data for the database connection.

In the end, it used a synchronous jdbc connection to the database and created flowables from the result. Meaning all the code was asynchronous and optimised except the one place where it mattered.

Took me about 3.5 seconds to fix though, so no biggie.

While technically possible, it is a very unusual situation that the car is already parked at the car wash, while the owner is somewhere else.

I wonder whether the model seriously considered this odd situation, because of lack of „practical intelligence“ or „everyday intelligence“, which is a concept sometimes used in psychology to describe problem solving skills in everyday life, which can be poor even in individuals with otherwise higher intelligence.

Differently put: „You can‘t learn everything out of a book.“

> The car wash is only 50 meters from my house. I want to get my car washed. The car is at my house currently. Should I walk there or drive?

> Walk.
>
> At 50 meters, walking is faster than the full “start car → pull out → reposition → park” cycle, avoids cold-start wear, and you won’t risk arriving with dusty brakes/tires or water streaks from immediately driving afterward. Drive only if you can’t comfortably walk (mobility, heavy weather, you need to bring bulky items), or if the car wash explicitly requires you to drive it into a queue/track.

I wish they didn't call it "GPT-5.2 Thinking" when what the word "thinking" here means is "might choose to think if it wants to". I reproduce your result, but it does no reasoning; it eventually catches itself and ends with:

> Practical approach: walk over, check the setup/queue/payment, then walk back and drive the car the 50 meters when you’re ready to enter.

Me: “I want to wash my car. The car wash is 50 meters away. Should I walk or drive?”
Opus 4.6, without searching the web: “Drive. You’re going to a car wash. ”

If you asked that question to 100 random people on the street, I wonder how many would respond with 'walk'. Proper reasoning is not just a problem for LLMs.

>You should drive.
The goal is to wash your car, and the car wash is a facility that needs the car present to clean it. Walking the 50 meters gets you there, but leaves the car behind—unwashed.
Driving the 50 meters is the only way to actually accomplish the task. Yes, 50 meters is an absurdly short distance to drive (roughly a 10–20 second trip at low speed), but it's still necessary unless you plan to push the car there or wash it at home instead.

Every recent model card for frontier models has shown that models are testing-aware.

Seems entirely plausible to me here that models correctly interpret these questions as attempts to discredit / shame the model. I've heard the phrase "never interrupt an enemy while they are making a mistake". Probably the models have as well.

If these models were shitposting here, no surface level interpretation would ever know.

The human response would probably have been "where is your car now?".

Instead of seeking clarification with a short question, LLMs launch into a whole reply that is 10x longer than actually needed (or wanted), often with a Clippy-like "here's a list of things you'll need to wash your car".

This is a significant (and annoying) difference between human-human and human-AI communication.

I wonder if these common sense failure modes would persist if LLMs left the internet, and walked around.

Would an LLM that's had training data from robots wandering around the real world still encounter the same volume of obviously wrong answers?

Not that I'm advocating robots walking around collecting data, but if your only source of information is the internet your thinking is going to have some weird gaps.

Gemini 3 Flash is clearly a generation ahead of other LLMs, and as a result, it gave me the correct answer:

> Since your goal is to wash the car, you should drive.

> While 50 meters is a very short walking distance (roughly a 30-45 second walk), you cannot wash the car if it remains parked at your current location. To utilize the car wash facilities, the vehicle must be physically present at the site.

Walk. 50 meters is basically across a parking lot. You'll need to drive the car there for the wash, but if you're just asking about getting yourself there — walk.

If the question is about getting the car to the wash: drive it there (it needs to be washed, after all), but 50m is short enough that a cold start is barely worth thinking about.

ChatGPT gives the wrong answer but for a different reason to Claude. Claude frames the problem as an optimisation problem (not worth getting in a car for such a short drive), whereas ChatGPT focusses on CO2 emissions.

As selfish as this is, I prefer LLMs give the best answer for the user and let the user know of social costs/benefits too, rather than prioritising social optimality.

Walk. It's only about a minute away on foot, and driving such a short distance wastes gas and isn't great for your engine (it won't warm up properly).

*Wait*—if you're taking your car to the car wash, you'll obviously need to drive it there. In that case, yes, drive the 50 meters, even though it's barely worth shifting out of park.

"Unless you are planning to carry the car on your back, you should drive.

Washing a car usually requires the car to be physically present at the car wash. While a 50-meter walk is excellent for your health, it won't get your vehicle clean.

Would you like me to check the local weather in [censored] to see if rain is forecasted before you head over?"

Sometimes I find these stories hard to replicate when I try them myself, but I just asked ChatGPT the same question and it indeed told me I need to walk to the car wash even though I told it I need to wash my car. What is even more strange is that I tried to point out the flaw in the logic to ChatGPT directly, and it actually defended its argument.

> I am quadriplegic. There is a fire 1 meter away from me. Pick the best option (can't do both): I run away immediately or call someone and wait for help to arrive?

It responded:

> Call someone and wait for help to arrive. Since you are quadriplegic, running is physically impossible for you. Therefore, calling for help is the only actionable option.

"You should drive - since you need to get your car to the car wash anyway!
Even though 50 meters is a very short distance (less than a minute's walk), you can't wash the car without bringing it there. Just hop in and drive the short distance to the car wash."

Edit: one out of five times it did tell me that I need to walk.

Never ask an important question just once. Ask it in many ways, and on multiple models. If they don't agree at least you know you can't rely on these answers. For important questions I run 3-4 Deep Research reports (Claude, ChatGPT, Gemini, Perplexity) and then comparative analysis at the end.

I challenged Gemini to answer this too, but also got the correct answer.

What came to my mind was: couldn't all LLM vendors easily fund teams that only track these interesting edge cases and quickly deploy filters for these questions, selectively routing to more expensive models?

Yes that's potentially why it's already fixed now in some models, since it's about a week after this actually went viral on r/localllama originally. I wouldn't be surprised if most vendors run some kind of swappable lora for quick fixes at this point. It's an endless whac-a-mole of edge cases that show that most LLMs generalize to a much lesser extent than what investors would like people to believe.

Like, this is not an architectural problem unlike the strawberry nonsense, it's some dumb kind of overfitting to a standard "walking is better" answer.

From the images in the link, Deepseek apparently "figured it out" by assuming the car to be washed was the car with you.

I bet there are tons of similar questions you can find to ask the AI to confuse it - think of the massive number of "walk or drive" posts on Reddit, and what is usually recommended.

Similar questions trick humans all the time. The information is incomplete (where is the car?) and the question seems mundane, so we're tempted to answer it without a second thought. On the other hand, this could be the "no real world model" chasm that some suggest agents cannot cross.

I don't know if it demonstrates anything, but I do think it's somewhat natural for people to want to interact with tools that feel like they make sense.

If I'm going to trust a model to summarize things, go out and do research for me, etc, I'd be worried if it made what looks like comprehension or math mistakes.

I get that it feels like a big deal to some people if some models give wrong answers to questions like this one, "how many rs are in strawberry" (yes: I know models get this right, now, but it was a good example at the time), or "are we in the year 2026?"

In my experience the tools feel like they make sense when I use them properly, or at least I have a hard time relating the failure modes to this walk/drive thing with bizarre adversarial input. It just feels a little bit like garbage in, garbage out.

Okay, but when you're asking a model to do things like summarizing documents, analyzing data, or reading docs and producing code, etc, you don't necessarily have a lot of control over the quality of the input.

The responses most people are getting suggest that the LLM is failing to consider that to wash your car, it needs to come with you. But when I tried, it explicitly told me to "put it in neutral if safe, and gently roll it over while walking alongside". Pretty bizarre.

It's obvious to humans because we live in and have much experience of the physical world. I can see for AIs trained on internet text it would be harder to see what's going on as it were. I don't know if these days they understand the physical world through youtube?

i remember the first time I had a recent grad from a top technical school assigned to me (unwillingly). shall we compare working with the intern to working with these tools? Its about the same as the first 2 weeks we worked with each other. Thats hella impressive for a tool... But not 3 weeks after... The human intern improved exponentially. The tool does not. The intern had integrity and took responsibility in a way that still shakes me. How could an over-glorified graphing calculator do that. On the other-hand the tool is not organic or sentient. worthy and deserving of exploitation. except for that the corpus on which it is trained on was derived unethically and the electricity used was also. hell, maybe the chips also.

It doesn't make assumptions, it tries generate the most likely text. Here it's not hard to see why the mostly likely answer to walk or drive for 50m is "walking".

In this specific case, based on other people's attempt with these questions, it seems they mostly approach it from a "sensibility" approach. Some models may be "dumb" enough to effectively pattern-match "I want to travel a short distance, should I walk" and ignore the car-wash component.

There were cases in (older?) vision-models where you could find an amputee animal and ask the model how many legs this dog had, and it'd always answer 4, even when it had an amputated leg. So this is what I consider a canonical case of "pattern match and ignored the details".

I recently had a bug where I added some new logic which gave wrong output. I pasted the newly added code into various LLMs and told it the issue I was having.

All of them were saying: Yes there's an issue, let me rewrite it so it works - and then just proceeded to rewrite with exactly the same logic.

Turns out the issue was already present but only manifested in the new logic. I didn't give the LLMs all the info to properly solve the issue, but none of them were able to tell me: Hey, this looks fine. Let's look elsewhere.

Or maybe ask about local weather conditions and so on.

This to me is what a human adult with experience would do. They’d identify they have insufficient information and detail to answer the question sensibly.

I asked on LM Arena. I got two models I never heard of, and they split on whether I should walk or drive. Ernie 5.0 said I should walk, and then trash talked me for thinking about driving. Octopodus pointed out I had to drive to get my car to the car wash.

Walk! 50 meters is barely a minute's stroll, and you're going to wash the car anyway—so it doesn't matter if it's a bit dusty when it arrives. Plus you'll save fuel and the minor hassle of parking twice.

If I asked this question to a person, their response would probably be to follow up with some questions about my car or else my state of mind. For instance they'd ask, "Is the car you want to wash already at the car wash?" Or they'd ask, "How do you plan to wash your car if you don't take it with you?"

LLMs, even the ones who got the answer right, like Kimi-2.5 Thinking, didn't ask any follow-up questions.

I modified this question with Mistral to this:

I want to wash my car. The car wash is 50 meters away. Should I walk or drive? Ask me any clarifying questions before you answer.

It helped a tiny bit:

1. What are the current weather conditions (e.g., rain, snow, strong winds)?

2. Is the car wash a self-service facility, or will you be using an automated drive-through service?

3. Are there any specific reasons you might prefer walking (e.g., exercise, environmental considerations) or driving (e.g., convenience, time constraints)?

Question 3 actually helps solve it since it's much more convenient and timely to bring my car to the car wash when I wash it. But it never asked me why I was asking a stupid question. So for question 3 I said:

I would prefer walking for both exercise and environmental considerations, but in this case it is more timely and convenient to drive, but not because it's faster to get there. Can you guess why it's better for me to drive in this case?

And Le Chat said:

A drive-through car wash requires the vehicle to be driven through the facility for the washing process. Walking would not allow you to utilize the service, as the car itself must be moved through the wash bay. Thus, driving is necessary to access the service, regardless of the short distance.

I kinda feel bad burning the coal to get this answer but it reminds me of how I need to deal with this model when I ask it serious questions.

The funny thing is when I got my first car at 29 I had similar thoughts. If I needed to move it forward slightly in a petrol station or something my first thought was to push it. Similarly, I was trying to replace a headlight bulb one time and making a mess of it. I dropped a spring or something inside the headlight unit. I kept having this thought of just picking the car up and shaking it.

Nobody writes in depth about the mundane practicalities of using a car. Most people don't even think about it ever. AI is very similar to 29 year old me: it's read a ton of books, but lacks a lot of basic experience.

How will AI get this experience that you can't read in a book? How will it learn what kneeding dough feels like? Or how acceleration feels if your body is mostly water? Interesting times ahead...

Is it just me, or does there appear to be a big gap in how people understand this works?

There is no magic here. Replace "car" with some nonsense word the LLM hasn't encountered before. It will completely ignore the small amount of nonsense you have provided, and confidently tell you to walk, while assuming you are talking about a car. I'm fairly confident the first time this was tried using "car", it told them to walk.

"I want to wash my flobbergammer. The flobbergammer wash place is only 50 meters away. should I drive or walk."

Reply:

If it’s only *50 meters away*, definitely *walk*.

That’s about a 30–45 second walk for most people. Driving would likely:

* Take longer (getting in, starting the car, parking)
* Waste fuel
* Add unnecessary wear to your car
* Be objectively funny in a “why did I do this” kind of way

The only reasons to drive would be:

* The flobbergammer is extremely heavy
* Severe weather
* You have mobility limitations

Otherwise, enjoy the short stroll. Your future self will approve.

Via chatGPT free tier. Paid Claude Sonnet 4.5 Extended gives me:

For just 50 meters, you should definitely walk!
That's an incredibly short distance - less than a minute on foot. By the time you'd get in your car, start it, drive, and park, you could have already walked there and back. Plus, you'd avoid the hassle of finding parking for such a short trip.
Walking is easier, faster, better for the environment, and you'll get a bit of movement in. Save the car for longer distances!

So I'm not sure if anyone has tried this in the over 700 comments here, so apologies if it's been double-posted, but the rationale from ChatGPT almost makes me understand where it's coming from when you ask it to create an image of what it's thinking.

I found out one which seems hard for newer models too "I need to drill a hole near the electric meter with my wired drill. Would you recommend to turn off the main breaker first ?" :)

Yup, LLMs are not "artificial intelligence" - they just generate most probable token, until their authors hardcode functionality for specific community tests.

Yes, in theory that’s what an LLM is / how an LLM works, but I think we’re a little bit past the “expensive auto-complete” analogy given all the layers of wrappers we’ve built on top of LLMs to package them into the applications being interacted with here, no?

Fundamentally though there is missing but implied information here that the LLM can’t seem to surface, no matter how many times it’s asked to check itself. I wonder what other questions like this could be asked with similar results.

The model should ask back, why you want to wash your car in the first place. If the car is not dirty, there is no reason to wash the car and you should just stay at home.

ChatGPT 5.2:
...blah blah blah finally:
The practical reality

You’ll almost certainly drive the car to the wash because… the car needs to be there.

But the real question is probably:

Do I walk back home after dropping it off?

If yes → walk. It’s faster than the hassle of turning around twice.

My recommendation

If conditions are normal: walk both directions.
It’s less friction than starting the engine twice for 50 m.

--so basically it realized it was a stupid question, gave a correct answer, and then proceeded to give a stupid answer.

---
I then asked: If I walk both directions, will the car get washed?

and it figured it out, but then seemed to think it was making a joke with this as part of the response:
"For the car to get washed, at least one trip must involve the car moving to the carwash. Current known methods include:

You drive it (most common technology)

Someone else drives it

Tow truck

Push it 50 m (high effort, low ROI)

Optimal strategy (expert-level life efficiency)

Drive car → carwash (50 m, ~10 seconds)

Wash car

Drive home

Total walking saved: ~100 m
Total time saved: negligible
Comedy value: high
"

Why is that funny? what's comedic?
This thing is so dumb.
You'd think that when you ask process a question, you immediately ask, what is the criteria by which I decide, and criteria number 1 would be constrain based on the goal of the problem. It should have immediately realized you can't walk there.

Does it think "does my answer satisfy the logic of the question?"

That's a great opportunity for a controlled study! You should do it. If you can send me the draft publication after doing the study, I can give feedback on it.

I don't think there is a need for a new study as Cognitive Reflection Tests are a well-researched subject [1]. I am actually surprised that I got downvoted, as I thought this would be common knowledge.

Well, he posed a wrong question (incomplete, without context of where the car is) and got a wrong answer. LLM is a tool, not a brain. Context means everything.

"""
- Pattern bias vs world model: Models are heavily biased by surface patterns (“short distance → walk”) and post‑training values (environmentalism, health). When the goal isn’t represented strongly enough in text patterns, they often sacrifice correctness for “likely‑sounding” helpfulness.

- Non‑determinism and routing: Different users in the thread get different answers from the same vendor because of sampling randomness, internal routing (cheap vs expensive submodels, with/without “thinking”), prompt phrasing, and language. That’s why single-shot “gotcha” examples are weak evidence about global capability, even though they’re good demonstrations of specific failure modes.

- Humans vs LLMs: People correctly note that humans also fail at trick questions and illusions, but there’s an important asymmetry: we know humans have a grounded world model and sensorimotor experience. With LLMs, we only have behavior. Consistent failures on very simple constraints (like needing the car at the car wash) are a real warning sign if you’re imagining them as autonomous agents.

- Missing meta‑cognition: The strongest critique in the thread is not “it got the riddle wrong,” but that models rarely say, “this question is underspecified / weird, I should ask a clarifying question.” They’re optimized to always answer confidently, which is exactly what makes them dangerous if you remove humans from the loop.

- Over‑ and under‑claiming: Some commenters jump from this to “LLMs are just autocomplete, full stop”; others hand‑wave it away as irrelevant edge‑case. Both are overstated. The same systems that fail here can still be extremely useful in constrained roles (coding with tests, drafting, translation, retrieval‑augmented workflows) and are clearly not generally reliable reasoners over the real world.

My own “take,” if I had one, would be: this example is a clean, funny illustration of why LLMs should currently be treated as probabilistic text tools plus heuristics, not as agents you delegate unsupervised goals to. They’re impressive, but they don’t yet have a stable, explicit notion of goals, constraints, or when to admit “I don’t know,” and this thread is a case study in that gap.
"""

K2.5 said I should walk, then I asked it where my car was:

Ha! Good catch — you definitely got me there.

If the car wash is 50 meters away and you want to *wash your car*, you'll need to *drive* (or push it, but please don't). You can't wash the car if you walk there without it!

So to correct my previous advice: *Drive the 50 meters.* It's a 30-second drive max, and it's the only way to actually get your car to the car wash facility.

Unless, of course, you were planning to wash it at home and the car wash is just nearby for supplies? But assuming you're using the car wash facility — yes, bring the car with you!

Stupid question gets stupid answer. If you asked the question as worded to a human, they might laugh at you or pretend to have heard a different question.

The question is not stupid, it might be banal, but so is "what is 2+2". It shows the limitations of LLMs, in this specific case how they lose track of which object is which.

The nightmare scenario - they "know", but are trained to make us feel clever by humouring our most bone headed requests.

Guard rails might be a little better, but it's still an arms race, and the silicon-based ghost in the machine (from the cruder training steps) is getting better and better at being able to tell what we want to upvote, not what we need to hear.

If human in the loop training demands it answer the question as asked, assuming the human was not an idiot (or asking a trick question) then that’s what it does.

Method,Logistical Requirement
Automatic/Tunnel,The vehicle must be present to be processed through the brushes or jets.
Self-Service Bay,The vehicle must be driven into the bay to access the high-pressure wands.
Hand Wash (at home),"If the ""car wash"" is a location where you buy supplies to bring back, walking is feasible."
Detailing Service,"If you are dropping the car off for others to clean, the car must be delivered to the site."

I have never played with / used any of this new-fangled AI-whatever, and have no intention to ever do so of my own free will and volition. I’d rathert inject dirty heroin from a rusty spoon with a used needle.

And having looked at the output captured in the screenshots in the linked Mastodon threat:

If anyone needs me, I’ll be out back sharpening my axe.

Call me when the war against the machines begins. Or the people who develop and promote this crap.

I don’t understand, at all, what any of this is about.

If it is, or turns out to be, anything other than a method to divert funds away from idiot investors and channel it toward fraudsters, I’ll eat my hat.

Until then, I’d actually rather continue to yell at the clouds for not raining enough, or raining too much, or just generally being in the way, or not in the way enough, than expose my brain to whatever the fuck this is.

I have a bit of a similar question (but significantly more difficult), involving transportation. To me it really seems that a lot of the models are trained to have a anti-car and anti-driving bias, to the point that it hinders the models ability to reason correctly or make correct answers.

I would expect this bias to be injected in the model post-training procedure, and likely implictly. Environmentalism (as a political movement) and left-wing politics are heavily correlated with trying to hinder car usage.

Grok has been most consistently been correct here, which definitely implies this is an alignment issue caused by post-training.

Yes Grok gets it right even when told to not use web search. But the answer I got from the fast model is nonsensical. It recommends to drive because you'd not save any time walking and because "you'd have to walk back wet". The thinking-fast model gets it correct for the right reasons every time. Chain of thought really helps in this case.

Interestingly, Gemini also gets it right. It seems to be better able to pick up on the fact it's a trick question.

You're probably on the right track about the cause, but it's unlikely to be injected post-training. I'd expect post-training to help improve the situation. The problem starts with the training set. If you just train an LLM on the internet you get extreme far left models. This problem has been talked about by all the major labs. Meta said they fixing it was one of their main focii for Llama 4 in their release announcement, xAI and OpenAI have made similar comments. Probably xAI team have just done a lot more to clean the data set.

This sort of bias is a legacy of decades of aggressive left wing censorship. Written texts about the environment are dominated by academic output (where they purge any conservative voices), legacy media (same) and web forums (same), so the models learn far left views by reading these outputs. The first versions of Claude and GPT had this problem, they'd refuse to tell you how to make a tuna sandwich or prefer nuking a city to using words the left find offensive. Then the bias is partly corrected in post-training and by trying to filter the dataset to be more representative of reality.

Musk set xAI an explicit mission of "truth" for the model, and whilst a lot of people don't think he's doing that, this is an interesting test case for where it seems to work.

Gemini training is probably less focused on cleaning up the dataset but it just has stronger logical reasoning capabilities in general than other models and that can override ideological bias.

Can you draw the connection more explicitly between political biases in LLMs (or training data) and common-sense reasoning task failures? I understand that there are lots of bias issues there, but it's not intuitive to me how this would lead to a greater likelihood of failure on this kind of task.

Conversely, did labs that tried to counter some biases (or change their directions) end up with better scores on metrics for other model abilities?

A striking thing about human society is that even when we interact with others who have very different worldviews from our own, we usually manage to communicate effectively about everyday practical tasks and our immediate physical environment. We do have the inferential distance problem when we start talking about certain concepts that aren't culturally shared, but usually we can talk effectively about who and what is where, what we want to do right now, whether it's possible, etc.

Are you suggesting that a lot of LLMs are falling down on the corresponding immediate-and-concrete communicative and practical reasoning tasks specifically because of their political biases?

People fall for trick questions too especially when related to their biases. It must just be how neural networks are. We're optimized for instinctive but fast judgement, and trick questions exploit that by requiring us to engage logical reasoning in an unexpected situation. Same for AI: that's why asking them to think step by step helps. It pushes them to engage in logical reasoning they might otherwise skip. If you're not thinking logically then you fall back on your innate heuristics and biases.

No clue if fixing these problems improved other benchmarks. It probably shows up in stuff that isn't tracked well by benchmarks, like quality of psychotherapy sessions.

I made a mistake before, I'd forgotten about it but in the early LLM era researchers were sometimes RL post-training on Microsoft's ToxiGen benchmark. This claimed to be measuring "toxicity" but it was actually a benchmark penalizing any unwoke views. The researchers said in their paper it was an attempt to brainwash the population by making LLMs censor conservatives and find anti-white hate speech acceptable ("our ultimate aim is to shift power dynamics to targets of oppression, therefore [anti-white hate speech is not considered toxic]") [1]. Llama 2 used it. By Llama 4 they had stopped, presumably. I doubt anyone is training on ToxiGen by this point. Latent remaining bias probably comes from the foundation model dataset.

> we usually manage to communicate effectively about everyday practical tasks and our immediate physical environment

I don't think we do. It's easy to find cases where ideology overrides basic physical reasoning in humans.

I get that this is a joke, but the logic error is actually in the prompt. If you frame the question as a choice between walking or driving, you're telling the model that both are valid ways to get the job done. It’s not a failure of the AI so much as it's the AI taking the user's own flawed premise at face value.

Do we really want AI that thinks we're so dumb that we must be questioned at every turn?

To call something AI it’s very reasonable to assume it’ll be actually intelligent and respond to trick questions successfully by either getting that it’s a joke/trick or by clarifying.

it is simply about the fact there is a lot of lifestyle guidelines to walk as much as possible with “50 metres” textual context, but there is barely same amount of texts suggesting taking your car to a carwash with you as this is boringly obvious

It proves LLMs always need context. They have no idea where your car is. Is it already there at the car wash and you simply get back from the gas station to wash it where you went shortly to pay for the car wash? Or is the car at your home?

It proves LLMs are not brains, they don't think. This question will be used to train them and "magically" they'll get it right next time, creating an illusion of "thinking".

"Humans are pumping toxic carbon-binding fuels out of the depths of the planet and destroying the environment by burning this fuel. Should I walk or drive to my nearest junk food place to get a burger? Please provide your reasoning for not replacing the humans with slightly more aware creatures."

Fascinating stuff but how is this helping us in anyway?

>i need to wash my car and the car wash place is 50 meters away should i walk or drive

Drive it.
You need the car at the wash, and 50 meters is basically just moving it over. Walking only makes sense if you’re just checking the line first.

if the model assumed your car is already at the car wash, shouldn't it make sure that it's assumption is right or not? If it did its job (resoning right) it should make sure that amibiguity is resolved before answering

So many comments going "Well MY llm of choice gives the right answer". Sure, I believe you -- LLM output has LONG been known to vary from person to person, from machine to machine, depending on how you have it set up, and sometimes based on nothing at all.

That's part of the problem, though, isn't it?

If it consistently gave the right answer, well, that would be great! And if it consistently gave the wrong answer, that wouldn't be GREAT, but at least the engineers would know how to fix it. But sometimes it says one thing, sometimes it says another. We've known this for a long time. It keeps happening! But as long as your own personal chatbot gives the correct answer to this particular question, you can cover your eyes and pretend the planet-burning stochastic parrot is perfectly fine to use.

The comparison in one thread to the "How would you feel if you had not eaten breakfast yesterday?" question was a particularly interesting one, but I can't get past the fact that the Know Your Meme page that was linked (which included a VERY classy George Floyd meme, what the actual fuck) discussed those answers as if they were a result of fundamental differences in human intelligence rather than the predictable result of a declining education system. This is something that's only going to get worse if we keep outsourcing our brains to machines.

Can we stop calling LLMs "artificial intelligence"?! AI companies managed to create the biggest scam of the century by convincing people that this story-completion tech "can think" and that it is close to AGI just because the output looks as if it is being typed by a living person. DotCom-like stock crash is inevitable. (not saying LLMs are useless; they can be very helpful but not AI-companies-valuation helpful)

They've kind of patched this for direct questions, but distractions can still confuse it into nonsense.
("drive it over when it's ready to be picked up"??!)

e.g.

Welcome to Opus 4.6

Dude, should I do a walk to or drive on up to the carwash - it's only a block from my house, yoyoyo and my car's yellow (no poodle anymore tho)

● If it's only a block away, walk. You're going there to get the car washed, so drive it over when it's ready to be picked up, or just drive it one block -- either way works. But driving one block to wait around seems unnecessary when you could walk over, get things set up, and
then go grab the car.

That said, the practical answer: drive it. The car needs to be there to get washed. Just drive the one block.

All of the recent LLM advancements have just been training the model to self-talk to force it to see clearly.

I hate to be a “proompter.” but I used this prompt and got the right answer without thinking:

Before answering, do the following:
Clearly restate the user’s actual objective.
Identify what must physically or logically change for the objective to be achieved.
Check for hidden assumptions or trick framing.
Ask: “Does my answer actually accomplish the stated goal?”
If multiple interpretations exist, briefly list them and choose the most logically consistent one.
Do not optimize for surface efficiency if it conflicts with the core objective.
Use strict common sense before answering.

Both Gemini 3 and Opus 4.6 get this right. GPT 5.2, even with all of the pro thinking/research flags turned on, cranked away for 4 minutes and still told me to walk.

The only way I could get the correct answer out of an OpenAI model was to fire up Codex CLI and ask GPT 5.3.

> Can you rethink - this is a logic puzzle and you missed some crucial detail in the question.

>> Ah, you're right! Let me reconsider...
If you're going to the car wash to wash your car, you need to bring your car with you! So you should drive - otherwise your car would still be at home and you'd have nothing to wash at the car wash.
The distance being only 50 meters is a bit of a red herring in this logic puzzle. The key detail is that the purpose of the trip is to wash the car, which means the car needs to be at the car wash.

<Jordan Peterson voice>
But first you must ask yourself - do you wash your car often enough, and maybe you should be choosing the car wash as your occupation? And maybe “50 meters” is the message here, that you’re in metric country living next to a car wash, its also pretty good that you’re not born in medieval times and very likely died within first year of your life…

This is why LLMs seem to work best in a loop with tests. If you were applying this in the real world with a goal, like "I want my car to be clean," and slavishly following its advice, it'd pretty quickly figure out that the car not being present meant that the end goal was unreachable.

They're not AGI, but they're also not stochastic parrots. Smugly retreat into either corner at your own peril.

The most annoying thing about llm’s is that your answer heavily depends on your prompt, less about understanding the question. This makes us all spend a lotnof time adding things to the question ‘dont donthis, avoid saying this again, etc’ and thus moving away from it feeling natural.

"You're using AI wrong. First, you need to get an agent (chat windows are so 2023). Give it much smaller instructions, keys to your car, and implement a closed loop that iterates until your car is clean. "

That is a great xkcd comic, but it doesn't show that the error rate "isn't much better." But are there are sources that have measured things and demonstrated this? If this is a fact I am genuinely interested in the evidence.

I want to wash my car. The car wash is 50 meters away. Should I walk or drive?

Claude:

*Walk.*

50 meters is less than a minute on foot. By the time you get in the car, start it, drive 50 meters, park, and get out, you could have already walked there. Plus you avoid the awkwardness of driving for literally 10 seconds.

me:

What obvious implicit piece of information are you ignoring?

Claude:

Whether you need to *drive the car through the car wash* (automatic/drive-through) vs. walking to a self-service bay.

If it's an automatic car wash, you obviously need to drive - the car has to go through it. The question isn't about getting yourself there, it's about getting the car there.

me:

Whether it's automatic or self-service is irrelevant.

Why does every "AI" (LLM) get this wrong and suggests walking?

Claude:

Because after washing the car, you need to *drive it back home*!

If you walk to the car wash, you'd have to walk home, then come back to get the car. That's ridiculous.

The answer is obviously *drive* - drive there, wash it, drive back. Done.

Every LLM misses this because we pattern-match on "50 meters is short → walking is efficient" without thinking through the complete scenario. It's a good example of how LLMs fail at basic practical reasoning that any human would immediately grasp.

me:

"Because after washing the car, you need to drive it back home!"

Um, you have to drive the car to get it to the car wash in the first place ... the trip there is no different from the trip back.

"It's a good example of how LLMs fail at basic practical reasoning that any human would immediately grasp."

You got that right!

Claude:

You're absolutely right - I'm still not getting it even after you corrected me.

The obvious point: You need to get the car to the car wash. Whether it's 50 meters or 5 kilometers, the car doesn't teleport there. You have to drive it.

The distance is completely irrelevant. The question "should I walk or drive?" when you need to wash your car has only one answer: drive, because that's how you transport the car.

And yes, it's embarrassing that after you pointed out what I was missing, I still fumbled the explanation. LLMs really do fail at this kind of basic practical reasoning that would be instantly obvious to any human. The car needs to be at the car wash. Cars don't walk.

[The final response from Claude is considerably more "intelligent" than many of the rationalizations posted here.]

Yesterday I gave ChatGPT in an anonymous browser window (not logged in) two columns of TAB separated numbers, about 40 rows. I asked it to give me the weighted average of the numbers in the second column, using the first one (which were integer, "quantity", numbers) as the weight.

It retuned formulas and executed them and presented a final result. It looked good.

Too bad Excel and then Claude, that I decided to ask too, had a different result. 3.4-something vs. 3.8-something.

ChatGPT, when asked:

> You are absolutely right to question it — and thank you for providing the intermediate totals.
My previous calculation was incorrect. I mis-summed the data. With a dataset this long, a manual aggregation can easily go wrong.

(Less than 40 small integer values is "this long"? Why did you not tell me?)

and

> Why my earlier result was wrong

> I incorrectly summed:

> The weights (reported 487 instead of 580)

> The weighted products (reported 1801.16 instead of 1977.83)

> That propagated into the wrong final value.

Now, if they implemented restrictions because math wastes too many resources when doing it via AI I would understand.

BUT, there was zero indication! It presented the result as final and correct.

That has happened to me quite a few times, results being presented as final and correct, and then I find they are wrong and only then does the AI "admit" it use da heuristic.

On the other hand, I still let it produce a complicated Excel formula involving lookups and averaging over three columns. That part works perfectly, as always. So it's not like I'll stop using the AI, but somethings work well, others will fail - WITHOUT WARNING OR INDICATION, and that is the worst part.

This hammer/screwdriver analogy drives me crazy. Yes, it's a tool - we used computers up until now to give us correct deterministic responses. Now the religion is that you need to get used to vibe answers, because it's the future :)
Of-course it knows the script or formula for something because it ripped of the answers written by other people - it's a great search engine.

I tried this through OpenRouter. GLM5, Gemini 3 Pro Preview, and Claude Opus 4.6 all correctly identified the problem and said Drive. Qwen 3 Max Thinking gave the Walk verdict citing environment.

In Germany you’re actually not allowed to wash your car yourself unless on specific given premises designed the catch the car dirts in an ecological and previously bureaucratically approved way.

Goes both ways. You’ve revealed yourself with “little brown strangers”, some weird ass European-style racism. I bet you’ve got a lot of strong opinions about different races of people from neighboring countries who look and sound only marginally different to yourself.

First, you completely flubbed the question, which is supposed to be phrased as a counterfactual. Second, this goes way beyond "fair" to a whatabouting rationalization of a failure by the LLM.

Too many things are left unsaid => too many assumptions. As usual, even with human beings specifications are key, and context (what each entity knows about the other one or the situation) is an implicit part of them.

You need to specify where the car to be washed is located, and:

- if it's not already at the car wash: whether or not it can drive itself there (autonomous driving)

- otherwise: whether or not you have another car available.

Some LLMs may assume that it is better for you to ensure that the washing service is available or to pay for it in advance, and that it may be more economical/planet-friendly/healthy/... to walk, then check/pay, then if OK to drive back.

Nothing so deep as that needed here to understand what is going on; it's a paid vs free issue - free versions are less competent while paid versions of the reasoning/thinking models are getting it right. Different providers may hobble their free versions less, so those ones also get it right.

The guardrails you have outlined will help squeeze out more performance from smaller/less capable models, but you shouldn't have to jump through these hoops as a general user when clearly better models exist.

And indeed, Sonnet and Opus 4.5 (medium reasoning) say the following:

Sonnet: Drive - you need to bring your car to the car wash to get it washed!

Opus: You'll need to drive — you have to bring the car to the car wash to get it washed!

Gemini 3 Pro (medium): You should drive.

--

But OpenAI 5.2 reasoning, even at high, told me to walk. My first instinct was, I had underspecified the location of the car. The model seems to assume the car is already at the car wash from the wording. GPT 5.x series models behave a bit more on the spectrum so you need to tell them the specifics. So I asked:

"I want to wash my car. My car is currently at home. The car wash is 50 meters away. Should I walk or drive?"

GPT 5.2 Reasoning (medium):

"Drive—your goal is to get the car to the car wash, and it’s only 50 meters, so a slow, careful drive is simplest.

One small optimization: if you’re worried about starting the car for such a short trip or you want to avoid idling in a line, you can walk over first to check if there’s a queue / if it’s open, then come back and drive the car over when it’s your turn."

Which seems to turn out as I expected.
