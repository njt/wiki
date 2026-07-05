---
url: https://www.oreilly.com/radar/what-you-bring-to-ai-determines-the-result/
date_fetched: 2026-07-05
backfilled: true
---

Harper Carroll came to AI education through a CS background at Stanford, machine learning engineering at Meta, and a brief stint at a small GPU compute startup in late 2023, where she noticed that almost no one understood how to fine-tune open source models. She started writing and teaching to help drive signups for the startup’s platform. Her first guide, posted right after Mistral 7B was released, when she had about 50 followers, got 50,000 views. In March 2024, a video explaining the difference between AI and machine learning got 5 million views, with 1 in 20 viewers following her afterward. She now has more than 500,000 followers across multiple platforms and is a full-time AI educator.

We covered fine-tuning versus prompting, what it actually means to learn to code in 2025, and what the AI field gets wrong when it talks to the public.

## Understanding the world with math

We started with Harper’s own AI learning journey, and it contained a wonderful insight. She grew up loving math and came to computer science at Stanford because algorithms seemed like wonderful math puzzles. Eventually she realized that AI is “understand[ing] the world around us with math.” Text-based LLMs are only one branch. The field as a whole is “the math of the world.” That seems like a deep intuition that all of us need to internalize.

## AI as a medium

A study that circulated last year found that people who used AI to write essays showed reduced brain activity compared to people who write unaided. The reaction in many quarters was alarm. People said, “We’re outsourcing cognition and our brains will atrophy.” Harper’s smart response was that those users must have given the AI a one-sentence prompt and accepted whatever came back.

As she put it, that’s the equivalent of just telling Alexa to order you the most popular book this week. Of course less brain activity is being measured! Contrast that with the difference between shopping for a book by browsing and searching at Amazon versus driving to a physical bookstore. There’s certainly a difference, but it isn’t outsourcing cognition. It’s saving time, and that time might well be spent on other demanding cognitive tasks.

My framing is that AI is a medium, the way language is a medium, or photography. Anyone can take a photograph or write a book. The words available to every writer are the same; what differs is what they do with them, just as some photographers do something with it that others can’t. The same is true of software. There’s a line in Aaron Sorkin’s movie *The Social Network* where the Zuckerberg character says about the Winklevosses, “If you guys were the inventors of Facebook, you’d have invented Facebook.” An idea and its execution aren’t the same thing. One person gives AI a prompt and the output is bad. Another builds a process around AI and the output is great. What you bring to the medium is what determines the result. Harper agreed.

## Fine-tuning is like psychedelics for AI

I’ve been trying to figure out how we can use AI for writing and editing at O’Reilly. We want skills and workflows that accelerate our productivity but don’t produce copy that reads as whatever the base model sounds like when nobody’s putting in any effort.

Takeaway posts like this one are a great use case for AI-assisted writing. As source material we have a transcript, with the actual conversation between the participants (or in the case of one of our online conferences, their presentations). We want a structured summary that captures the high points and suggests possible clips for social media. I (or whomever is using this AI-assisted workflow) can then rewrite, rearrange, elaborate, or delete from that first draft. It might not be as good as a draft written from scratch, but quite frankly, it’s far better than the alternative, which is no summary at all. I just don’t have time to write them all unaided.

When I’m writing an article, I generate a similar “transcript” by recording myself talking about the ideas I’m wrestling with and trying to put into the world. Then I ask Claude to put it together into something a bit more structured.

I’ve been improving Claude’s ability to produce prose that we can use by rewriting its output, showing it the differences, and then asking it to construct a skill that captures what it’s learned. Over time, it’s gotten closer and closer to something that I’m comfortable with, and I’m now generalizing that into a system that learns any author’s voice, respects the various conventions of the target content type (which can be very different across books, articles and blog posts, social media, and marketing materials like back cover copy and course descriptions), and applies editing suggestions from my favorite books on good writing, including Strunk and White and *On Writing Well* by William Zinsser.

Harper attacked the same problem from a different angle. She built a dataset of roughly 1,000 of her Instagram captions, video transcripts, and X posts, then fed them to Claude as context and asked it to write in her style. Unfortunately, the output tested 100% AI by a detection tool, even with 1,000 examples of her real voice in the prompt. She then fine-tuned an open source Llama model on the same data. The fine-tuned output tested 100% human. She gave a compelling demo at South by Southwest showing how easy this is to do. It took her about 20 minutes.

After Harper said that prompting doesn’t shift the output distribution the way fine-tuning does, I told her the story about the French writer Marcel Proust that I first used in my conversation with Steve Wilson, which I picked up from Alain de Botton’s *How Proust Can Change Your Life*. A friend comes to visit the bedridden Proust, and making polite conversation begins to tell him about the train trip to Paris. “More slowly,” Proust replies. This cycle repeats several times until the friend is telling him small details like the old man feeding pigeons on the steps of the station.

