---
url: https://www.oreilly.com/radar/when-smaller-models-win/
date_fetched: 2026-09-20
---

The following article originally appeared on theAsimov’s Addendumblog and is being republished here with the author’s permission.

## Even the best AI models can suck at chess

The launch of ChatGPT had an interesting effect on the online chess discourse. Chess has already long been conquered by machines. As early as 1996 a computer (IBM’s Deep Blue) was able to beat the human world champion, grandmaster Garry Kasparov, in a game watched by over six million people.1 The world was shocked that a machine took on the best player and won a game, but chess engines didn’t stop evolving there. Since the ’90s, they have gotten better while the machines needed to run them have become much smaller. Today Stockfish is widely considered much stronger than any human player. It has run on consumer hardware since its launch in 2008, and by 2014 it was beating some of the world’s top grandmasters.

It came as a surprise to many, therefore, that modern LLMs, trained on a vast portion of the internet and requiring **supercomputers to run**, couldn’t help but cheat on almost every move. There are endless videos showing how just a few moves into a normal chess game, ChatGPT and some of its competitors would gladly throw the rules out the window to escape a checkmate or gain an advantage.

But in truth, this isn’t surprising. The LLMs were not trained with chess in mind. Sure, they may have seen countless chess games scattered throughout the internet, but the vast majority of their parameters and training compute were devoted to capabilities that are completely useless once you put a chessboard in front of them.2 Stockfish on the other hand uses a tree search algorithm that is built to be good at chess. If you want a chess engine, you use a chess engine.3

## Smaller models are sometimes better

While chess is a particularly potent example of a large language model losing to a much smaller specialized system, it’s far from unique. In a 2025 position paper, NVIDIA researchers argued that small models4 (which it defines as models under 10 billion parameters) are the future of agentic AI and that they “provide significant benefits in cost-efficiency, adaptability, and deployment flexibility.”

Just because a larger model can do a job does not mean that a small model fine-tuned for that specific task can’t do it better and more cheaply. There are countless examples of smaller models doing just that. LiteResearcher is a 4B model that beat out Claude Sonnet 4.5 on some search benchmarks. Terminus-4B allows larger models to save compute by handing off terminal execution to a smaller model without suffering capability loss. The Docling family of open source models start at just 258 million parameters and allow for fast extraction of PDFs to text without having to feed 100-page PDFs into an expensive LLM. Researchers also trained a small 4B model to outperform even the GPT-5 series of models in a few social negotiation situations such as negotiating salary or bargaining for a purchase. Each wins, not by raw intelligence but because it is built or fine-tuned for a narrower, more specific purpose.

A frontier model may know how to do all of these jobs, but that doesn’t mean it’s the right tool for the job. Large models are expensive and unpredictable, and doubly so when it comes to agentic tasks which can span several turns and hundreds of thousands of tokens.

NVIDIA draws the line for small models at 10 billion parameters, but the more important boundary for developers may be whether a model is small enough to run yourself. There is still a whole class of models that are not necessarily small but are still small enough to fit on one consumer GPU (at least when quantized). This includes models like Qwen 3.8 27B, Gemma 4 26B and GPT-OSS 20B. These models are very capable even without specialization and rank very highly on benchmarks (with Qwen sometimes outranking top models from a few months ago). But they can still be easily run on premises without spending thousands of dollars on GPUs.

The ability to run smaller specialized models adds more than just efficiency; it provides a more realistic opportunity for a developer to train and fine-tune their own model, and to host the model locally or in the cloud instead of relying on the model provider to do so for it. This in turn can provide developers more control over how tokens are used, how outputs are structured, and how each part of the pipeline can be improved individually—instead of assuming an improvement in the most popular benchmarks will lead to every task improving. And as noted above, smaller models can be easier to fine-tune, thereby creating a more specialized AI. As my colleague Ilan Strauss has noted, specialization is a powerful economic force.

## How do you train it, and where does it run?

The strongest argument for using an off-the-shelf generic chat model is often one of convenience. For most tasks a general model will be good enough, and with products like OpenRouter, developers can easily pick and choose from hundreds of models (plenty of them open source) all competing in capability and cost without putting in any upfront work to train a model. As Raffi Krikorian of Mozilla noted while reviewing this article, generic models also make particular sense early in a company’s lifecycle, when the problem itself is still being defined. At that stage, experimenting with the largest and most capable model available can help a team figure out exactly what it needs. But as the problem space narrows and the required architecture becomes clearer, so too may the need for a large generic model. And despite many first-party model makers discontinuing their fine-tuning products, fine-tuning and hosting a smaller model remains relatively easy, largely thanks to parameter-efficient techniques like LoRA.

