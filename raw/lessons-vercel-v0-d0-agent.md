---
url: https://gist.github.com/8f638d27df4504ece90af0623ff6ce1a
date_fetched: 2026-07-05
backfilled: true
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Agents are easy to build; don’t buy, build. Vercel’s internal data agent (Dzero) and V0 were built in-house. The thesis: building agents is trivial, and you should just do it yourself rather than purchasing a solution.

Simplicity wins when models get smarter. Dzero’s initial architecture used many tools in a loop. They threw it away and rebuilt it as a two-tool agent (bash + Execute SQL) that reads a YAML file describing every Snowflake column in plain English. The agent uses grep/tail to explore semantics and writes SQL. Result: 50 lines of code, transformational.

Make your problem look like a coding task. Models are disproportionately good at coding because that’s what they’re trained on. If you can frame a non-coding problem (like text-to-SQL) as a coding task, you get outsized results.

Product-market fit shifts with model capability jumps. V0 started as a frontend tool, became a backend engineer’s assistant, then a full-stack app builder for non-engineers, and now serves “tech-adjacent” roles (PMs, designers, business people) and internal tool builders. Each pivot was forced by a model leap (e.g., Sonnet 3.5 enabling full-stack generation).

The “Tailwind moment” was serendipity. Early models couldn’t handle separate CSS files. Telling the model to use Tailwind (inline styles) made it work because the model had enough training data and could reason inline. This insight made V0 viable in 2023.

Shipping culture: optimistic locking, no approvals. Anyone can ship anything, but they must announce it; the organization can veto. This empowers teams and speeds up the outer loop. Legal, for example, only steps in when something is truly problematic.

Reliability and speed aren’t opposites if you design for it. Vercel ships its control plane on every push to main, but serving systems only once a day. Serving regions are autonomous and changes roll out in waves, preventing global config-driven outages.

Unlimited tokens for devs is a no-brainer. It’s in the job description. Cost is negligible compared to productivity gains, though you should watch for cache-breaking prompt changes.

The job of an engineer is becoming management. Senior ICs benefit most because they already orchestrate work—now they have more “minions.” Juniors thrive because they’re digital natives. The middle is the interesting, uncertain space.

Software is free, like a free puppy. Making software is approaching zero marginal cost, but maintenance is the real burden. The market will produce vastly more software; whether that means more or fewer engineers depends on how elastic demand is and how maintenance is handled.

The transformation is more like the mainframe era than the internet. The 1960s shift eliminated rooms full of “computers” (human calculators) but ultimately made everyone richer. Today’s shift may be similarly painful for some but beneficial overall.

Pithy and provocative quotes

“Building agents is actually extremely easy and you don’t need to buy them, you can just build them yourself.”

“In the world of agents you have to be humble in the sense of we’re just discovering how to build them. … Just because something was best practice in the summer of 2025 means quite little today.”

“If you can make things look like they’re a coding task, even though they’re not, then you get disproportional good results.”

“I’m deep in 12 hours a day coding. And by coding I mean I can do meetings. … I can just kind of every so often optimize the go in, check the prompts, deliver code, review.”

“The new crazy onslaught of shadow IT is definitely something that we’re very interested in.”

“We automated 87% by now of our support intake. … Did we let support agent go? No. … They have a much better job now because they only have to solve the hard problems.”

“Software is free, like getting a free puppy. It has to be maintained.”

“The most senior ICs are in a way the one that benefit the most because they’re they kind of already did a very similar job, but now they have just more minions.”

“The other folks who benefit the most are the most junior engineers because … they’re better at making TikTok videos than me.”

“I think what’s most interesting is kind of what happens in the middle.”

“We just quote unquote, decided that we cap employee count at 1024. … I could see that … maybe you actually don’t need more people.”

“The pattern that we use at Vercel is what in the nerdiest way you could describe as optimistic locking. So there are no approvals. Anyone can ship anything, but they have to tell the organization that they’re going to do it and the organization can veto things.”

“I would tell someone, go figure out what the product is and give me a demo. And if the demo is good, I’m going to give you a team.”

“I think the big difference is that today you actually don’t need a team. … Teams just, I don’t know, make things go slower.”

“I think there’s a whole revolution ahead of us of agents participating further in the DevOps space and running your application. We started calling it kind of self driving infrastructure.”

