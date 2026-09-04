---
url: https://softwaredoug.com/blog/2026/08/29/ai-team-mistakes.html
date_fetched: 2026-09-04
---

Your company might be forming an AI team. That AI team starts with excitement. Frequently, though, it collides with reality.

I work in RAG. By now, I’ve had a front-row seat to a dozen plus budding AI teams. As a consultant being brought into expensive disasters on the search side of AI, I’m consciously biased here. Like the cop that only sees the hard, rough side of life in the streets.

A surprising amount of the pain I see on AI teams looks very much like what budding search teams in the 2010s went through. Admittedly this gives me a biased perspective, but perhaps a useful one.

## Evals: We can’t assume we know what ‘good is’

Great search/AI organizations spend ~50% of their investment on understanding the problem, not solving it. The same goes for AI.

This means **evals**.

You want to find opportunities for product improvement? Don’t blindly trust your PMs opinions, eval!

You know something’s wrong, but can’t find exactly where your agent fails? You need evals.

You want to train a model on user behavior? Evals become training data.

My colleagues Hamel Hussain and Shreya Shankar teach an entire course on AI evaluation. They focus on measuring end-to-end product success, not generic metrics. Then they have a methodology for breaking that down and finding the weak points. Maybe its retrieval? Maybe it’s your guardrail? Maybe it’s something else?

I couldn’t even start in search without a basis for evaluation. I created Quepid 12 years ago, because there’s no objective right / wrong answer in conversational systems, search or AI.

I learned to actively NOT trust my instincts. I try to discover with my clients / customers what should happen. Years ago, I did a project for Advanced Auto Parts internal search. I assumed when employees searched for a product, they just wanted that product to come up. No: what they actually wanted to know was “what will I get an incentive for selling?”.

Your job isn’t just to build things, it’s to be a scientist. Evaluate, hypothesize, test, improve. Don’t lose track of that!

## Retrieval: it’s not a checkbox, it’s the whole thing

You knew I was going to say this. But how complex this gets - how diverse retrieval can be - becomes a blind spot. Teams just assume there’s one classic RAG architecture to build that applies to everyone.

We need to recognize that AI teams are search teams. One of the easiest findings out there in research: **retrieval dictates AI quality**.

In one such paper you can see how dramatic having the right context improves AI quality. They give the LLM the right context, and we see how much answer quality improves.

(Of course, this assumes you have the evals to know the right context!)

Solutions to get the right context vary dramatically.

You will spend an inordinate amount of time focused on decisions like how to chunk, what technologies should be used to retrieve those chunks, how should they be ranked, how do you give a diverse set of answers.

For example, the authors of the paper above try different retrieval approaches with varying results:

Each of these solutions are quite different directions. It’s easy to get sunk-cost into one approach in search. Yet there are cheap ways to experiment with naive solutions before jumping to high levels of complexity.

Navigating cheap/easy/dirty to discover what works before building out expensive/robust takes search experience.

## Context doesn’t mean chunks - it means metadata

In RAG, the classic chunking strategy involves breaking up text into useful passages. We assume, in search, that RAG involves a classic question-answering paradigm: embed passages, embed queries, find the similar results, inject them into agent context.

Yet I’d argue, that’s not *really* what RAG is. RAG is about presenting useful information to an agent. Useful to let the agent evaluate its trustworthiness and relevance to the prompt.

If you built RAG for my blog, which chunk is more useful to the LLM?

While I worked at Shopify, I implemented a search relevance solution


Or

```
## Title: My search relevance work at Shopify
## Popularity
Medium
## Publication Date
07-20-2020
“While I worked at Shopify, I implemented a search relevance solution...”
```
The latter helps the LLM consider how to use the information. Is it recent? Is it trustworthy? Is further investigation warranted?

RAG is about helping the implicit judge inside the LLM make better decisions. It’s not about arguing how / where to exactly split articles up into paragraphs.

When building AI, you need to support the brain inside the LLM. That means representing information with structured metadata that’s important to your domain. Something can be semantically similar to the prompt, but be a horrible response to the user.

It goes beyond that too. We think of retrieval as only lexical/keyword or embeddings. But representing the right metadata and building tools for agents to select on that metadata can matter more. I think of it even as the third, hidden pillar in retrieval: query understanding and metadata.

Don’t think about “chunks” think “how do I represent a unit of information and its provenance to an LLM.” and “how do I let LLMs navigate that information architecture.”

## The intersection of engineering and data science

Like search, AI teams thrive when they can be multidisciplinary. Can you build scalable systems, but also think in hypotheses? Can you create a retrieval system that scales, but you know targets the weak points in your evals?

I help orgs build their search teams. I have long encouraged getting away from siloed engineering + data science. You need minds that can fit both perspectives into one brain to make minute-to-minute tradeoffs when building. Not data science throwing models over the wall, wait 3 months once built out, only to realize it’s the wrong model, and you need to start over.

That’s also increasingly how I think about educating AI teams. If you want to get a batteries-included “getting started with AI” perspective, check out my course with AI expert Hugo Bowne-Anderson. We’re going to actually teach at Building with AI course that doesn’t yadda-yadda over the search part.

What else do AI teams miss? Get in touch and let me know!

### Learn to Build Agents + Retrieval from Scratch

Take a course from Doug and AI educator Hugo Bowne-Anderson as they build an agent + retrieval from scratch! Sign up!
