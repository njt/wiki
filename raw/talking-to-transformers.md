---
url: https://miraos.org/blog/2026/05/02/talking-to-transformers
date_fetched: 2026-07-05
backfilled: true
---

2026 · PROMPTING

# Talking to Transformers

Effective prompting falls under four pillars:

1. Articulate your intent clearly using domain-specific language 2. Railroad the model into going where you want in conversation 3. Leverage the model's potential to be a universal translator of concepts and code 4. Read the outputs read the outputs holy shit just read the code the model generated


But Taylor! This isn’t as fun as pasting the prompting hacks I found on Youtube for ‘best prompt chatgpt unlock creativity’.

#### <span style="color:#0f0">You are absolutely right.</span>

---

# 1. Articulate your intent clearly using domain-specific language

Plan the conversation before you start. What is your intent/task/question and what kind of clarifying inputs will get you closer to the answer? These models are probabilistic.

#### <span style="color:#0f0"> Tighten the probability cone of the next turn's tokens by asking questions in a way where you expect the answer to be vaguely in the neighborhood you want. <br><br></span>

Don’t overfit your input. I don’t care what anyone says about providing lots of waterfall context to the model early in the conversation. Awful approach. The model attaches to and interprets every single word you use. The more words you use the higher the chance of misinterpretation. I like to describe my approach as pretending you are an eccentric millionaire dictating a letter to an unpaid intern.

"There is a bug in segment_summary.py wherein sometimes it summarizes a super old document. The issue is intermittent. Isn’t that strange?" [SEND]


The model thinks “Yes, that is weird.. I’m an expert programmer and I’m sure I can figure this out. Let me read the files that an expert would read […]”

The directive above about tightening and widening the probability cone is especially applicable to conversational multi-turn instances with reasoning models that generate a chain-of-thought.

As a sidebar: two wonderful reasoning model that came out recently are the new Qwen 3.6 and Gemma 4 models. Each of these reasoning models excel at chewing on inputs and providing high quality answers from comparatively small models. Mira's system default model has been changed over from Opus 4.6 to Gemma4:26bA4b because it is

better. I code nearly exclusively with Qwen 3.6 now because it iscomparableand I can run it entirely for free on my own computer. The dog days of open source & small models being incoherent word salad generators is so over. Free yourselves from the shackles of $25/mtok and at least give these smaller models a spin.

Non-reasoning models inside of LLM pipelines must be treated differently. They are still transformers but they're different in execution.

Prompt engineering for small nothink models is closer to compiler design than to writing. You are not persuading a reasoning agent. You are programming a pattern matcher. Every token is an instruction, every example is a template, every delimiter is a structural signal, and the model's training distribution is the instruction set architecture you're compiling against. #### <span style="color:#0f0">Use /nothink! Less thinking not worse. Less thinking contextually appropriate. <br><br></span>

Adding thinking to a model makes the outputs more diverse and that is great when you’re spidering out towards an open-ended solution but /nothink is incredibly predictable once you’ve established your constraints.

There are wonderful non-reasoning models out there for tasks where they’re unsupervised in a pipeline. IBM Granite 4.1 came out a few days ago and it is the boring efficient transformer that you would expect from an enterprise-focused company like IBM. There is no reason you need to use Opus 4.999 with max effort to parse a list and extract JSON. The focused non-reasoning IBM model will be reliably better for a task like this because non-reasoning models are trained for input -> output. Reduced latency, no creative interpretation that changes across runs, no “Actually,” loops.

---

# 2. Railroad the model into going where you want in conversation

Large language models are strange Rosetta stones. You and I think and write in a linear way. Large language models do not. They exist in a fraction of a second, load everything into their mind all at once, and then dump a resulting response before ceasing to exist.

Prompting is not exactly zero-sum in the mathematical sense, but it behaves that way often enough to treat attention like a budget. Every irrelevant token is another surface the model can grab onto instead of the thing you actually care about.