“The transformation is probably closer to the one in the 60s when the mainframes were introduced and the buildings full of quote unquote computers no longer had a job then. It is like the transformation of the introduction of the Internet.”

Tools, practices, and methodologies

Two-tool agent architecture (Dzero): A text-to-SQL agent with only a bash tool and an Execute SQL tool. It reads a YAML file containing plain-English business descriptions of every database column. The agent uses grep/tail to explore semantics and writes SQL. Why it matters: extreme simplicity leverages model intelligence; complex tool-loop architectures become unnecessary as models improve.

“Make it look like coding” pattern: Frame any problem as a coding task (e.g., exploring YAML files with bash, generating SQL) to exploit models’ disproportionate strength in code generation. Use this when designing prompts or agent scaffolds.

Optimistic locking for shipping: No approval gates; anyone can deploy, but they must announce their intent, and any stakeholder can veto. This shifts responsibility to the vetoer to be active, eliminates waiting, and speeds up the outer loop. Use it to balance velocity with oversight.

Unlimited AI tokens for engineers: Give every developer unrestricted access to AI coding tools. Monitor for accidental cost spikes (e.g., breaking prompt caching), but otherwise treat it as a trivial expense relative to productivity.

Wave-based, region-autonomous deployments: Ship infrastructure changes one region at a time; keep regions autonomous so no single config change can cause a global outage. Use this to get fast feedback on changes without risking widespread failure.

Separate shipping cadences by risk profile: Ship the control plane continuously (every push to main), but ship the serving stack only once a day. Deliberately trade off feature velocity for reliability where it matters.

Tailwind-for-prompts insight: When generating UI, instruct the model to use Tailwind CSS (inline utility classes) rather than separate CSS files. Early models handled inline reasoning far better; this pattern may generalize to other domains where co-location of logic reduces model confusion.

Self-driving infrastructure (concept): Agents that understand running applications and participate in DevOps tasks. Still nascent, but the speaker sees it as a major upcoming shift.

Agent for sales lead qualification and support automation: Vercel built and open-sourced an agent that qualifies sales leads; they automated 87% of support intake. The practice: automate routine work, let humans focus on hard problems, and don’t cut headcount if the company is growing—just upgrade the role.

Solo exploration before team formation: To start a new AI product, have one person (or a tiny group) find product-market fit and produce a demo. Only then allocate a team. This avoids premature scaling and keeps the initial loop fast.

Unanswered questions and omissions

The hidden complexity of “simple” agents. Dzero is described as 50 lines of code, but the YAML file with business semantics for every column required massive upfront work. How do you create and maintain that semantic layer at scale? What happens when the schema changes?

Optimistic locking’s failure modes. The veto mechanism sounds elegant, but what if a veto comes after a damaging deploy? How do you ensure stakeholders actually review announcements in time? What’s the blast radius of a bad change that slips through?

The middle of the engineering org. The speaker says the most interesting dynamic is what happens to mid-level engineers, but offers no insight. How do you develop senior ICs if the middle is squeezed? What career paths remain when junior engineers are hyper-productive and seniors act as managers of agents?

Maintenance at scale with agents. The “free puppy” problem is acknowledged, but no concrete strategies are offered for using agents to handle maintenance. What does agent-driven maintenance look like? How do you ensure quality and avoid compounding technical debt?

When to throw away vs. iterate. The talk describes multiple pivots triggered by model improvements, but doesn’t share the decision-making framework. How do you know when a model leap warrants a full rewrite versus incremental adaptation? What signals do you watch?

Observability of long-running agents. The interviewer asks whether the lack of observability is temporary or fundamental. The speaker says a revolution is coming but doesn’t address the current gaps. How do you debug, monitor, or trust an agent that runs autonomously in production?

Competitive and vendor risks. The speaker uses Claude and Codex personally; V0’s architecture presumably depends on specific model behaviors. What happens if a model provider changes pricing, deprecates a model, or alters behavior? Is there a multi-model strategy?

Quality and error rates in automation. 87% support automation sounds impressive, but what’s the false-positive rate? How do you measure customer satisfaction when a bot handles the intake? Are there classes of issues the agent consistently mishandles?

From solo prototype to team product. The advice to start with one person and a demo is compelling, but how do you transition from agent-generated prototype code to a maintainable, team-owned codebase? What practices prevent the prototype from becoming a ball of mud?

