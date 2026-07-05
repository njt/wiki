---
url: https://gist.github.com/db554463163daf8bfbb4cfafac89e1c7
date_fetched: 2026-07-05
backfilled: true
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Here is a structured summary of Chip Huyen's talk:

Key points

The Ghibli moment of software has arrived. Just as AI can now generate any describable visual style, it can replicate any describable software. The more you build and put out into the world, the more you remove the need for imagination—someone can just say "analyze that website, do that for me" and recreate it.

Existing moats are evaporating. Data isn't a moat, it's just expensive. If someone can throw money at acquiring data and replicating models like DeepSeek or GPT, then the traditional defenses for software businesses collapse.

The incentive structure for building is broken. When anyone can replicate what you build in a very short amount of time, the question becomes: why build at all? The speaker received an email within a day of launching a side project from someone who had already used Claude Code to recreate it exactly.

Problems follow a long-tail distribution, and that's where builders should operate. AI is trained on common patterns and will get very good at the "top of the long tail"—frequent, common problems. But edge cases never go away. The sweet spot is problems big enough to be worth solving but too small or specific for large companies or frontier models to target.

Human preference is deeply local and not reducible to an equation. Cultural, geographical, and age-dependent nuances create problems that require deep, specific understanding. Example: in Vietnam, companies deploy voice bots before text chatbots because people are on motorbikes constantly and dislike typing. Conversational latency expectations differ radically across cultures (80ms response gaps in the US vs. 200-300ms in some Asian cultures).

Human-to-human collaboration workflows are outdated for the AI era. A senior engineer still reviews teammates' AI-generated code line-by-line for mentoring, but the teammates don't read the feedback because they didn't write the code. The feedback needs to shift from the code itself to how to instruct the AI to produce better code.

The world is not agent-ready. AI is getting better at interacting with the world, but the world isn't being redesigned for AI. Web search is done in a human-centric way (visiting pages, extracting snippets, re-querying) that is wildly inefficient for AI. Rate limits designed for human speed become bottlenecks. Websites and digital infrastructure need to be rebuilt for agentic interaction.

Reversibility of actions is the critical guardrail. Current environments (code, databases) are reversible via commits and backups. But as agents move into submitting forms on external sites or into the physical world (cars), actions become irreversible. This is where things get "really, really scary."

Building for joy is a valid and necessary response. The speaker is trying to normalize building things for fun—creating apps as birthday gifts for friends, like a tea-tracking app. The question is shifting from "how to build" to "what to build" and "who imagines what doesn't exist yet."

Pithy and provocative quotes

"I just fluctuate between excitement and despair. Because on one hand, I feel like now I can build anything I want, but at the same time, anyone can build anything I want. So what is the incentive structure for me to do anything?"

"I call it the Ghibli moment of software. If you can describe a style, AI can generate it. Same thing with software. If you can describe a software, then AI can build it for you. The more you build, the more you put things out there, you remove the need for imagination."

"Data is not a moat, it's just expensive. If somebody can just throw money at it and acquire data and build it, then what exactly is the moat here?"

"Just two days ago Claude Code wiped out my Postgres locally. It was trying to create a new app and I already have another Postgres running locally and it was like, wait a second, this port is taken, let me just remove it. I was like, dude."

"The senior person reviews his team members' AI-generated code line by line. He said, oh, it's not just for code quality control but also for education. And then I asked his team members, do you read his feedback? And they were like, no. Because it's not actionable—they are not the people who write the code."

"If you can describe the problem and the solution you want, usually AI can do it. Maybe not today, but maybe like two or three years from now. The question is what you build. Who's going to build the things that don't exist yet? Who should be imagining it? Do we want AI to be able to just create this solution or imagine a future of how humans should live?"

"I spent a lot of energy in doing things just to get to the part of building. But now it's just so much more fun, it's so much easier. I do think that we can normalize building things for fun."

Tools, practices, and methodologies

Long-tail problem targeting: Identify problems that are big enough to generate value but too niche or culturally specific for large AI labs or big companies to prioritize. The speaker analogizes this to finding a trading market that is "big enough to make some profit, but not too big that the sharks get in."

Cultural latency calibration for voice bots: When building voice agents for different markets, adjust response timing to match cultural norms. US users expect ~80ms response gaps; some Asian cultures expect 200-300ms. Getting this wrong makes conversations awkward or unusable.