Harper got it, and broke it down more slowly in her inimitable way. Here’s why in-context prompting fails where fine-tuning succeeds:

Basically AI models are these massive mathematical equations, and the parameters are variables when you’re training, and then they become constants in those equations when you’re running inference . . .So what you’re doing when you’re training the model is you’re learning how to map, by adjusting those constants when they’re variables during training,. . .input to desired output.


Once the model is deployed, the probability distribution over output tokens is fixed. You can put 1,000 examples in a prompt and ask the model to pattern-match, but you’re asking it to do that with frozen weights. The surface behavior bends a little, but the underlying distribution doesn’t shift. Fine-tuning lets you actually modify the weights and how the model *wants* to write.

Her suggested approach for building the training dataset is to take your own writing, have AI rewrite it with its characteristic tics, then train with the AI version as input and your original as the target output. You’re teaching the model to undo the tells.

## Should people still learn to code?

We also spent time on the inevitable question of whether people should still learn to code. We both agree they should, but not necessarily like they used to, by learning the detailed syntax of a programming language, then by trial and error as they painfully learn how hard it is to get the desired behavior.

Harper’s take (which I also agree with) is that vibe coding has lowered the floor. People who could never afford to hire someone to build a product can now do so themselves. But it has also raised the ceiling, because people who actually understand systems can build vastly more sophisticated things with the same tools, which takes us back to the case for AI as a medium.

Perhaps more importantly to the question of how much coding you should learn, experienced developers will also see failure modes that pure vibe coders miss. Harper gave an example that came from watching a friend using an agent tool that had, at some point, started storing its data in a Word document and using it as a makeshift database, probably because the session started with a Word doc. It was extremely slow and extremely inefficient. An engineer sees the problem immediately. A vibe coder might run that system for months before noticing something is wrong.

So yes, you should learn enough about coding to understand what’s happening. The art of teaching programming to the next generation will be developing useful projects that also highlight underlying concepts of software architecture and engineering.

## Intuition as differentiator

Silicon Valley runs heavily on logic and on the idea that good decisions come from better data, more rigorous analysis, and sharper models. In this environment, intuition can get dismissed as something “soft and fuzzy,” Harper noted. And that’s the wrong mindset for AI.

AI is getting better and better at exactly the things the logical axis does well, but intuition remains a challenge because it often contradicts what the data says. Good intuition “goes against the input,” to use Harper’s phrase. A model that’s been trained to recognize patterns in data will, almost by definition, struggle with making decisions that run counter to those patterns. Just as skills-informed judgment supercharges AI-assisted engineers, intuition could be a uniquely human skill for a long time. Elevating it as a concern might bring the industry more of an attitude of humility towards ourselves and our place in the world.

## What the field gets wrong

I closed by asking Harper what the AI field most consistently gets wrong in how it talks to the public. She said that too much of the public-facing discourse leads with fear, of job displacement, of rapidly approaching AGI, and of a rocky transition that requires a universal basic income to cushion the blow. She’s not calling those impossible futures, but she thinks they’re the wrong introduction to the technology.

A lot of companies are using AI to ask how to do the same things at lower cost. The better question is how to raise ambitions. AI doesn’t just scale individual capabilities. It scales what organizations can attempt. But for it to work out that way, *everybody* has to actually learn AI. We can’t have AI haves and have-nots. That means lower-cost models, serious open source investment, and companies that don’t just become serfs to the major platforms.

Harper has been making this point for a while, to audiences ranging from engineers to people who’ve never written a line of code. “There is not really much to fear right now,” she says. “AI is this incredible productivity tool.” The people who will struggle, in her view, are the ones who refuse to engage with it at all.

At O’Reilly, we’ve been working on a version of the same narrative at an organizational level. The fear-first narrative produces avoidance, and avoidance is the one thing that will actually leave someone behind. So we’re building a corporate AI transformation practice that starts with people’s existing jobs, and figures out how to “mix in” AI to make them more impactful. We’re learning how to teach both the humans and the agents at the same time to make them more productive together.

*On July 9, I’ll be speaking with Trail of Bits cofounder and CEO Dan Guido about the playbook his company used to go AI native, which he first outlined at this year’s [un]prompted. He’ll give a version of the same talk, then take about 40 minutes of audience questions on what worked, what didn’t, and what is still unsolved. I hope you join us to find out what’s changed since [un]prompted and where the playbook is heading next. Register here; it’s free and open to all.*