Ethical and workforce implications. The mainframe analogy is offered, but the talk doesn’t grapple with the near-term displacement risk, the need for retraining, or the societal impact of “maybe you actually don’t need more people” while revenue grows. The cap at 1024 employees is a joke, but it hints at a real tension that isn’t explored.

Speaker A: Malte, thanks for having the conversation with us today. This time it feels different when it comes to coding agents. I think there have been times in the past where it felt like it was a little revolutionary, but I think this time we're all trying to figure out how we are supposed to build what while the ground underneath us is shifting. And so super happy to be speaking with you today. I think that with your products, DZero, your internal data agent, and then Vercel V0 as well, your public facing thing, you've been able to roll with the punches and be able to build and create products as the world has changed around us. Let's start with DZero, your internal data agent. What was your first approach there and why did it turn out to be so flawed?

Speaker B: Yeah, to give folks a little bit of context, the thing that makes Vercel strong is that we always build things with our own technology. And one of our thesis is that building agents is actually extremely easy and you don't need to buy them, you can just build them yourself. That's why we build one ourselves. And so the agent that we call Dzero internally is a text to SQL engine. So you give it any question in Slack and it will reply with the answer and it has access to our entire Snowflake subject to the access rules of the user. I'll share one query which I think is the funniest one. It's a salesperson, you'll hate it all. Earlier this year this person asked the following question which S&P 500 CTOs and VPs of engineering have private Vercel accounts and have deployed over Christmas. So presumably they got an email or a call, I don't know, but I think that's if actually finding that from. I mean obviously you have to do a little bit of research, you have to go to LinkedIn, et cetera, the agent does as well. But then eventually you kind of figure out the right snowflake query. It's very, very difficult. And so the initial version of this agent was kind of this kind of traditional architecture where you have the agent as a tools in a loop type of thing. All kinds of different tools to do all kinds of different things. And it wasn't like it was working badly, but it was maybe not as magical. And then we deleted everything and switched to an agent that looks very similar to a coding agent. Now it's still kind of custom coded, like it doesn't use for example cloud code as a harness because again, building such a harness is actually really easy and you can just do it, but otherwise it kind of looks the same way. So the general architecture is that we did the work of going through our entire snowflake and for every single column in prose explaining the business value and that gets export it to a YAML file and the agent essentially gets told, you got post trained on using grep on tail and all these things that you need for coding go to town on these YAML files to figure out the business semantics of everything. And then you make a SQL query for that. It has a completely different custom tool. That's it. So there's just two tools. There's the bash tool and there is the Execute SQL tool and that's the entire agent. So it has like 50 lines of code and it's completely transformational for the business.

Speaker A: What led you to understand that you needed to throw everything away? Because I feel like you go and you build all of this code and it's just so hard to light it on fire and then start from scratch again.

Speaker B: Yeah, I think, I think in the world of agents you have to be humble in the sense of we're just discovering how to build them. And so just because something was best practice like in the summer of 2025 means quite little today. And that's different from, I don't know if you've built a website, it's been 30 years and we kind of know how to do it. But agents are in their maybe third year for real, if at all. Right. And so you have to be willing to entertain the notion that, yeah, maybe there's actually not a better way to do it. And it's kind of proportional to the models getting more intelligent. So it's more viable now to have agents that are essentially fully relying on the emergent behavior and respectively are more simple because you don't have to prompt them so much, you don't have to hard code so many rules.

Speaker A: On the Vercel blog you had a. A post. It was titled I think all you need is the file system and bash is something like that. And so you just took a complete step backwards and described what the data was in plain English and sort of like you just step back and let it cook and move to a much more declarative model with everything.

Speaker B: Right? Yeah. The intuition is that you have to think about like what was the model trained on, what was optimized on for now there's lots of coding tasks. Right. And it'll change in the future. And so if you can make things look like they're a coding task, Even though they're not, then you get disproportional good Results.

Speaker A: How many CEOs and CTOs in the Fortune 500 are using Vercel on their personal accounts?

Speaker B: I don't have the precise number. It was pretty substantial. Okay. Yeah. Dozens of them.

Speaker A: Well, I just. It is, it's quite amazing now to see all of these CEOs and CTOs come out of the woodwork and start coding and doing all these agents. Are you one of those CTOs that are close to the ground and shipping stuff?

