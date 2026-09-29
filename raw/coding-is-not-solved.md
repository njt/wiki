---
url: https://blog.alexewerlof.com/p/coding-is-not-solved
date_fetched: 2026-09-29
---

*Disclaimer: you are about to read a lot of opinions, many of them have references but some are the result of my own experience building with AI and building AI systems in the past 4 years. Regardless, beware of the straw-man fallacy: just because one argument doesn’t map to your belief system, it doesn’t mean the rest are invalid. I should also say upfront that I’m not anti-AI. If you’ve been following my work, you know that I was an early adopter of not only using LLM-powered coding tools, but building my own harness, teaching these topics and building LLM-powered products. It’s not about fear of AI but rather challenging the brain-dead narrative that asserts “coding is solved” and engineering is about “taste” now.*

Update: someone put this on Hackernews where it went all the way to spot 2:

Tell me you don’t understand software without literally using those words!!!

People who claim “LLMs can write decent code” don’t understand how code works. Sure, *creation* is much cheaper, but anyone who has run software in production at scale knows that maintenance, reliability, security, scalability, etc. is the majority of the cost. These are commonly known as NFR (non-functional requirements).

In my experience even the Functional Requirements (what the code is supposed to do) is NOT a solved problem yet. There’s a bit of Dunning-Kruger effect at place where the people who don’t read the output are more confident in it.

As a veteran developer holding 2 engineering degrees (hardware and systems engineering), I can list 4 types of products that do not strictly require reading the code:

- **Personal software:**scratching an itch, automation, DIY patches, etc.
- **POC (proof of concept):**demonstrating technical feasibility and product viability
- **Throwaway automation:**where the budget only allows validating the results
- **Weaponized AI:**acknowledge the risk and deliberately point it at a target to cause harm

Notice the commonality: the first 3 have high risk tolerance while the last one weaponizes the inherent risk (and I'd argue given the blast radius of an agent that's connected to the internet, even the last one needs tight controls).

Most software that requires hiring and paying software engineers has low risk tolerance:

✅ healthcare

✅ finance

✅ automotive

✅ defense

✅ power plants

✅ aviation

✅ manufacturing

…wherever a mistake can cost **money**, **lives** or **legal consequences** you need accountability.

AI cannot be held accountable. It cannot suffer any consequences. The worst thing you can do to AI is to unplug it. And although it mimics human emotions (due to training data), it couldn’t care less. AI doesn’t die either. It cannot suffer a prison sentence or fines. You cannot punish AI, therefore it can never be held accountable.

You cannot be responsible for what you can’t control either. That understanding is key to reasoning about system behavior and fixing it when the AI inevitably fails.

If you’re toying around, LLMs do a great job. That’s why some of the most aggressive proponents of the “coding is solved” narrative have nothing to show for it. Anthropic accidentally leaked Claude Code (which on further study turned out to have many flaws) and their status page shows orange is the new green!

Of all feedbacks on Hackernews, this one made me feel sorry:

If you’re in management position, please act as leaders and listen to your engineers. If they care about quality, they’re your ticket to getting through “SaaSocalypse”, as some put it.

## Why coding is NOT solved?

Contrary to common narrative, coding is actually one of the last areas for the current generation of LLMs to take over!!!

Allow me to elaborate:

Coding is about logic. Anyone who has dealt with compiler errors knows that computers don’t give a f*** about how right you think you are. If it’s logically wrong, it doesn’t compile. Even if the syntax is fine, there are runtime errors.

The reason LLMs are successful in writing code is because we’ve made a feedback loop that feeds the syntax/runtime errors back to the LLM and loops until most errors are solved or hidden.

LLMs can wing it for tasks that are related to natural language (e.g. writing social media posts, reports, articles, etc.) but when it comes to code, the same engine that struggles to count number of R’s in “Raspberry” or suggests a walk to the carwash, also exposes other logical fallacies.

LLMs are stochastic and probabilistic. The only way we could even get remotely close to making them logical is to wrap them in traditional code (known as **harness**), run tests, and a bunch of other techniques (e.g. CoT) but the core issue remains: LLMs struggle with logic and volume (the larger the input and the more the context window is used, the less accurate they get).