Shift code review from code to prompt review: Instead of senior engineers giving line-by-line feedback on AI-generated code (which the human didn't write and can't act on), feedback should target how the developer instructs the AI. This makes the feedback actionable and educational.

Reversibility as a design constraint: When building agentic systems, categorize actions by whether they are reversible (code commits, database snapshots) or irreversible (submitting external forms, physical-world actions). Build guardrails accordingly, with irreversible actions requiring far stricter controls.

Agent-centric web infrastructure: Current web search for AI is inefficient because it mimics human behavior (visit page, extract snippet, re-query). A more efficient approach would pull entire pages at once rather than repeatedly visiting the same URLs. The implication: build APIs and web structures designed for agent consumption, not human browsing patterns.

Building as gift-giving / joy practice: Use AI-assisted development to build small, personal applications as gifts for friends (e.g., a tea-tracking app). This reframes building from economic necessity to personal expression and relationship maintenance.

Unanswered questions and omissions

What is the actual incentive structure? The talk raises the core question—"why should I continue building if whatever we build can be copied in a very, very short amount of time?"—and explores it from multiple angles, but never lands on a satisfying economic or structural answer. The closing is essentially "I enjoy it and we'll figure it out."

How do you monetize long-tail problems? The speaker identifies long-tail, culturally-specific problems as a sweet spot, but doesn't address whether these are economically viable at scale or whether they suffer from the same replicability problem once identified.

Is "building for fun" a privilege of those with economic security? The normalization of building for joy is presented as a resolution, but the talk doesn't grapple with whether this is accessible to people who depend on building for income.

What happens to the people whose jobs are automated? The talk opens with an audience poll about job automation and notes that many people think their jobs will be automated, but never returns to the implications for those people. The focus shifts entirely to the builder's existential crisis, not the broader labor market disruption.

The "artisanal software" idea is raised and immediately dismissed. The speaker wonders if there's a future where people value hand-built software the way they value custom-made clothing, then says "I'm not sure if I would actually be interested" in that distinction. This potential counterargument to the despair is left unexplored.

Who is responsible for making the world agent-ready? The talk identifies that websites, rate limits, and search infrastructure are designed for humans and need to be rebuilt for agents, but doesn't address who should do this work, what the incentives are, or what standards might emerge.

The Postgres wipeout story is treated as a funny anecdote, but the implications are glossed over. An AI agent deleted a local database to free up a port. The speaker had a backup, but the systemic question—how do we prevent AI agents from taking destructive actions based on incomplete understanding—is raised only in passing before pivoting to the broader reversibility point.

Okay, see, it's very uplifting. Who here thinks that the job won't be automated by AI in the next five years? Wow. What do you do? How do I get a job?

So who here thinks that a job can be automated in the next five years? Okay, so how about the rest of you? Like, you don't have a job or something? Yeah. So I did this question yesterday.

It was just curious, like, what people online would think, and it seems like a lot of people think that their jobs are going to be automated. So don't despair. I think I was trying to end the talk on a very uplifting note, but I think recently I launched something as a side project. It's small, I'm happy about it. It got some eyes on it.

It's like a week. It's like 300,000 views. And within a day I got an email from someone saying, hey, I love what you built. So I use cloud code to recreate exactly that. And here's a link.

And I'm just like, I'm flattered, but I'm so, like, what the. Like, so, so and so, like, made me realize that whatever exists can be replicated. So I think, like, it's. It makes me feel weird because at the same time, like, I just fluctuate between excitement and despair. Because on, on.

On one hand, right, I feel like now I can build anything I want, but at the same time, anyone can build anything I want. So what is the incentive structure for me to do anything? Right? Like, I think I have stopped using a lot of what I call SaaS, light, like some, some products. I feel like, sold a very small problem and charged me like a lot of money, like, per seat.

First of all, like, I know $50 per seat per month for a single one. Like, and I feel like, why should I keep paying for you? Why don't you just recreate exactly that, like with, with AI? And of course people can tell me that, like, okay, it's not quite the same, but there are things that's like, harder to build, right? You can't just get AI to do like a Google, like in a day.

Of course it takes longer, but I also notice that like, AI just get better and better over time. Maybe it cannot recreate Google in a day, but maybe like over time it can. And I think it's about research by people showing that AI can accomplish tasks like, exponentially more complex. So what AI can do today is already way, way more powerful than what I imagined it could do, like just a year ago. Or like way, way More than three years ago.