Speaker B: I was already before, but I'm deep in 12 hours a day coding. And by coding I mean I can do meetings. Right. Because I can just kind of every so often optimize the go in, check the prompts, deliver code, review. Yeah, I'm super back. I've been for the longest time kind of oscillating between coding a lot and not coding so much. And I'm back on like 24 7.

Speaker A: What's your drug of choice, your agent of choice?

Speaker B: So for Vercel, we try for folks to get a broad experience across the company, want to feel what our users feel, so we don't mandate anything. My current stack, but it's subject to change, is Claude with 4.6 fast and then I'm using codecs 5.3 for code reviews.

Speaker A: Yeah, it's quite amazing. It's just this dopamine hit where you're hitting the slot machine hoping that you get something awesome out of it. Right.

Speaker B: By the way, there was an article in the Wall Street Journal of me basically saying that. And so that was my first ever quote in a major newspaper. Here you go.

Speaker A: Oh, okay. Oh, wow. We both came to that, voted to

Speaker B: be addicted, but yeah, maybe I'm not addicted, but I do love it. It's such a great experience.

Speaker A: Well, let's shift to Vercel V0, which I think you launched back in 2023, is that correct? Yeah. So the original pitch was anybody should be able to make an application, even non engineers, and you could do some prototyping and stuff like that, which was I think pretty revolutionary at the time. What did you think v0 was going to be and how quickly did it start diverging from the plan?

Speaker B: Yeah, I think initially we thought we were building a tool for front end engineers because that's what we're doing in general.

Speaker A: Right.

Speaker B: And so like we're thinking we were making a tool for ourselves and we then realized we actually built a tool for backend engineers because it was empowering them, but it also Sucked in the sense that it doesn't work all the time. You know, this is 20, 23 GPT, 3.5 GPT, four days. And so it didn't work all the time. But the back engineers, they were good enough to fix it in the cases where it didn't work. But you couldn't have given it to a non engineer at the time, right? Because it was just not reliable enough. That was the main product market fit for a while. But again in the AI space you have to be humble, you have to be willing to entertain that the world is changing around you. So sometimes people call it pivot, but it's not really the right word because usually that's used for the cases where you realize that you kind of made a mistake on your internal data and you have to do something else. Whereas let's say anthropic ships, Sonnet 3.5 and suddenly essentially the same prompt can be used to build full stack apps which previously just wasn't possible. You cannot now keep your product the same because the world is different. And so that was like a major milestone where we, we realized okay, we have to. And I mean obviously this is amazing, right? Like we can now adopt our product to be able to build like end to end applications and also with a success rate that kind of approaches the case where a non engineer can be successful.

Speaker A: So you told me There were about five moments when it came to V0 over the years where the models took a really big step forward, forward or usage patterns shifted and you needed to rethink things. Can you take us through one or two of those moments and how you were sort of thinking about things?

Speaker B: Yeah, I mean definitely the first one was like in the summer of 2023 when to take us all back, right. The ChatGPT had launched and LLMs were still kind of new and broad adoption and these things were making text. And so I think us and everyone was thinking like we gotta like it must be possible to make web pages with these things. And back in the day when you tried, you would realize that it wouldn't do a good job. Like it would kind of do it, but like it wouldn't look good, it would kind of be subpar, not something you would package as a product. And I still remember to this day sitting at the, like we had someone actually who is from China and lives in Berlin, but he was in our office in San Francisco and he was hacking away and at some point he was raising his hand like guys, guys, guys, guys, I found something. And so he had Just changed the prompt to say use tailwind. And this was such a serendipity moment because tailwind at the time, which is a way to write css. At the time it was old enough that it was in the ChatGPT 3.5 training kickoff, but at that time it wasn't super popular yet, but there was enough training data. And so the model is so much better at doing inline reasoning rather than saying have to write a CSS file, then later write CSS HTML. The models at the time are too dumb to do that, so they couldn't. But if you told them basically just put everything in the same place, then they did a real job. And so that's kind of how we figure out how to build a product that actually was viable back in the day. So that was kind of the first milestone to actually make it work eventually. I already mentioned that the second really big pivot was that I think the final step, obviously there was this whole ecosystem now of products kind of in this space and some of them went very much into the consumer space. And we definitely discovered, okay, there's this enterprise use case. So for V0, the users we are aiming for, I would call tech adjacent. So it's the pm, it's the designer, it's the product owner, the business person who's kind of interested that found a lot of product market fit and then the final use case. And I'd love to talk a little bit more about that, but it's essentially, I'm a business person in a company, I'm going to make an internal app for myself. The new crazy onslaught of shadow it is definitely something that we're very interested in, both from the VCR point of view, but also from the Vercel deployment point of view.