**LoRA**

LoRA (Low-Rank Adaptation) allows developers to fine-tune a model without touching the actual model weights. It works by attaching a relatively small number of trainable weights that are updated during fine-tuning. This is important for several reasons. A small adapter can be easily transported, and serving a new LoRA does not require loading an entirely new model as long as the underlying base model is already available. Unlike full fine-tuning, a LoRA also reduces the risk of catastrophic forgetting.

Training a LoRA is much cheaper than full fine-tuning as it only updates a small selection of weights. This can be done on consumer GPUs using libraries such as Hugging Face Transformers or Unsloth. There are also APIs that mimic or improve on the fine-tuning APIs that used to be provided by the big three providers (Anthropic, OpenAI, and Google), Fireworks, for example, provides a straightforward fine-tuning API that takes example completions for it to learn from. Going beyond SFT (supervised fine-tuning, or learning by example), Tinker allows developers to build custom RL (reinforcement learning) environments that reward results meeting certain criteria, while the environment itself runs on the developer’s machine.

Hosting a LoRA is similarly straightforward and, importantly, portable across platforms. Transferring a fully fine-tuned model to a new API platform can be costly and may require the provider to serve your model separately on expensive GPUs. Using LoRA allows the platform to just load a small adapter onto the model they are already using to serve other users’ requests. This means that fine-tuning a LoRA does not lock you to a specific platform, and it also doesn’t force you to rent your own GPUs.

## Control beyond the model weights

As Tim O’Reilly previously argued, open source AI should not stop at the model weights. In a similar vein, the possibilities for developers building a custom system do not stop there either. Model APIs are inherently limiting, they impose on you what parts of the model’s input can be touched, what can be cached, and how you can affect the output. Going back to the chess example, enforcing valid chess moves at output time is easy, assuming you have access to the code that runs the model, but it’s not easy to do when you are relying on an API built for a chatbot that you are unable to modify.

Self-hosting a model gives you a level of control far beyond what is possible through a standard chat completion API and allows you to build the model around the task instead of building the task around the model. Fine-tuning is only one aspect of specialization. You can also constrain which outputs are valid, expose and modify probabilities of every token, cache any state, and add task-specific logic directly into the inference pipeline.

This matters because the default approach to improving AI systems has increasingly become to reach for a more capable general model. Sometimes that is the right answer. But improving *general* model intelligence is only one lever, and often not the cheapest or most reliable one.

In 1997 nobody complained that Deep Blue gave bad recipes because it wasn’t built to do anything but play chess. By specializing around one narrow problem, it was able to beat a grandmaster at a game that many had thought machines would never conquer.

The lesson from LLMs cheating at chess is that the best tool for a problem is often not the most general one. Super general intelligence does not automatically translate into high capabilities in specialized tasks. The opposite is closer to being true. Specialized machine intelligence requires lots of data and often its own pipeline. Small models can help companies get there.5

## Footnotes

- In 1996 Deep Blue ended up losing the match 4–2 despite a great start where it won its first game; the next year after more upgrades Deep Blue beat out Garry Kasparov by one game in a rematch. ↩︎
- AlphaZero, a model trained with self-play, beats even Stockfish. It is not unheard of for a machine learning model to get really good at chess when it is set as the goal. ↩︎
- I ended up pretraining my own tiny language model (15 million parameters) on my local Mac mini for chess as an experiment and achieved 27% accuracy of predicting a human’s next move. I suspect I can do a lot better after I fix my tokenizer to break up moves and use a bigger model but that is still up for debate. ↩︎
- This contrasts with larger models like DeepSeek-V4-Flash (284 billion total parameters, with 13 billion activated per token) and huge models like Kimi K3 (2.5 trillion total parameters, 104 billion activated per token) and presumably flagship models from OpenAI and Anthropic ↩︎
- Thank you to Ilan Strauss, Tim O’Reilly, Mike Loukides, and Raffi Krikorian for their helpful comments, copy edits, and suggestions. ↩︎
