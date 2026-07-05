---
url: https://gogogolems.substack.com/p/slowing-down-in-the-age-of-coding
date_fetched: 2026-07-05
backfilled: true
---

# Slowing Down in the Age of Coding Agents

### Using AI for Deep Thinking, not Tokenmaxxing

Everyone is trying to go faster, running more agents to run more agents, to open more pull requests to orchestrator agents to then get reviewed by another fleet of agents. The arrival of coding agents has turned software development into something that looks like a factory (or casino) floor — and the temptation is to optimize for throughput, to feel the dopamine hit of watching five Codex tasks resolve while you start three more. I’ve been there at points in the past and it’s a wild experience to see it happen to a significant chunk of my profession.

I decided to take break from X in august and to go back to playing with LLMs on “my own terms”. I went the other direction, I slowed down. What’s the point of AI if it doesn’t allow me to become more in the world.

In the previous two articles, I argued that LLMs inherit complexity from our codebases and our conventions and that the representations we choose determine how well the model can do its job. The hard part is the human part, the part that demands creativity and hard work: recognizing which complexity is essential (which only works if you put in the hours necessary to grasp the core of a technology), choosing the right notation (which requires practice and failing), and shaping the task so the model can handle it (which requires continuous practice; imagine getting a new instrument every 3 months).

This article is about what that looks like in practice — how why it involves a lot of handwriting and stepping away from the computer.

## Optimizing for thinking time

The bottleneck in agent-assisted development is no longer writing code. It’s the quality of the ideas you feed into the context window: a mediocre prompt produces mediocre architecture at high speed. Compound with that with a ridiculous amount of skills and tooling and complex harnesses, and you make it happen even faster. An effective one — one that makes a complex problem simple (through abstraction, notation, architecture), which in turns makes the task simple for an LLM — produces something you can actually build on in the long term.

So I optimized for thinking time. I take two or three hours most mornings at a coffee shop — reading books, papers, and Deep Research outputs, not writing code. I do my drafting on an e-ink typewriter (it allows me to type in the sun), where the screen is slow and there’s no browser to tab to. I read papers and design documents on an e-ink tablet with a pen, because the act of scribbling slows my brain down enough to have the second and third thoughts that matter. I’ve always drawn software more than I wrote it, and have accumulated many sketchbooks over time.

This isn’t about rejecting technology or really an affinity for analog, I usually try the more “digital” version of a device, but somehow it is only I turn to the “human” version that I really fall in love with a device. LLMs themselves eliminated one of the biggest obstacles to deep thinking — the web search rabbit hole. I can ask a question and get an answer without losing twenty minutes to tabs. Tools like Kagi help too, surfacing small blogs where the more interesting ideas live, outside the algorithmic mainstream. The analog parts of my workflow exist because they create a different kind of cognitive space — slower, less reactive, more generative, more personal.

I enjoy when a Codex run takes twenty minutes. It means I can step away, think about the problem from a different angle, or read another section of whatever LLM document or paper I’m working through. I usually have one or two tasks running, but I don’t feel pressure to maximize parallelism. Most of my brain is already occupied with thinking hard about the *shape* of what I’m building — and that work can’t be parallelized. I *can’t* do more; anything on top would encure a task switching cost.

## The Annotation Cycle

### What the agent produces

When I start work on a problem, I follow a fairly standard LLM coding technique: I don’t start with actual code. I ask the agent for a design document — an analysis of the problem, an implementation plan. I’ll often also read the diary of what it did during a previous implementation session (which is also a way to maintain memory over agent sessions). These are structured markdown documents, cross-linked and organized by a small tool I built called docmgr, which gives every project a consistent document structure I can navigate quickly. These documents however contain a lot of code: I explicitly ask for file and symbol references, API signatures, code snippets, pseudocode. It is basically a literate programming document.

### Reading with a pen

I upload these documents to my e-ink tablet using a fairly sprawling tool (also semi-vibed: I gave the agent SSH accent to the device so it could capture the screen for a feedback loop) and read them with a pen. This is where the real work happens.

I read these fairly carefully, not just with the mindset of reviewing and controlling a coding agent, but to learn the codebase. I have a pretty bad memory and cannot remember code for the life of me. I can however remember the shape of a codebase, the building blocks it contains, the affordances of its APIs. Having a literate programming for literally every commit is life-changing.

When I read a design document, I’m asking myself one question at every paragraph: *can I trace this back to something I asked for?* I am particularly on the lookout for symbols or jargon I don’t recognize. These could be either a part of the codebase I don’t know or remember, a concept or algorithm I’m not familiar with, a problem with my prompt (using the wrong keyword, referencing the wrong file) or the agent following down the wrong path. Some of these I will correct in a follo-up, but depending on the tool I use, I will go back to the original prompt and modify it and start a new agentic loop.