Speaker A: How do you think about the fact that the models are so fully capable right now and where does Vercel sit in as a product as these models get better and better?

Speaker B: Yeah, so we're a multi product company, but so our core business for folks who don't know is you throw shit over the fence and we're going to run it for you. And the beauty of that is that there's no more being made right now. Sorry for cursing on tv, but here we go. So like we love it if you use cloud code, if you use cursor, at some point you're going to have to have a place to run it and we can connect Vercel to your git repository and every time you push we'll make you a new version of it. And have it in production. So that's how I see my primary position in that marketplace. And then with V0, for example, obviously you're also playing on the creation side. But I certainly spend most of my time actually on the question of how do I run these apps that are made in a different way in production in a way that's appropriate for this new type of application.

Speaker A: So on a scale of 1 to 10, as the CTO of a tech company in 2026, how worried are you that the stuff that you've built will become obsolete as these models get better and better?

Speaker B: There's things I'm worried about, but I don't really see. Again, since we're in the business of operating software, I don't see a major disruption there. You could argue the agents are also great at writing Terraform, they can just make the infrastructure for you, but that's actually not how that works. The whole idea of any form of professional DevOps is that I have somewhat an idea of what's going on. You can't really vibe code that there always has to be some form of platformization of how I run something. And so essentially we were playing in that space relatively successfully. So from that point of view, I'm actually not particularly worried.

Speaker A: Well, do you think that the lack of observability is a temporary limitation or a fundamental limitation of long running agents?

Speaker B: I think there's a whole revolution ahead of us of agents participating further in the DevOps space and running your application. We started calling it kind of self driving infrastructure where you have essentially something that in an agentic way understands what's going on. But I think that's still kind of like relies on the fact that there is a certain uniformity how things are running versus essentially, you know, letting the AI kind of lose.

Speaker A: If you were to start the V0 team today from zero, how would you do it differently than when you started it in 23?

Speaker B: Oh, that's a really good question. I think the obviously now we like if you already know what the product market fit is, you can jump over all these steps, right. I think the big difference is that today you actually don't need a team. So if you start, why would you have a team? Teams just, I don't know, make things go slower. Right. You can always go to the step where you have enough that you can make a judgment call as to whether that's worth investing with a single person. And maybe there's two of them, maybe there's three of them. Right. But it's not like seven people, right. And so I would tell someone, go figure out what the product is and give me a demo. And if the demo is good, I'm going to give you a team.

Speaker A: So you just have one dude, like just go and do it after you found the market fit.

Speaker B: Or Gallup.

Speaker A: Oh yeah, yeah, yeah. That's amazing. So how are you thinking about your dev teams within Vercel then? Are they much more about like one person and one sphere of ownership and then they go and they do everything? Or are you trying to retrofit maybe like a broader team dynamic?

Speaker B: I definitely don't think we have in a major way kind of changed how we operate. I think it was always a good idea to have individuals to be able to ship something end to end and to not always rely on the entire operation to make progress. And I think that's kind of like many of these things continue to be true, but they're probably more painful now if you, if you increase the speed of the inner loop through genetic coding, but your outer loop of shipping is different. I'll give you one example which I think is relevant for this group. So I was at Google before, for 12 years I led part of the search team. You can imagine essentially being very slow operation. My job was essentially to approve things via email and GitHub meetings. And so I didn't want to build the organization. And so the pattern that we use at Vercel is what in the nerdiest way you could describe as optimistic locking. So there are no approvals. Anyone can ship anything, but they have to tell the organization that they're going to do it and the organization can veto things. And so veto actually kind of sounds scary and weird, but it's really empowering. Right? Like if the legal team says, most of the time they say, yeah, I mean this sounds totally reasonable, I don't need to spend any time on it. Right. But sometimes they'll say, oh my God, you cannot ship this to children in North Korea. Sometimes that's true. Right, but, but if you have a process where legal has to say yes, then you have to wait for them and there's round trips and they're on vacation and whatever. It's complicated. If you say, well, no, you can always say, this is not ok, but you have to now be an active actor in this process. It's both empowering for those teams, but also puts the responsibility.