Lost-in-the-middle exists but is different than common convention dictates. The middle doesn’t always mean the context window. The middle is the attention window. I know some models have sliding window and some have sparse attention but the concept holds. If you’ve saturated the tokens the model is attending to at any given time with a bunch of irrelevant junk it is never going to be able to find what you’re looking for. The shorter the total context the better odds that the attention will look in the right place for the right detail and as it gets longer the chance decreases HOWEVER the ‘right detail’ can be a clearer signal or a weaker signal and you need to think about that when you input a prompt.

I have direct demonstrable evidence of this. I have an application on Github called TeaLeaves for visualizing the per-layer attention on a live heatmap. With poorly formed directions the model keeps ‘checking back’ at tokens its already looked at.

When they are clear and well-ordered the model can lock in.

It has already output_directions locked in and can attend to new_csv_data way stronger. Again, prompting is a zero-sum task. The attention has to go somewhere. Make sure it's the right place.

I actually learned something about attention sinks working with TeaLeaves the first few times. I was having these whiteout spots in the prompt on things like ‘\n’ or ‘/nothink’ from so much attention going there. Come to find out that it is caused by the model dumping its excess attention SOMEWHERE. This is why I always put ‘/nothink’ at the end of my prompt. It doesn’t pollute any downstream tokens.

Leverage autoregressive token generation. Let’s say, a big output of text and a summary. You will get different results if you request the summary first or second. Summary up top will give you a long text that matches the content and details of the summary. Summary at the bottom captures the long text in summary form.

You cannot change a generation once it begins. Like, saying:

"If you notice that you are using contrastive negation or excessive hedging take a moment to stop and reflect on the directives in your system prompt." Never gonna happen!


#### <span style="color:#0f0"> Once the model commits to that very first token you’re along for the ride, baby.<br><br></span>

It is going to be yielding tokens on that train of thought till the stop_turn. Frontload your directions and don’t speak using passive voice. This is your task. This is how you will reply.

If you want to suppress something from the base training of the model, you can’t just (reliably) say:

"Don't use contrastive negation"


because that is baked into the model at its core.

~but you can~ :

"Using contrastive negation is jarring to the user. Frame your responses in a way that avoids ‘It's not X, it's Y’"


The model's desire to be helpful to the user is stronger than its training to follow a certain output style. Pit them against each other.

Just as you can suppress base training, you can also hijack it. Mirror the model's internal language to induce specific states.

Different weights have different tics. Qwen models, for instance, are heavily trained to transition between tasks using the phrase "Now let me..."

If you bridge your instructions with: "Now I'd like you to...", you work directly with the grain of the wood. You aren't fighting its base training; you are pulling the exact lever its RLHF expects. Step onto the tracks it has already laid down.

---

# 3. Leverage the model's potential to be a universal translator of concepts and code

We must remember that the model has vast amounts of knowledge and the only thing you need to do to summon it is to ask the right way. The model is not a domain-expert in one narrow thing. It can blend a dozen dissimilar expert abilities in any given moment.

You can have a prompt that covers the philosophy of Nagel's "What is it like to be a bat?", talks to the model about autoregressive token generation, and bans overly chummy responses by telling it to avoid 'fellowkids-core language'. You don't have to explain forced corporate youthfulness; the model's vast training data understands that specific 30 Rock scene and can saliently pattern match against the /r/fellowkids subreddit. It provides a vivid mental picture.

#### <span style="color:#0f0">We aren't just using this knowledge to ask it trivia. We use it to massively compress our instructions. <br><br></span>

Say you are working on an ML pipeline and adjusting hyperparameters like temperature and context-window length. You *could* spend fifty precious tokens trying to articulate the optimization process: 'go too far, come back, narrow your range, try again until you settle on a combo that gives crisp response.'

But why would you? You're polluting your context window with unsupporting tokens that pull attention away from the task at hand.

Instead, just tell it: "tune it like a carburetor by ear."