So maybe it's just a matter of time. And people used to tell me that, okay, there are different modes, like data is a mode, but it turns out that data is not a mode, it's just like expensive. You have seen how easy it is for people to replicate deep seq or like GPT5. It's not really a moat. If somebody can just throw money at it and acquire data and build it, then what exactly emote here.

Why should I continue building? So sometimes I call it like the Ghibli moment of software. So the idea is that like a few years ago everyone was excited about, hey, you can use AI to generate pictures in any style. And somehow the Ghibli studio style became the style that people really like. And I think by that point I realized that if you can describe a style, AI can generate it just sending the software.

If you can describe a software, then AI can build it for you. The more you build, the more I put things out there, you remove the need for imagination. You can say, okay, now analyze that website, do that for me. This is quite weird. The question I want you to understand is why should I continue building?

So I'm curious here, like why, why do you think that we should continue building if whatever we build can be copied in like very, very short amount of time? Learning process, that's great. And then what? So does this make you feel like, yes, purpose, but if I don't do it, somebody else will do it, you know, like it's somehow not needed anymore. It feels great.

Okay, you stole my punchline. It was supposed to be like the end of the talk. But I do think that's one thing that makes me want to build, is that I build because I want a sole problem, right? Build. You create a product and it doesn't exist in a vacuum.

The product you build is your solar problem. So that like is the more you do that, the better you become a problem solving. And one thing I do believe that it will never change, that there will always be problems to solve. I don't think AI will just instantly makes me a happy person. I don't think AI would magically make me stop being annoyed at customer support agents.

I don't think it's going to go away ever. So I think there's a lot of problems you saw. And when I look at the world of problem, I do believe that problems follow the long tail distributions. And just by how AI is trained, it will be able to do a lot of things that it see a lot So I think of them as the top of the long tail problem. Something very common issues that a lot of people experience.

AI wouldn't get really good at it. And over time, AI will cover more and more edge cases, but the edge cases will never go away. So there are a lot of things I consider a long tail problem. By the way, anyone here in the precision market, prediction market, like anyone into betting, gambling, it will never go away, huh? Yeah.

So. So I'm thinking. So it was so it was thinking about. So. So I did build a bot to do trading.

Everyone has. I told my friend was like, I'm shocked as our trading phase comes so late in life because I feel like if you're into engineering and math at some point in your 20s, you just have to get into trading. So, so I got into trading and I realized it's like the bigger the market, like if some higher trading volume, the more efficient it is. Now if you get into spot prediction, it's almost like impossible to compete with a sport trading firm. Or if you get into like, I don't know, like you cannot compete with like hedge funds because anything when it becomes big enough, so people with a lot of money and amazing infrastructure and get in.

So I found out like the sweet spot, it was something that's like that is a market that is big enough to make some profit, but not too big that like on the sharks, you know, ready? So, so I think of it as the same thing as the problems that I want to solve, right? Like if it's a big problem, like everyone can see, then all these big companies when get into it. But whereas it's like there are a lot of problems, it's like smaller. Then maybe OpenAI won't be motivated to solve it, but maybe I can.

Like a lot of people can. And I think what do these problems look like? And I think it's like human preference is one thing. I don't think that human preference is just an equation that people can just package nicely and like, hey, ask people, hey, which of these two answers people would prefer? It's very, very personal, very culturally dependent, geographically dependent, age dependent.

So I'm from Vietnam and recently I went back to Vietnam and talked with a bunch of people doing AIs there and I noticed something very interesting. So here, when a lot of companies deploy customer support, chatbot agent stuff, right? They usually go the text route first. Like you do text and text is easier. And then they do voice bots because like voice is like so much more complicated.

Like instead of having like Text in and then text out. You first have like transcribe from a speech to voice and then, oh, sorry, voice. You speak, oh, sorry, speech to text. And then you input the text, the question from users into the LLM, get back the answers and then synthesize into the voice and then get back to the people. So it's a lot more complicated.

So it's natural that people here do text first. But in Vietnam and also in a lot of other Asian countries, people are on the move all the time. Like people are on the motorbike all the time. So actually really don't like typing. So the voice.

Like a lot of the companies in Vietnam actually deploy voice bots before they do chatbot. And I talk to them and there are a lot of like cultural nuances. So for example, like the response time. So I was like just showing. So I have a niece and nephew who's like very young.

