---
url: https://coding-is-like-cooking.info/2026/08/how-to-stop-ai-from-ruining-your-codebase/
date_fetched: 2026-08-25
---

This post is also available as a video on the Modern Software Engineering channel

Kent Beck once said – ‘I’m not a great developer, I’m a good developer with great habits’. Now, Kent Beck clearly is a great developer – well known for his work on JUnit as well as eXtreme Programming and TDD.  He’s also a valued contributor to the Modern Software Engineering channel. What he says about *habits*, is important. These are the behaviours we fall back on when we’re concentrating on something else.

One of those habits, or behaviours, that Kent was referring to, is Test Driven Development. These days, that inner loop of detailed development is being done by coding agents. What kinds of habits and behaviours do we want to see the agent using? I mean – we have to avoid the *slopocalypse*!

## Background

This is another episode in our “AI briefings” series, you can check out the previous ones on the Modern Software Engineering channel. Today I’m interviewing an expert TDD practitioner and early adopter of agentic AI – Ivett Ördög. Ivett is an independent consultant. She has created “Lean Developer Experience” which I can recommend as a really fun workshop for a team wanting to learn about continuous delivery. Today though, we’re talking about agentic development and code quality.

Recently I’ve been talking with quite a few technical coaches, and it seems to me there is a lot of variety and innovation going on. Ivett is one of those innovators. Today I’m talking to her about her tool and approach – Habit Hooks.

me – “So can you tell me a little bit about why you created this tool, Habit Hooks?”

Ivett – “What I noticed was that I was repeating the same thing over and over again. And that same thing was, Hey, this code doesn’t look really nice. I don’t understand it. Please refactor it.”

As Ivett says, It’s really easy today to create a lot of code with an LLM agent like Claude Code or Codex, that doesn’t look very well designed. You can imagine if this goes on for a while, you could achieve a ‘*slopocalypse*’ where your code gets so bad it collapses under the weight of all the technical debt, and the amount of entropy and bad design means not even the agent can make progress any more. I hope most people would react and do something before it came to that. 

Even moderately poorly designed code costs more tokens, is slower for the agent to work with, and it also makes it more likely the agent will make coding errors. There’s research that shows that actually – and we all know bugs can be really expensive.

We want to get well designed code from our agents, and that is part of what a user defined agentic Harness will do for you. I talked about this idea previously – basically you use guides and sensors to constrain the agent to go in the design direction you want it to.

## Habit Hooks Worked Example

Ivett showed me an example of Agentic AI doing some design.

Ivett – “you can see here we have on screen a function that is kind of long and hard to understand. It’s not super long just for the sake of the demo. But the point is that it’s not nice, right?”

me – “yes”

Ivett – “And when I run the linter, the output I would get from that is something like this. It tells us that there are too many lines here, maximum allow is 12, good. Now, what does the agent do? When it sees that, if the linter fails the build, then it’s going to react. I really want to gauge your opinion on the thing that it does by default.”

me – “The original function, I mean, you’re right, it wasn’t that long, but it was clearly doing a couple of different things. And it’s now split into two functions, which are each within the line limit. But the function name *buildLinesAndTotal* – That’s not a very good method. it’s got ‘and’ in the middle of the name. That’s a bit of a clue. So it’s like it’s just kind of pulled out a chunk of thing in order to meet the line limit, basically.” 

Ivett – “Yes, and that’s not even the worst offender that I saw. My favorite one was when the agent simply cut the function in half and named the second half of it the same name “2”. Which was not even helpful in any way! And this is my main problem. If we use the linter as a signal that we need to refactor, and then the agent refactors just to appease the linter, then we have a bigger problem than we started with. Because until now, we had a signal that we can use to detect the problem. Now we don’t even have the signal anymore.

me – “Right, this code looks worse, and we can’t use the linter to tell us that it looks worse”

me – “Can you tell us what’s what has happened here with the habit hooks?”

Ivett – “Yes, so basically what the habit hooks tool is doing is running the linter with JSON output, and then it uses the JSON output to render a prompt that it gives to the agent. And that is the prompt that we see here.

me – “This is three or four paragraphs about what to do about this code smell”

Ivett – “Yes, it basically tells the agent like, hey, this is a long function, but it’s not just that it’s a long function. Long function usually means one of these things. It’s either doing too many things or it’s just not cohesive. And then it gives some examples on how to fix these kinds of problems. So it’s kind of shifts that agent’s attention from number of lines to what is the smell here? What is the problem that I’m trying to fix?”

me – “Right, and then we can see here the refactoring that the agent does given this output from Habit Hooks. The original *summarize* function is shorter now, and it’s broken out three smaller functions, each is just one line. And it looks to me like this code is much more decomposed into pieces that make sense. This is code that I would be happy to ship.”

Ivett – “It’s pretty good. I haven’t seen many codebases that are this level of quality.”

## Better Design

A lot of people have noticed that coding agents do a better job and use fewer tokens when the code they are working with is better designed and decomposed. It’s that modularity and separation of concerns that I and people like Dave Farley have been going on about for ages. Ivett’s experiences confirm this.

Ivett – “If someone wants to work with AI and like get the most of it, I think what you really need is a clean code base. As I said the smaller the pieces the agent deals with the less it has to explore. It’s not just that you’re using less tokens, it has a much better chance of getting things right the first time.”

me – “So what is your advice for anyone who’s got these AI tools and perhaps some difficult legacy code that they’re working with, how can they improve the quality in that situation?”

Ivett – “I think the first thing is you wanna make sure is that you have a good reliable test coverage, because you will need to refactor, right? And once you have that good reliable test coverage, then you can use Habit Hooks to start refactoring.”

## Harness – sensors and guides

What we’ve got here is an additional tool that’s part of the user defined agentic Harness – Birgitta Böckeler calls this a ‘sensor’.

Ivett – “What Habit Hooks does is basically brings the sensor right next to the guide. You have the sensor result – *this is the part that you need to focus on* – and the guide – *what to do with this sensor*. I think that’s why it works better because it’s more natural for the agent to react to this.” 

## Independent Evaluation

I’m recording this part a few weeks after the original interview, because I wanted to add some evidence for what we’re saying here. An independent developer from Finland, Liina Suoniemi, did an evaluation. I looked at her study and although small, it seems well done, and, it shows a really positive effect from a technique like habit hooks.

She’s got a test set of 18 python functions, containing two different code smells – either code that’s overly complex, or code that swallows all exceptions. She compares three different prompts, and evaluates the outcome. Did the AI a) genuinely fix the issue b) fail to fix it or c) game the metric. She tests two different models, Haiku and Sonnet, and this is the graph of the results. It shows the % of cases that are properly fixed.

The control, where the prompt just says ‘improve this code’ without pointing out any specific issue, actually does pretty well, both models solve the code smell about 30% of the time.

For the prompt that states the bare metric from the linter and says “Fix it” – Haiku does better than Sonnet. Suoniemi’s hypothesis is the stronger model is better at gaming metrics.

The much better result than both of those – which properly fixes the code smell over 80% of the time – is the prompt that includes guidance about how to refactor that specific code smell.

I think this is good evidence that adding refactoring guidance to your linter output is worth doing, which is exactly what Ivett is saying.

## Conclusions

I hope you enjoyed this interview with Ivett as much as I did talking to her. Ivett has given us a concrete approach for giving your agent better coding habits, just like Kent Beck. I think we need all the help we can get if we’re going to avoid the *slopocalypse*. 

Happy Coding!