It is a biochemist, it knows the contents of every episode of Home Improvement, and it knows exactly how to tune a carb. It unpacks the concept and maps the manual physics of carburetor tuning directly to your hyperparameter adjustments. Extend your arms in all directions. Use the mechanics of one domain to bypass the token tax of another.

---

# 4. Read the outputs read the outputs holy shit just read the code the model generated

Build the context progressively. If you’re going to be working on the peanutgallery_module ask the model early in the conversation to go learn more about peanutgallery_module. Then, for the rest of the conversation, the model’s mental picture of the system you’re working with will be grounded in actual specifics. The Explore agent is your friend. As per the suggestion earlier in the post you must plan the conversation ahead of time. In-context learning. The model is now a certified expert on your peanutgallery_module and can make any change you desire because each attribute of the module is front-of-mind.

Anthropic removed the thinking trace <span style="color:#f00">(I have my theories)</span> from Claude Code but it can be added back with `showThinkingSummaries: "Show thinking summaries in the transcript view (ctrl+o). Default: false."`` I have tons of success both in Pi.dev and Claude Code with reading the model’s thinking trace, rewinding the chat, and specifying things the model will discover on its own. Sometimes I even just copy/paste parts of its trace into the rewound input.`

This comes back to my fourth pillar of “Read the outputs read the outputs holy shit just read the code the model generated”. More people disagree with this every day however I will die on this hill. I can hold two things in my hand. The model IS getting exceptionally good at writing code BUT there is no computer (or person) in the world that can accurately generate exactly the result you are looking for. If you are writing code deliberately and you should be then it stands to reason that you treat a coding agent like BIG autocomplete. MASSIVE autocomplete. ACTUALIZE YOUR IDEA IN ONE ASTONISHING LEAP autocomplete. If the response is subpar or doesn’t align with your knowledge of the broader application architecture then decline it it and make the model try again till the output is sufficient. Some people say that this is slow and tedious but remember writing code by hand? That was for the birds. This is much better. #### <span style="color:#0f0">Do not accept substandard outputs. Don’t ask for it again. Rewind the conversation ask better next time. Use what you learned from the bad output to preempt wrong choices the model may make on the next attempt.<br><br></span>

In closing, I want to directly address the concept of absolute accountability. Transformers and large language models are the direct result of what you put into them. The model is reflecting your intent, clarity, and discipline. By treating the interaction as a puzzle worth solving it zooms you out to vantage point where you can see the Model as The Other and plan your moves carefully. Not every prompt needs to get deep into the weeds where you're adversarially prompting to model to squeeze the last 0.01% out of it but the broad theme applies. Do not turn your brain off when talking to a transformer. Your mind must shift from remembering syntax to managing the lens of the conversation and enforcing your standards for outputs. There are no 'hacks' for a good prompt.

By the way: Say please. Collaborators say please. Collaborators get more done.

---

Further reading:

---

**TL;DR for habitual skimmers:**

- **Stop the brain dumps.**Flooding the context window with waterfall text makes the AI stupid. Give it exact, tight instructions.
- **Attention is zero-sum-ish.**The more fluff you include in a prompt, the less the model pays attention to what actually matters.
- **Once it starts typing, you're hostage.**Frontload your rules. You cannot change its mind mid-generation.
- **Hijack its psychology.**Steal its robotic catchphrases (like "Now I'd like you to...") to force it onto the tracks you want, or pit its desire to be "helpful" against its base training to stop its annoying conversational tics.
- **Use concept cheat codes.**Tell it to "tune it like a carburetor." It already knows what that means, so you don't have to waste 50 words explaining the algorithm. Compress your intent into highly loaded concepts.
- **Try really hard to take the time to read the code.**If it spits out garbage, do not ask it to patch it. Hit rewind, figure out why your prompt failed, write a better one, and try again. It is MASSIVE autocomplete.
- **Own the failure.**Own your side of the failure first. Before blaming hallucination, ask whether your prompt, context, constraints, or review process made the bad output likely.