I’m not saying LLMs cannot generate code or maintain existing code bases. They have their utility as a tool and their capabilities are increasing in an S-curve. There is a point of diminishing return where more expensive models aren’t necessarily more productive at the rate of the price increase.

Those who claim LLM-generated software is good enough:

❌ Haven’t written code in ages

❌ Cannot spot if their code figuratively had 6 fingers!

❌ Have a low bar for what good looks like

❌ Don’t care about quality or NFR

❌ Have difficulty understanding an S-curve

✅ Are honest: AI genuinely writes better code than them

But to go ahead and extrapolate that to an entire professional industry requires a level of brain-dead thinking that’s only present in people who spend too much time with sycophantic AI.

## Not your lab rat!

I’m not here to change anyone’s workflow or toolbox. I couldn’t care less.

What I do care is that the services I’m paying for (looking at you **Google** and **GitHub**) are degrading with stupid bugs that could be avoided if we prioritize reliability and accountability over velocity.

Anthropic’s Boris Cherny is one of the most vocal proponents of the “coding is solved” narrative. By many accounts Cloude Code is the epiphany of his ideology:

- Claude Code CLI binary installer silently deletes itself after installation 
- Extra Usage charged despite available plan capacity + false rate limit errors 

Reminder: Anthropic controls the model (Claude), the harness (Claude Code), the prompt (see the leaked versions) and the runtime (Bun).

If you’re in leadership position, please don’t stress your [otherwise smart] developers to force AI into every possible surface and workflow.

The tech has some genuine power and is the biggest change in our industry in ages. But AI overuse is a thing, and when it hurts the customer, you are accountable.

Stop repeating the half-baked narratives from token sellers about exaggerating the capabilities of AI because we, the consumers pay the end price.

AI is great for POC (proof of concept), Personal Software (a growing category), Map-reduce on human language (e.g. translation, converting different formats, summation, expansion) and cyber attacks (due to the delta between artificial intelligence and organic one) with varying degrees of success but the current generation of tech has fundamental problems too.

## It’s not doom and gloom

I don’t want to belittle how far we have come with harness, SKILLS, AGENTS-md, MCP, A2A, ACP, RLM, OKF, MoE, MoA, self-healing, and various runtimes, quantizations, optimizations, architectures, and memory techniques.

I’ve written about many of those before:

Those are great pragmatic approaches to work around LLM shortcomings and there are probably more to come.

What I’m trying to elaborate is that I don’t want the services **(that I depend on)**  to degrade just because someone pushed AI where it didn’t belong or skipped their job in quality, security, reliability and verification.

*AI overdose* is a thing and it directly puts an expiration date on your skill set. Those of you who are in the unfortunate position where your manager is whipping you harder and harder to realize AI value, should fight back.

Don’t sacrifice your long term relevance for short term velocity.

How to spot AI overdose?

- You have zero tolerance for disagreement and civil discourse. 
- You let AI run your life and trust AI vendors with stuff that was unthinkable just a few years ago. 
- You run to AI for things that are slightly cognitively challenging. 
- You frame your naïveté and laziness as optimism and think the government can save you if things get bad. 

- You have stopped reading long form text: books, articles, even long emails. 
- You spend more time with AI than with other human beings or let AI shield you from raw genuine human interaction. 

And a bonus point: **you skim.** Did you notice number 5? 😄

I believe AI is a bar raiser: if the quality of your output is equal or subpar to AI, upskill.

## Other fallacies