Speaker A: That's interesting. You guys are ultimately, you're an Infrastructure and DevOps Corporation. How do you see the role of reliability and maybe moving A little bit slowly and more conservatively for the uptime of all of your customers versus wanting to move really quickly in this age of agentic coding.

Speaker B: Yeah, I think it's just not right to take those things to be in opposition. But essentially everything we do in our business, and we have this kind of maybe great situation where it's our business to make our customers move fast and we're taking advantage of it ourselves, we're our own first customers. And so everything I do is I think about how can I make my teams move faster, how can I improve the speed of the inner loops, how can I improve the speed of the outer loops of shipping? That doesn't mean I never make a trade off. For example, we only ship our serving systems once a day, whereas we ship our control plane every time someone pushes to main. And so I would have loved to be able to make a new feature in our consumer serving stack on like 15 times a day. Yes. But we did make some trade offs there. So obviously you make some decisions, but it doesn't mean that you cannot move fast and not break things. It's possible.

Speaker A: Well, I did a deep dive into Cloudflare's outage and it turns out that it was some bad data in the control plane twice. And the whole reason they had the dynamic configuration was that they could move really quickly and make changes and customers would see that reflected really quickly. But this globalized key value pair store that is storing these configurations has the potential to take out 20% of the Internet. Do you make a distinction between the Ops and the DevOps side of the corporation and then the software and the engineering and the architecture, are those the same thing?

Speaker B: No. It's a really good question. I think the also from my experience at Google, every time Google goes down, which at a large scale only happens maybe like every five years, every single time, it's one of two things. It's either like a bad config change or it is like something around Secrets Management Service that everything depends on, or it's a config change of the Secrets Management Service, which I think was the last one. So it's always going to be that. And so I mean, in Vercel's architecture, so we operate 20 regions, 20 core regions and so on the serving stack again, we have deliberately decided that we want these regions to be autonomous and we don't have a mechanism to change them all at once. And so we thankfully cannot do these type of conflict changes that break everything at once. And so we change them in waves. And so yeah, there's another kind of deliberate decision to say, okay, we, if I want to move fast, I want to be able to get relatively quick feedback on my config changes, let's say feature flags, for example. But for that to get that feedback, you actually don't have to ship it globally. It's completely sufficient to say, maybe I only put it into one region and see what happens. Then obviously when that region goes down, we can easily just take it out of rotation. And so I think every time you do something, it's obviously important to think about what's the risk profile and can I find a way to make essentially progress at the same velocity, but without taking down the risk of, for example, global outages of 20% of the Internet.

Speaker A: Are you one of these shops that give unlimited tokens to your devs?

Speaker B: Absolutely. We even put it into our job description. I could not see why I could possibly not want that. I mean, obviously people, you know, there's always going to be people who abuse some policy. But like, I mean, how much could it be?

Speaker A: I don't know. You're the cto, don't you know about your cost?

Speaker B: No. We had a. Guys like, basically people like, mostly it's bucks, right. Like I had an engineer who was 10x. The second one was like, first of all, I was impressed. Second of all, we found out that his custom made coding harness was changing the prefix of the prompt so that it wasn't hitting the caching system, which is drastically more expensive. So obviously it happens. But yeah, I think it's worth having an eye on it. But otherwise I think the more the merrier.

Speaker A: Are your code changes going into production? CEOs code changes, are they going into production?

Speaker B: So my CEO, I think still deploys an app to Vercel every day, but he very, very rarely does production code changes. And I do, I mean regular, but you know, maybe two, three times a week, nothing super big. I think it's a really common profile people in the room will have where you say I do things, but not the ones that actually are important.

Speaker A: What do you think the future of your tech organization is going to look like as these agentic tools just get more and more powerful?

Speaker B: I think what's already happening is that there is like the job looks much more like management than looks like IC work. And that I think that's kind of changed a lot about the profile of the people. And so what we see is that the most senior ICs are in a way the one that benefit the most because they're they kind of already did a very similar job, but now they have just more minions. And then the other folks who benefit the most are the most junior engineers because they. For the same reason that they're better at making TikTok videos than me. Right. They're growing up with that stuff. I think what's most interesting is kind of what happens in the middle, like where people have to still learn to be in this, in a way, in your role and also have a kind of a need to be just willing to engage with the technology.