I annotate everything: questions, corrections, things I don’t understand, connections I notice between sections. I use colors and arrows to link related ideas. Most of this is inner monologue — I write in the margins partly because the physical act of forming letters slows me down enough to sit with a thought longer than I would on screen.

### Tracking vocabulary

One thing I pay particular attention to is vocabulary.

When an LLM generates a text, having been trained for next-token prediction and using an autoregressive attention mechanism, every word it generates is basically a query of the training corpus: the LLM will “fetch and recombine” words. Every word will in turn influence something that comes later. Because we persist the LLM output as code, this could mean influencing code 12 days down the line. The more a word is put in the limelight (a class name, a variable name), the larger its impact. And LLMs (and especially anthropic models) love to generate new words and adding new concepts and new features and twisting existing jargon into new jargon. And then they pile further layers of words around their frustrating tendency to keep legacy wrapper and feature flags.

Some of those words are precise and useful — a name for a concept that fits the project. But many are jargon imported from the training corpus or from unrelated context: Controller, Manager, Registry, Manager, Interface, Wrapper. The model doesn’t distinguish between a word that maps to a real concept in your codebase and a word that *sounds* like it should. We all know how much programmers like to argue about these choices: the training corpus itself is very inconsistent, surfing on the latent space can easily lead you down a different mountain.

This is where the simplicity argument becomes key: if you have the right vocabulary and syntax, language is clear and unambiguous. In that article, I described how LLMs inherit the cargo cult — the architectural complexity baked into their training data. Vocabulary tracking is how you catch it at the word level, before it metastasizes into unnecessary abstractions across your codebase. These unnecessary abstractions and competing architectures and an impressive amount of unit tests (which I always found to be on the side of harmful, reifying the current architecture and causing models to focus on the wrong things too early. GPT-5 was a major improvement in that regard) turn the codebase into the mesmerizing, fractal equivalent of a diffusion model’s oddly fingered hands and hallucinatory yet tepid iconography.

### Filtering and feeding back

About 90% of my annotations don’t serve a real purpose. They’re thinking-out-loud marks that served their purpose on the page. What comes out of a reading session is a short, filtered list:

- Clarification requests — things the document asserted that I don’t understand or can’t verify 
- Corrections — things that are wrong or don’t match my intent 
- Vocabulary investigations — words I want to trace back to their origin in the context 
- File references — specific source files I want to read or have the agent re-examine 

I type these into the agent session or open my IDE (jetbrains, ironic how cursor manages to be insufferably slow compared to a full featured IDE), sometimes continuing an existing conversation, sometimes launching new research threads. If the questions are architectural, I’ll ask for an updated design document and do the cycle again. If they’re implementation-specific, I’ll ask for targeted tasks, now cross-linked to the analysis I’ve reviewed. The cycle is traditional: design → review → refine → implement (nothing special here, except that it often spans 2 or 3 20 minutes codex runs. What’s different is that the “review” step happens on paper, with a pen, at the speed of handwriting.

## Prompt engineering is hard

The handwriting slows me down: manually entering my prompts make me to decide what matters (I played with the idea of submitting my tablet notes in an automated fashion, but quickly realized I didn’t have the prompting control I needed). The vocabulary tracking catches jargon drift before it compounds into architectural bloat. And the whole cycle builds a mental map of the codebase that no amount of “just read the diff” would give me.

This is the work I described in the previous articles, made concrete. When I wrote about simplicity, the core claim was that the hard part is recognizing which complexity is essential and which is inherited — and that the LLM can’t do that for you, because the conventions that created the complexity are the same conventions it was trained on. The annotation cycle is how I do that recognition: slowly, on paper, one paragraph at a time.

When I wrote about notation and generalization shaping, the argument was that the right representation makes the model’s job tractable. The prompts and design documents that come out of my reading sessions are better representations — more precise, less contaminated by generic patterns — because I’ve spent time thinking and had the opportunity to switch “mental environments” before entering my final prompt in the context window. In order to properly linguistically simplify the problem, I order to understand the implication of each word in my prompt and I can only do that if I slow down enough to notice what’s in there.

## Thinking slow

I don’t think everyone should buy an e-ink tablet and annotate documents by hand. The specifics of my workflow are shaped by how my brain works — I need quiet, I need to write things out, I need color and arrows and margins full of questions. Other people think differently.

But the underlying shift is general: the bottleneck in agent-assisted development is no longer writing code. It’s understanding what should be written, recognizing what shouldn’t be there, and choosing the words that will shape what comes out. That work is slow, it has to be.

Just curious, What eink tablet do you use? Also what’s the terminal that looks like a 70s mainframe?