And then I have like my godparents here who are like a lot older and then I put them on the call. I feel like it's a disaster when you get like a 60 years old American grandparents who like 10 years old Vietnamese boys and girls. And my godmother, right, because she wants to avoid awkwardness, she just kept on talking and when she asked her questions, the kid didn't answer. She just kept on asking a lot of questions. I get conversion going and then after that I told her just like, do you know that there's a research that show that in the US people expect you to respond instantly.

So like when you finish a sentence, people only give like 80 seconds, 80 milliseconds for the other person to respond. Otherwise you need to continue. Whereas in Asian culture, like our respect, like you actually wait a lot longer, more like 200 millisecond or like 300 millisecond to make sure the person finish. So if you just keep on throwing to erase awkwardness, it will become awkward because the episode was like, wait, I can never get my voice in. So the same thing with building chatbot voice bot, right?

Because voicebot, you want to balance our latency and also like humanness of it, right? You need to wait for the human to rest to finish. But then if you wait too long, it wouldn't be too slow because now we had to generate on this process of parsing and then generating. So like all of that is very hard to solve. You have to understand all these nuances.

And all the examples that I show are just like, they are like very obvious like cultural differences, but they are a lot More things that only when we go into specific use cases and specific demographics that we're targeting that we can understand. Another thing that I think is very important is the way humans interact with AI. So I think there's a lot of things we still trying to imagine what would be an AI driven world look like. I think people are trying to retrofit whatever exists to fit what they think is a new workflow. For example, the IDE and the terminal.

So who here is using a lot of coding in the terminal? So who here only started using the terminal because of coding? Nobody. I guess you have more engineering, but I think saw someone here. Thank you.

Appreciate. I have friends who a lot of them are like PMs or doing more of like a product. They never used terminal before, but now because of AI they actually like became like, wow, what is this? Like I have to do this now. And it's terrible because you cannot copy and paste, you cannot like upload a file into it.

It's just like very annoying to use. But people use it, right? Because that is what what we have. And people was like, okay, can we have a different terminal? Why is there separation between a terminal and a VS code, for example?

Why is there a debate? What they do is they take an instruction and they produce code or product. Why should it be different? What's the fundamentally difference between a terminal and an ide? So I think a lot of that is still ongoing questions that we need to figure out.

Another thing that I think is very important is the human to human collaborations. Because I do think that AI is getting really good at solving problems for each person. But I do things that you build things that are meaningful. We need to work together now to human collaboration with AI and makes actually very complicated. Here's an example.

Who uses GitHub a lot, right? Who here collaborate with your team on GitHub via PRS? So who here still review every single PR lie by line? So PR the way it's like it should guardrail the human to human collaboration. So the idea is that somebody could review your coworker work to make sure that it meets the standard before merging.

So I talked to a team recently and one of the most senior people person on the team told me that he still does that lie by lie. But he does not review his AIJ record lie by lie but he reviews his team members AIJ record lie by lie. And the reason he said, oh, it's not just for code quality control but also for education. Like he's mentoring his team. So he wants to give feedback so your team can get better.

And then I asked his team member like okay, do you read his feedback? And they were like no. Because the reason is that it's not actionable because those union members are not the people who write the code. If you say okay, instead of writing code like this, write like this they were like yeah, but I'm not writing the code. How do I give that feedback to my AI so you can write code like that?

So I think there's a whole workflow of reviewing code is very outdated. I think as a senior member instead of giving feedback on the code they should be giving feedback on how you give instruction to AI to produce better. So that's like another example of like how the human to human collaboration is going to be very different. And another thing that is, I think it's very exciting is to AI interaction with the environment. So right now we interact with AI mostly on the computer so I think we're just giving AI more and more access to different things.

Right. First is the leetcode on the IDE and then terminal which terminal is already getting a bit more dangerous because recently for example like just two days ago Clark code has wiped out my postgres so locally and the reason is that it was trying to create a new app and I already have another postgrad running locally and was like wait a second, this port is taken, let me just remove it. So to create this new I was like dude, like yeah, so. So. So it's very scary but luckily I have a local backup because I'm not stupid but.