Speaker A: So are you one of these companies that's really leaning heavily into interns and junior engineers?

Speaker B: Yeah, I mean, year over year. Very, very impressed with the internship program. And yeah, we also have a pretty decent cohort of junior engineers.

Speaker A: So in terms of your organization, what does it look like in a year? What does it look like in two years?

Speaker B: Obviously, I don't know. My CEO and I have this quasi joke that we. So right now we're like 750 people and we just quote unquote, decided that we cap employee count at 1024.

Speaker A: Nice round number,

Speaker B: which is notably below our hiring plans for this year. So we'll see what actually happens. I could see that there's essentially this, as I'm told, right where you know, you grow the company, but it's some like the growth kind of even let's. Hopefully our growth on the revenue side kind of keeps growing up, but that maybe you actually don't need more people. Is it realistic? So we see agentic transformation. Obviously on the coding side, everyone here does. We automated, for example, our sales leads qualification with an agent that we also open sourced. We automated 87% by now of our support intake. And that, you know, that's actually a good example for. Did we let support agent go? No. I mean, first of all, the company's growing fast enough that, you know, that would never be necessary. And B, they have a much better job now because they only have to solve the hard problems, which I think as a support agent you enjoy more. So I think there are some factors that kind of are at least reducing growth. And so I do think there is going to be a transformation on that line. Otherwise, the real journey that I think we're all on is that we are figuring out how elastic the software market is because we are making it cheaper to make software. And what's definitely true, and I think we're very much seeing this also in our own numbers, is that leads to more software. And so the question is, obviously that's going to balance out at some point on some equilibrium. And it's completely unclear to me right now if that equilibrium has more software engineers than today or fewer. But it's very possible that it's more. And it's also very possible it's fewer. But I couldn't tell how because the maintenance burden that is associated with all that free software is very substantial. But also, as long as it's gains, the productivity wins, it's now worth investing in having people who manage it and so forth.

Speaker A: Yeah, I can see how you're positioned to support all of this new software that's going to be created. So I'm pretty bullish on that. But yeah, I just think that the big question about whether we're software light or software heavy in this new world where software is free, I think of it sometimes like, like YouTube. So before it was very, very expensive to create a TV show or a movie. Now anybody can do it. And so turns out we were actually video light in 2005 or 2006 when YouTube came out. And I do think nobody knows the future, but if we are software lite, it's going to be pretty great.

Speaker B: That's like, I love that analogy. Right? That's exactly my point. Right. There's going to be more software and you know, there's now like the number of people that are engaged in professional video production is drastically higher than it was 20 years ago. And again, like, that doesn't mean there's no changes. Right. I think when like to quote kind of Ben Thompson, who I think is making this really good analogy, that the transformation is probably closer to the one in the 60s when the mainframes were introduced and the buildings full of quote unquote computers no longer had a job then. It is like the transformation of the introduction of the Internet. And so that if you look back at the transformation in the 60s, that was very painful for some folks. But overall from a, from a societal perspective, obviously everyone's drastically richer now. So in the long term it goes well. And so I think there's a possibility of a transformation that's very similar.

Speaker A: Last question for me, what's your prediction about the future? That's a hot take that you haven't shared with anybody else yet.

Speaker B: It's difficult because I tweet everything out immediately.

Speaker A: Okay. Oh, your spiciest tweet then.

Speaker B: I think what I like in recent world, I think definitely. Certainly the point is that software, you already mentioned it's free. I mentioned it's free. I think we all realize it's free, like getting a free puppy. It has to be maintained. And so I think, I mean, I'm very excited about the role of agents in software maintenance and exploring that. And we touched on, like, how does stuff happen in production? Can I automatically maintain something? And. Yeah, and I want to. I mean, but otherwise, I'm just in the same position as everyone. Like, I, I now get an engineer saying I build a new file system. And they wouldn't have done that before. Right. It would have been just too much work, or it would have been on the plan that took them six months. But I know it's not a spicy take, but I really want to figure out, what do I do with that file system? Can I ship it now? Or am I now in a. Or is it actually not real?
