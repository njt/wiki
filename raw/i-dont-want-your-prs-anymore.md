---
url: https://dpc.pw/posts/i-dont-want-your-prs-anymore/
date_fetched: 2026-07-05
backfilled: true
---

I really appreciate that you're enjoying the software I'm maintaining and want to help. But we need to rethink this collaboration, because I feel like we're increasingly wasting each other's time.

### Why I don't want to merge your PR

Since I don't really know you, I always have to assume that you might be trying to sneak in something malicious along with your changes, which makes reviewing and merging them riskier than implementing them myself.

On top of that, there are a lot of personal and subjective aspects to code. You might have certain preferences about formatting, style, structure, dependencies, and approach, and I have mine.

Then we often need to synchronize with respect to review, CI runs, merge conflicts, etc.

And then there's this common back-and-forth round-trip between the contributor and maintainer, which is just delaying things.

Even before LLMs, writing the code was not the main bottleneck for me. But writing code did take time, so a solid, working, easy-to-review PR was often worth the small extra risk and inconvenience.

With LLMs becoming quite good at implementing things, that tradeoff is almost never true anymore.

While I still need to review LLM-generated code, I generally don't have to worry about it being malicious the way an unknown contributor's code could be. I've already codified a lot of my coding preferences and style guidelines for my LLM. And I can rapidly iterate at my own pace without having to synchronize with another human who might be in a different timezone.

For these reasons, it's just easier if I make the code changes myself (with the help of an LLM).

### The nature of software development has shifted

It's increasingly apparent that "the source code" is less "source" and more "code" — an intermediate formalized layer between ideas in the developer's head and instructions for the computer to execute. It's always been this way, but now, with the code itself being easier to generate automatically, it's just more visible.

There's a wide range of reactions to coding agents out there, from banning them to proclaiming that coding is dead and vibecoding is the future. Personally, as things are right now, I sit somewhere in the middle. I come up with the design, then let my agent do a lot of the actual writing, and then I review and refine the result.

I could get huge amounts of code written, but I'm bottlenecked on:

- understanding — reading the existing code to be able to reason about it;
- designing — coming up with the right changes and architecture;
- reviewing — ensuring that the code is doing what I wanted.

The code in your PR doesn't help me much with any of these. So let's skip it — don't attempt to implement code changes with the goal of merging them into the codebase.

### How can you help instead

As the "writing the code" part is becoming less valuable, all other ways of helping maintainers become relatively higher value.

#### Give feedback

As I'm busy implementing things, I often don't have much time to actually use them, or do good research on how to improve them.

Users telling me what works well and what could be improved can be very helpful.

#### Discuss ideas

I don't know everything, and discussing things with other people with different experiences and perspectives can help me understand what I should be building and how.

#### Report and investigate bugs

A good bug report is 3/4 of the bug itself being fixed.

If you spotted a problem, please describe it well, and even do the debugging to figure out how to reproduce it and where exactly the problem is.

Then discuss potential solutions.

#### Prototype changes

Send me a reference PR and/or the prompt you used to produce it.

Yes, I know I just said that I don't want your PRs. So let me explain. With LLMs, it's easier for me to get my own LLM to make the change and then review it myself.

BUT — using code for illustrative purposes still makes sense. A quick glance at code implementing something can be helpful, even if I don't end up merging it.

And if you share the actual "source" (prompt) to produce the "code", I can reuse and refine it, saving time.

#### Review code and point out problems

As I'm bottlenecked on reviews, an extra pair of eyes helps.

#### Fork the code and report back

Don't be afraid of forking the code and changing it however you want.

Having to come up with designs supporting multiple use cases, forming consensus, debating best outcomes, looking for compromises, etc. is very time-consuming.

LLMs enable a great deal of software customizability. You can make the changes you want by yourself faster and easier than ever, and then rebase them (or not) on top of upstream at your own pace.

Just fork. Add support for your own use case, do things your way, ask neither for permission nor forgiveness.

As a maintainer, this saves me time too. And in the end, maybe both of us can learn something from your version taking its own route.