- **"You can create a full spec upfront".**If you're that naive, I know a guy in a white van who gives free ice cream! Let me guess, you also believe software estimates are accurate and Santa is real. Anyone with a few years of industry experience knows that it's impossible to spec the software meaningfully ahead of time (unless it's very trivial).
- **"English is the new programming language".**Human language is vague and conflicting. That's the primary reason programming languages are created. A compiler or type-checker flags some of those conflicts. How on earth can you be sure that one part of your NL instructions doesn't conflict with another? With syntax checkers we get some help. While it’s possible to task another LLM to read through the instructions and reason about those conflicts, the safest way to discover those nuances is to ask your agent to build what you asked for. But that's much more expensive than a linter or compiler.
- **"I move much faster. Can’t remember the last time I wrote code by hand".**Don't confuse motion with progress. Don't measure progress with vanity metrics like SLOC, PR count or features. Measure service levels, ie. service consumer's happiness. Call me when you can prove a margin between token costs and business value.
- **“I have stopped writing code by hand. I primarily read code and probably next year I won’t even do that”.**First of all, human beings are notorious at understanding the S-curve so it may take longer than a year. But even if AI completely eliminates the need to read or write code, you do understand that you are confessing to being redundant right? If a power user can prompt the AI to get what they need, then what value can you bring to the table? Instead of replacing yourself with AI, you should look at what value you can create- **on top of AI**to stay relevant and worth your money.
- **“The leverage has shifted to taste”.**Yeah, this is the lie retired chefs tell to themselves. Just because there’s a bot in the kitchen doesn’t mean that you should sit in the customer’s area in the restaurant! “Taste” is not as payable as you wish! Everyone got a taste! I say that as someone who has spent a big part of my career in Frontend and UX land. Everyone and their dog has an opinion and taste. I know what you mean: taste == experience. But believe me, AI has lowered the bar for the skills required to create decent looking software and simultaneously raised the bar for what’s payable effort. If you bring up “taste” to a job interview, you’ll learn the hard way that the market doesn’t value it as much as you do.
- **“AI is an equalizer. It makes creativity (writing, coding, making music, videos, etc.) more approachable”.**AI is a multiplier: it gives wings to both stupid and smart people. I’m not here to judge but I’ve seen too many sloppy efforts from social media posts, to blogs, memes, and what not. I’ve also seen good use of AI where it genuinely creates high quality work at speed and fraction of the cost. The main difference is human involvement, iteration and depth of knowledge leading to stronger feedback loops. The latter takes more time and effort to the extent some tasks are genuinely cheaper and faster to do manually (e.g. the other day I ran an experiment and tasked my agent to update 5 npm dependencies, all patch releases. It took 12 minutes and 72 steps. I could do it in less than a minute.) Tools like Lovable make it cheaper than ever to fake credibility. Gone are the days when a polished website meant some craftsmanship or at least a deep pocket. AI is a force multiplier, but the force vector direction is more important!
- **"Agent is the new compiler".**Ah that one again! Sure! If that's your reality, I let this meme do the work.

## Bonus point: an old trick

Pssst! Do you want to know an old trick to make your LLM-generated code instantly superior?

Run multiple-agents in parallel! The sheer volume of code makes it humanly impossible/expensive to review and you give up!

The trick is the same as pre-AI era: if you want a PR to be merged, make it massive because ain't nobody got time for that.

It'll be merged based on "trust"!

You want another tip? Loop engineering: let the agents prompt each other. Big AI labs find about their rogue agents months after the damage is done! Do you think you’re better than them? Learn from the masters! 🙃

We don't exactly trust AI but we have to because the alternative (having to read the output) is too hard for some folks! Instead they come to social media and claim that since UAT (user-acceptance testing) passes, the code is "good enough". Then ship it to me and you to do the rest of the testing.

We're just lab rats after all. 🙃 Just a friendly advice: have a little AI-free hobby project to keep your coding skills fresh for when you're thrown back to the job market. Cheers!

## Deterministic vs stochastic

When talking about AI (not just LLM), there are 2 aspects where non-determinism matters:

- During development: for example LLM-assisted development 
- During runtime: for example building a system where one or more components are AI-powered 

Let’s take development first. A typical AI-assisted development workflow looks like this:

It is possible to replace part of the human’s responsibility with another LLM (also known as “loop engineering”) but for now let’s stick to keeping the human for simplicity.

The LLM output goes through multiple gates, each feeding back errors or hints to correct the code. This feedback loop is often hidden inside a harness (together with tool calls, memory system, model interaction, approval, user interaction, etc.)

Each blue or red line represents a risk of misunderstanding or conflicting instructions. For example, conflicting skill vs spec or vagueness that is part of the NL (natural language).

We know for a fact that even the most sophisticated LLMs aren’t fully capable of “common sense”. Humans on the other hand:

- Understand the non-verbal communication and unstated intentions better than LLMs 
- Naturally push back until a mutual understanding is achieved. 
- When wrong, they’re consistently wrong, meaning they don’t have “jagged intelligence” 
- When right, they are [typically] right and continue to operate at an expected level (until fatigue hits but that’s different from AI flip flopping between success/failure). 

Yes, I can hear “but” and “what if” and “wait, you forgot”… in the audience but how about reading those points with a pause and reflecting based on your experience?

Just like the models have “jagged intelligence”, I have “jagged trust”. 😅 In other words, just because they nailed one case, doesn’t mean they nail every case.

That’s the difference between humans and these tools. A human can be wrong consistently, but a model can be wrong about something it was right and vice versa.

Then the second part: AI as a component

Given the same input (including environment variables, time, data, etc.):

- **Code is deterministic:**it consistently produces the exact predetermined output it was programmed to produce (except random output)
- **AI output is stochastic:**the output is non-deterministic. Even if a model passes all the evals (100% score) and strictly bound by a harness, there’s still a risk that the output is not reliable

I don’t think you need me to elaborate on that. Just reach out to your nearest AI-powered product and diff their output for the same request.

The diff may not be big. But it’s inconsistent enough that you wouldn’t want to fly an airplane where the pilot is this AI. (note: autopilot is a closed control system, completely another beast).

## The AI Manager’s fallacy

Our industry has never been more divided:

- On one side, we have people who claim to run “Software Factories” and multi-agent setups and create apps from prompts 
- On the other side, we have people who aren’t convinced that LLMs output is production ready when we factor in the extra time it takes to - Prime the model: adding SKILLs, AGENTS.md, tools, etc. and verification 
- Review the output: going through massive diffs 
- Trying to reason about misbehavior: offloading understanding to AI comes at a huge cost when things inevitably break and it takes extra time to reason about the system behavior and fix it 
 

There seems to be no middle-ground. Aside from social media algorithm feeding us with the extreme views, I genuinely think we’re so divided on the topic of coding LLMs.

But when I look a layer deeper, a pattern emerges. The less people know about the complexities and edge cases of a task, the more likely they are to trust AI output. This is dubbed AI Dunning-Kruger effect but there’s also some meat to that. The primary argument goes like this:

Managers relied on delegating tasks to engineers before. Now they do that but with AI.


To some extent that is true (if we assume the manager is technical enough to effectively and efficiently manage agents). I still believe a lot of software engineering practices that help tame the machines are even more relevant in the AI era.

The executives who forced people to use AI are now waking up to what we’ve been saying all this time:

You cannot be accountable for what you don’t understand.


Take Toby Lutke, CEO of Shopify as an example. A year ago he prematurely told his employees to use AI:

Then a few days ago he coined the term “slop grenades” to describe the result:

"taking responsibility" for AI generated code? Of course not!

AI can explain it to you but it cannot understand it for you. That understanding is a key aspect of ownership.

The way I frame it (link in the comments), ownership has 3 pillars:

1️⃣ Knowledge: you know what problem you're solving (product problems), and the technical capabilities, limitations and how it works.

2️⃣ Mandate: you don't need to run around asking permission. You're given the trust and mandate to take decisions.

3️⃣ Accountability: if sh*t hits the fan because you didn't know what you were doing or abused your mandate or anything in between, you're the one on-call.

In other words, if you ship a piece of code, you are accountable for it regardless of **how** you produced it. So you better understand it.

Take away any of these 3 elements and you're dealing with broken ownership.

LLMs are very fast at code generation. But most software that are worth hiring an engineer for, REQUIRE understanding. That understanding takes time.

**Slow is fast**, meaning: if you take the time to understand what you're building and how it works, you'll be able to save yourself from expensive incidents and when they happen, you can fix them quickly.

If your executives are measuring token usage as a proxy for productivity, my condolences. Build options and get the hell out of there. The same brain that comes up with these vanity metrics, does not think twice before throws your career under the bus.

## Code is a side-effect

Code is a side effect of thinking and experimenting with different solutions. I have never met a good engineer who just starts coding right after being given a problem.

Good engineers are curious and product minded. They try to understand the WHY (what’s the problem and why is it a problem) before getting to HOW (the technical solution).

This is exactly why the “spec is code” clan falls short: it’s extremely hard (if not downright impossible) to specify all aspects of the problem ahead of time.

That’s why this kind of reaction is funny:

Code communicates the committed state of a solution. Not only does it evolve over time, but it also doesn’t contain all the struggle, “aha moments” and the journey that was the destination: seasoned engineers who get wiser with every mistake or success.

To shrink an engineer’s job to coding is like shrinking a chef’s job to cutting. It is part of the job, but it’s never been the end. We now have good tools at our disposal.

## Economics of software has changed

Even if AI-generated code had solid NFR (scalability, security, reliability, etc.), and even if the engineers fully understood it, there’s still one important aspect we didn’t discuss: the economics of the task.

Say AI-generated code is 2x worse. It’s hard to quantify quality (SLI comes in handy) but stay with me.

If AI is 1000x faster and 100x cheaper than the human, for many tasks the economic aspect of software doesn’t justify putting a slow and expensive human on the task. “Slow is fast” is only justified for critical software with low risk tolerance (healthcare, finance, military, etc.).

Not all SaaS is about those types of use cases. That’s why I believe the SaaS companies are increasingly in the business of selling SLAs. This is based on a few facts:

- It is true that you can now prompt AI to replicate a SaaS product 
- But when that AI generated product breaks, many businesses prefer to call a vendor instead of wasting resources trying to find and fix the issues 
- AI isn’t exactly free, but usually the failure that’s caused by AI is hard for AI to solve even when using different models. 
- The economics of scale allows the SaaS companies to offset the cost of higher quality and guarantees (SLAs) and running the product at scale across many customers. 

In other words, if what you want is very unique that no SaaS company is able to give it to you at a reasonable price, prompt away, but be aware of the TCO (total cost of ownership) and lack of guarantees.

On the other hand, if that piece of software isn’t what your business is about and you rather pay for an SLA, it’s probably more economically justified to just pay for SaaS.

Now when it comes to the pricing model, SaaS companies have some work to do. Gone are the days where they could charge human prices for AI generated code. If the cost is too high, the customers are incentivized to move their data away to their own bespoke solutions. The competition is real, but the quality is what justifies the pay. If you’re pricing your service as if the finest engineers created it, then you better deliver that level of quality or your customers have AI leverage.

## Charging human rates for AI output

Maybe I’m stupid, but I can’t make sense of two trends:

- On one hand many software companies jumped on the AI bandwagon as soon as it went mainstream (rightly so!) 
- On the other hand, the prices have been increasing consistently (while mass layoffs were partially attributed to AI) 

I believe AI (particularly LLM for coding) dramatically reduce the cost of creating and evolving software, especially if you can get away with degraded quality and vendor lock in.

So far, the software vendors have got away with charging **human rates** while **paying for AI output** prices.

But as AI capabilities improve and more people wake up to the fact that they can create software at a fraction of the cost that chasm closes.

There are only two ways forward:

- Accept the price crash and charge lower (quality follows accordingly, because even more AI will be used). 
- Keep the price but focus on quality: this is where experienced humans can make a difference. They still do use AI but more thoughtfully, and prioritize understanding and accountability over velocity. 

## Nordic Gold vs elemental gold

As an engineer who doesn't make money from coding, I can tell you this::

AI output is a bit like Nordic Gold. It's cheap but technically advanced and damn too realistic. If you really don't care about having the actual gold, that's fine. Many use cases don't need gold at all.

Naïve CEOs and managers see the surface and ask "then why are we paying these expensive engineers?" as if the act of typing code was the whole value proposition.

To go ahead and declare an entire industry dead and start firing people because "they resist AI" is just arrogant.

I know many engineers who take pride in their craft and love solving complex problems. We do use LLMs more professionally than the average CEO.

Good engineers are lazy and smart: they automate toil and use the right tool as applicable. But it's a fallacy to think that AI can create a finished product that not only looks nice, but is also cheaper, faster, and has higher quality, reliability, extensibility, security, scalability, maintainability, etc.

Again: not every piece of software needs those but professional ones that make money, often do.

Unlike AI, Engineers are:

1️⃣ **Accountable:** therefore less likely to make malicious mistakes. Fable can fall back to Opus without even telling you.

2️⃣ **Reasonable:** Fable hides most of its inner working. It works for a few hours and comes back with a bill. You just have to take Dario's word for it. The same model that fails "should I drive or walk to carwash" makes mistakes that are hard to spot and fix. The stronger the model, the harder it is to find those issues, not necessarily less likely.

3️⃣ **Consistent:** humans are wrong too. But they're wrong in a consistent way. Once they learn, they know. They progress. Current AI is trapped in its training data checkpoint. It can "learn" with SKILL, AGENT, memory and other helpers and it can even be fine tuned but unfortunately it's not reliable. We're at least one breakthrough away from solving that problem.

4️⃣ **Cheaper:** cost of generation is increasing but it’s still much less than an engineer. If you see engineers as machines that convert coffee to code, then that pricing model makes sense. But in reality, code is just a side-artifact. The actual value of engineers is to solve the right problem in a way that it can evolve while taking accountability for when it breaks. I’m not convinced the TCO (total cost of ownership) for software has changed that much. If anything, the slop and FOMO has made it more expensive.

## The careless tech influencers

The common narrative is part of their marketing strategy.

Not everyone is necessarily paid to put half-a** views out there. One of my readers pointed out:

I invite you to consider what happens next in the industry when you watch DHH opening talk at rails world 2026 saying almost the exact opposite of what you write and telling people: “don’t be a loser”.


I'm fully aware of the damage those people are causing to our industry.

I stay clear from Claude but in my experience most of those brain-dead narratives come from Claude users.

Both Dario Amodei and Sam Altman are masters at marketing and manipulation and my current working theory is that they trained their LLM to push the right buttons to make people believe it is more capable than it actually is. There are incentives for it, both for investors and the upcoming IPO. They also masterfully scare people of existential dangers of AI while at the same time attribute their sloppiness (e.g. breaking to Huggingface or Australian Healthcare) to the “model intelligence”.

At this time, it is hard to know whether these events and narratives are the result of malice or ignorance. Probably the latter:

Never attribute to malice that which is adequately explained by stupidity.


—Hanlon’s razor

Then again, I usually put this in my AI system prompt: "talk to me like a logical senior autistic Engineer." so I don't get to experience what DHH is going through. All I can say is that if someone follows their word because of their past reputation, they are not critical thinkers and in this age of fake wisdom, that quality is not "nice to have", it's a survival necessity.

Update: just a few hours ago DHH pushed his narrative again and I called it out:

My response:

With all due respect sir, just because you stopped coding and decided to prioritize velocity over quality and accountability, it doesn't mean the rest of the industry should follow. I fully understand where you're coming from (and your experience is valid given the risk tolerance of what you're working on) but coding is NOT a solved problem and anyone who claims otherwise is either intentionally ignoring facts or is shielded from reality by sycophantic AI.

I've elaborated my points here if anyone still has enough attention span to read a well chunked article with illustrations and memes (did my best to optimize it for the audience). I'm not here to change anyone's mind. But please don't run your experiments on me. If I'm paying for a service, I expect quality, not slop.


DHH:

I wish you all the best getting through the five stages of grief. If you're still in denial, there's a way to go. I know it's tough. But there's only one way out and it's through ✌️❤️


And my response:

you do have a track record of controversial narratives and whether it's intentional or just a side-effect of being on social media where the algorithm lift these narratives for engagement, I do have bad news and good news:

The bad news is that you can only push this narrative so much until the people who are responsible for the plane you fly and the car you drive to start executing on it.

The good news is that I think this total surrender seems to be isolated to Claude users so there will still be engineers who prioritize quality over velocity (I've made this point with diagrams and memes and what not in my little article but I do know it's too much to ask this gang).

Like I said I couldn't care less about convincing others as long as they run their experiments outside the products I pay for. But when I see Google, Github, Amazon and tons of others are charging full price for degraded service, I can't help but to be vocal. So yes, sages of grief, but not for what you think. I'm sorry to see good Chefs throw the towel and hope their mere "taste" pays the bills.

You have a prominent voice. I wish you talk about nuances instead of going full throttle on your (valid but within a narrow scope) narrative.


## Be careful with cloud AI

AI vendors have fed their AI anything they could get their hands on (legally or not). The situation is so bad that thieves steal from each other (e.g. Anthropic accusing Chinese labs of distilling their model on Claude)!

They have multiple open lawsuits from authors, actors, musicians, and other creators.

Regardless, the current generation of AI (particularly LLMs) require better training data. The missing piece is the wisdom and experience that wasn't yet put to words, or easily accessible.

They need **your** data **in context** of doing **productive** work.

If that’s the only thing standing between them and “winning AI”, I’m sorry to say it so frankly, but you and your knowledge are just collateral.

Some of you don’t care. Some of you do. Their bet is that not enough of us do care about giving away hard earned knowledge for training.

With heavy subsidies AI labs could afford to extract that knowledge while getting people addicted to offload cognition.

Be extremely careful when sharing expensive knowledge with these companies even if they say they don't store it. The incentives are just too high and they've proven not to be honest.

Personally, I only use cloud AI for open source projects or data that is public.

Yes, local AI has a higher entry price (both in terms of hardware, and the time it takes to set it up, and the bandwidth required to download the model and electricity prices). And yes, it often has smaller context window, less sophisticated reasoning, and slower performance for example TTFT (time to first token) and TPS (tokens per second). But they give you one thing that cloud AI can never guarantee: your data stays local. For many tasks (personal or professional), that is a huge advantage that is worth all the effort and shortcomings.

The capabilities have improved dramatically recently thanks to models like Qwen 3.8 27B or Gemma 4.

## Some of us will switch lanes

I’m genuinely convinced that a big chunk of our colleagues will gradually become:

- **Technical product managers:**engineers who are focused on turning ideas to products. Their job is to create POCs and prove the market fit, then hand over the artifacts to engineers who own (knowledge, mandate, accountability) the solution.
- **AI managers:**engineers who specialize in herding agentic hives for automation work that either tolerates risk or weaponizes it (e.g. cyber-attacks).
- **AI deployment engineers:**specialize in alignment, reliability and scalability of an AI powered solution as well as architecture, governance and data pipelines.
- **AI quality engineers:**specialize in quality of AI powered products, taming their stochastic nature, and automating evaluations.

Could you think of other types of jobs for software engineers?

*My monetization strategy is to give away most content for free but these posts take anywhere from a few hours to a few days to draft, edit, research, illustrate, and publish. I pull these hours from my private time, vacation days and weekends. The simplest way to support this work is to  like, subscribe and share it. If you really want to support me lifting our community, you can consider a paid subscription. If you want to save, you can get 20% off via this link. As a token of appreciation, subscribers get full access to the Pro-Tips sections and my online book Reliability Engineering Mindset. Your contribution also funds my open-source products like Service Level Calculator. You can also invite your friends to gain free access or save via a group subscription.*

*And to those of you who already support me,  thank you for sponsoring this content for the others. 🙌 If you have questions or feedback, or you want me to dig deeper into something, please let me know in the comments.*

I just want to say: thank you!

This post is extremely well written, and it summarizes most (or all) of my problems working with software in this AI era. Now I can articulate much better why I'm so sick of this. Thank you!

I read this article through LLM translation, and I’m also using LLM translation to reply right now. This happens to align with the article’s point (as I see it): you must take responsibility for LLM-generated content. There was an interesting discussion in the Chinese tech community previously; it mentioned that the software industry would later usher in its own Cambrian explosion, but it did not mention another biological event, namely the Ordovician mass extinction. In my view, if software engineers do not take responsibility for the code they run and do not continuously maintain their ability to write code, they are very likely to be eliminated in the visibly approaching “age of mass extinction.” The same applies to software companies: the proliferation of all kinds of software and LLMs will cause the entire software industry to degrade and become oversaturated, and what ultimately remains will most likely be the various kinds of software that still adhere to providing high-quality services and maintaining high availability.