But yeah, so it was fine But I think like that made me also think about like a lot of the environments that I currently operate in are reversible they can be snapshot right? Like code if AI mess up you can revert back to the last commit even if you guide my database okay, can just go back to the previous backup but there are a lot of environments where the action cannot be reversible by us. So let's say that we have an agent that we use to fill out form for us maybe go to a website, enter a form and click submit. Now as a user we cannot reverse that action because now it belongs to somebody else computer and we cannot do that or if we give AI more access outside the digital wall like in the real world let's take an example of a car, it runs over a pedestrian. You cannot reverse that, it's just not working.

So I do things that we need to build out the whole guardrails for the reversibility of actions because that actually where things get really, really scary. And another thing that could be very interesting is that I do things like AI to environment interaction two way street. On the one hand we have on the foundation model frontier labs that are making models better at interacting with the world. But who is making the world more agent ready. For example, a lot of the website I do things for the book, I write books.

And as much I wish that books as a format will survive, I do things that people don't read books the same way anymore, right? Like, I mean I don't think people like read books ancient. I wish they did to my book. If someone told me that and was like you're lying, you probably jump around a little. So.

So I do think that like the world is changing and we need to come up with format like the website is easier for agents or like I was reading something else like so a lot of the world nowadays is like digital world is controlled by rate limit, right? And a lot of time it makes sense because as humans we don't do things that fast, right? But with AI now AI can just interact with like AI to AI can be really really fast. So on the whole concept of rate limit is irrelevant to AI. Like it's going to become bottleneck or the whole concept of search.

So recently I spent a lot of time looking into how AI do web search and it bothered me. So, so when I sent a search as like so I did a bunch of benchmark between Grok and Gemini and cloud and open and model to do web search and I asked it's a query. And I saw that full query, some of them do like 900,000 URLs visit and like that is so much use of burning my credits like crazy. And then I look at like how many of these URLs are unique and oh shoot, yes, I think that's fine how some are done. And it was like how many of these 1000 URLs are unique, right?

And it turned out it's like only 20 of them. So the AI kept visiting all of these LLs again and again. And to me it's just stupid. And then I realized what happened because it first visited ll, it took the citations, it took the part that relevant and then it do another query. It files the parts that are relevant.

And I feel like it's a very human way of doing web search, right? Because we enter things in the Google and we see on the citations quotations part. But why would we limit AI to that? If I only Visit a page. Why don't just pull the entire page out.

Why do you have to keep on doing that again and again? It's just stupid to me. So I feel like, or maybe not stupid. I'm sure the people who build that are smarter than me. I'm just saying that like my mental model is just seem off and I feel like I think there must be a more efficient way of doing things that are less human centric and more AI centric.

And I think I hope that somebody will do that. So I do things that. There are a lot of things you build and the question nowadays is less about how to build because if you can describe the problem and the solution you want, usually AI can do it. Maybe not today, but maybe like two or three years from now on. They can do a lot of those.

The question is what you build? Because we talk about yes, if something exists, AI can replicate it, but who's going to build the things that don't exist yet? Who should be imagining it? Do we want AI to be able to just create this solution or imagine a future of how humans should live? Or, or like we can also propose this idea like think about like we want to build and going.

Actually what you said about previously, like why should we, why should I continue building is. I do think this is like, because fundamentally I enjoy building. Like it just bring me joy and I do things that, I do things that I hope that we can normalize like building things for fun. Because before I found out that I spent a lot of energy in doing things just like just to get to the part of building. But now it's just so much more fun, it's so much easier.

I can do a lot more things and I think it's like if you look at our, a lot of economic or like industrial progress. So in early day, for example, right? Clothes, like we did everything by hand and then we have like all the mass manufacturer clothes which is great because people can access to clothes cheaply. Like anyone can have like fast fashion. But then when people have like higher like disposable incomes, they start looking, oh, actually I don't want mass produced stuff.

I want something just like custom made for me. I don't think we would have a future when like artisanal software. I don't think, I'm not sure if I would actually be interested like oh, analyze this app more because it's built by hand versus like this app, right? So I'm not sure it will get there. But for now I do actually enjoy building apps.

As a gift for my friend. So for their birthday, instead of, like, buying them something, I'm going to spend, like a weekend, build them, like an app. Because for they like tea, I'm going to build them a tea tracking app. You know, like, it's just, like, very, very simple and it's a lot of fun. So, yeah.

So I do enjoy building, and AI does make my life, for now, happier, except when I'm feeling very, very depressed, because I don't know why should I continue building. But I think we'll figure it out. But thank you so much, everyone. That is my.
