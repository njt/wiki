---
url: https://gist.github.com/njt/6f01cc4ac4835314dabf86b50c06947c
date_fetched: 2026-07-05
backfilled: true
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Coding agents can tackle the hardest AI engineering problems — not just boilerplate code, but systems-level work like writing custom CUDA kernels, fine-tuning LLMs, and running autonomous research loops. The speaker frames this as a way to keep engineering careers contemporary and move “closer to the silicon.”

Standard repos on the Hub are the boring but essential enabler — distributing kernels, training scripts, and experiment artifacts as versioned, discoverable repos turns agents into publishers and makes agentic workflows repeatable and shareable.

Three escalating “boss” levels of agent autonomy:

Interactive kernel writing — agent-assisted CUDA kernel generation with benchmarking, yielding easy compatibility speedups (e.g., 94% on Qwen2.5-0.8B for H100).

Zero-shot fine-tuning — a single prompt triggers dataset selection, training, and model push to the Hub, using HF CLI skills or Unsloth for cost efficiency.

Multi-agent auto-research lab — a team of agents (researcher, planner, workers, reporter) iterates on training scripts, runs experiments on HF Jobs, and monitors progress via an open data dashboard.

Open primitives beat abstracted APIs for agents — tools that expose their data layer (like Tracheo’s parquet store) let agents operate freely; hiding internals creates a ceiling. “We don’t always need to extract, it’s more about exposing.”

Skills turn zero-shot tasks into few-shot — file-based context (examples, scripts, references) that agents can open on demand, maintained alongside projects, dramatically improves agent performance on specialized tasks like kernel writing.

Pithy and provocative quotes

“Your coding agent should do AI systems engineering.” — the talk’s thesis, framing agents as capable of the hardest engineering, not just scaffolding.

“Maybe the boring part is that in order to do this, we’re gonna need standard repos and we’re gonna need those on the hub.” — acknowledging the unglamorous infrastructure that makes agentic AI engineering possible.

“In most cases, memory is usually the bottleneck. … the GPU is often waiting idle for these tensors to come back.” — a crisp explanation of why custom kernels matter, and why agents targeting memory-bound operations can deliver real wins.

“We keep the GPUs warm.” — the colloquial goal of kernel optimization, repurposed for agent-driven efficiency.

“It takes a task from being zero shot to being few shot.” — on how skills give agents just enough context to succeed without fine-tuning.

“Even though abstracted APIs are really useful, if we have a layer that we can’t necessarily get behind that is a ceiling.” — a call for tools that let agents reach into raw data, not just call endpoints.

“If you think this was completely wrong, come and find me afterwards and bully me, that’s fine.” — disarmingly honest about the experimental nature of the work.

Tools, practices, and methodologies

Hugging Face Kernels — a repo type on the Hub for distributing custom CUDA kernels with a TOML file specifying hardware/software compatibility. Agents can publish kernels just like models, turning kernel writing into a shareable, versioned artifact.

Skills (file-based context) — markdown or script files that give agents examples and workflows for specific tasks. Maintained inside project repos (e.g., a kernel-writing skill with benchmarking scripts). The speaker recommends using them to make agent tasks few-shot; check the huggingface-skills repo for experimental ones.

Upskill — an open-source library that generates skills, creates evals, and compares models on skill accuracy and token usage. Use it to iterate on skills and switch to cheaper models without losing performance.

HF CLI skills + Unsloth — pre-built skills for fine-tuning LLMs on the Hub. Unsloth provides optimized model loading for even cheaper runs. The speaker points to blog posts with free credits to try them.

Open Code (agent configuration) — used to define multi-agent setups with roles (planner, reviewer, workers, reporter) and templates. The auto-lab example shows how to orchestrate a research loop with a job queue and branch-based experimentation.

Tracheo — an open-source dashboard with an open data layer (parquet). Agents can read/write metrics directly, create custom visualizations (e.g., Gantt charts), and set up notifications. The speaker calls it “the best agent dashboard tool” because it’s just a data store.

HF Jobs — compute on the Hub that agents can launch programmatically. Workers in the auto-lab start training runs here, using labels for filtering.

HF Papers — a CLI to search papers on the Hub. The researcher agent uses it as a literature scout to formulate hypotheses.

Multi-agent research team pattern — a conceptual architecture: a researcher finds papers and proposes hypotheses, a planner maintains a queue of experiments, workers implement changes as training scripts, and a reporter monitors everything via Tracheo. The speaker implemented it in Open Code, Codex, and Claude.

Unanswered questions and omissions

How reliable and safe are agent-generated kernels? The talk shows a 94% speedup but admits it’s “not state of the art” and only about compatibility. No mention of correctness verification beyond benchmarking, or the risk of subtle numerical bugs that could silently corrupt model outputs.

What is the real cost of the auto-research lab? Running many parallel training jobs on HF Jobs could burn significant compute budget. The talk doesn’t discuss cost controls, experiment pruning, or whether the improvements justify the spend.

How much human oversight is needed? The multi-agent setup runs for hours autonomously, but the speaker doesn’t address when a human should intervene, how to detect “rogue” agents, or what failure modes look like (e.g., agents getting stuck in loops, proposing nonsense changes).

Skills maintenance and quality — skills are maintained by project maintainers, but what happens when they go stale? No discussion of versioning, testing, or community contribution models for skills.

The elephant in the room: distributing agent-generated code on the Hub — if anyone can publish kernels or models via agents, what stops low-quality or malicious artifacts from polluting the ecosystem? The talk sidesteps governance, review, and trust.

Generality beyond Hugging Face — the entire workflow is Hub-centric. How would this approach work with non-HF infrastructure, or in environments where the Hub isn’t available? The speaker doesn’t address portability or lock-in.

Evaluation beyond speed — the auto-lab optimizes for bits per byte, but real-world model quality metrics (accuracy, robustness) are absent. Could agents inadvertently degrade model behavior while chasing efficiency?

Agent coordination complexity — the multi-agent system uses tables and templates for inter-agent communication; the speaker admits it’s “a little bit of a verbose example.” No discussion of how robust this is when agents misinterpret each other’s output or when the queue logic fails.

Hi everyone. As you heard, I'm Ben from Hugging Face. And the talk that I'm going to present to you today is called your coding agent should do AI systems engineering. So there are two main takeaways that I want you to get from this talk. One, and probably the fun part is that we can use coding agents to tackle the hardest engineering problems in AI, so systems engineering and machine learning engineering.

And maybe the boring part is that in order to do this, we're gonna need standard repos and we're gonna need those on the hub. And in many cases we already have them. So I think in this case I'm preaching to the choir here. But in case you haven't noticed, coding agents have been accepted. Many of us have been using them for a few years, but in the last few months they seem to have crossed a sort of acceptance gradient where a broader group of people are using them.

So with this in mind, how do we keep our careers, our engineering, kind of contemporary and how do we keep challenging ourselves in new areas? And my proposal is that we need to go kind of closer to the silicon and tackle harder problems. And that's where AI systems engineering comes in. I've broken this talk down into three progressively more complex steps and more autonomous steps as well. And I've defined those like three bosses from games.

The first one is a hybrid approach where you interactively use an agent to solve to write a CUDA kernel. The second is a zero shot task where an agent takes a prompt and trains an LLM on hugging face. The third is a multi agent auto research setup like a kind of automated AI lab. So let's get started on the first boss. Right, this is writing CUDA kernels.

So for a while, writing custom kernels was seen as this unattainable goal for the humble agent. They required complex DSLs, they required integration with relevant hardware to be benchmarked and to be tested. And it was seen as something that couldn't be achieved by agents. However, that in most cases was wrong. If you look at kernel hackathons like those on GPU mode, the recent AMD hackathon, if you look at papers like Kernel Bench, you'll see that agents are able to write valid and optimized CUDA kernels.

And, and that's really cool and something that totally inspires me, I'm a part of GPU mode, I contribute to that and, and something that I think everyone should be doing. However, what do we do with them, how do we distribute them and how do we get them into our Inference engine so that we actually using these optimized kernels that we're generating. And that's part of the question of this, part of the talk. Let's take a step back now and just say what a kernel is, right? So when you run an AI model on a gpu, the actual work is executed through a kernel.

This will be defined in a relevant language for that hardware and it will use relevant features to that hardware that may not be available on other hardware. We can write custom kernels that will take advantage of that hardware for a specific math operation, kind of squeeze everything we can out of it so that the model will infer faster. In general, this requires a lot of expertise about writing CUDA kernels, about the hardware. And it's also a bit of an installation hell as you deal with a pretty large install matrix from hardware to software to generations and versions of say, CUDA and these kind of issues. So in short, it's hard efficiency in deep learning.

So efficiency in kernels is split into three main sections. One compute, two memory and three overhead. Compute is the flops. This is, these are the matrix multiplications and the real math of the process. Memory is the time spent moving data or tensors around memory, typically from slow to fast.

Memory and overhead is basically everything else, the Python environment, Pytorch, dispatch of those kernels, these kinds of things. In general, most people might assume that the compute is the bottleneck here because it's doing most of the math right? That's not correct. In most cases, memory is usually the bottleneck. And that's because a modern GPU, let's take a H100 for example, can do a petaflop a second of computation, but its memory bandwidth is 3 TB.

So in short, the GPU is often waiting idle for this tensors to come back for them to be computed. There are custom kernels, custom optimized kernels that exist, Flash attention being the poster child of these. And in general, what they do is increase arithmetic intensity. They basically make the GPU do more sums at once per read and write. So we move the tensors across, we, we do as much math as possible in the GPU in one go and then we write it back.

In short, people like to say we keep the GPUs warm and that's the objective of writing a custom CUDA kernel. Hugging Face has a library called kernels, which is maintained by kernel writers. And we're beginning to scale up to a kind of agentic workload. So at its core this is a way of distributing kernels. It has a TOML file like any kind of project which says which hardware it works on which versions of Cuda and other kind of softwares it requires to work.

And it's a, it's now also a repo on the Hub, just like models. So if you are a kernel writer or you're an aspiring kernel writer with an agent that you want to set up, you can now be a kernel publisher, just like a model publisher. And my point is that this is like a kind of super fervent ground for AI engineers looking to kind of scale their career. If you check out these repos on the hub, you'll see that they're, there's compatibility for different hardware. You can configure that.

So you'll know like, okay, this works on my GPU or on my laptop. And this is what it looks like here, right? Let's take a look at what this looks like for an agent and how we're helping an agent to do this. So first we're going to go to how we do this. So, skills.

So I'm sure everyone here is familiar with skills and I'm sure there've been a number of talks that really go deep into skills. I don't like to, I like to keep them pretty simple and really they're just kind of file based context with all the wonders of files. We can open them and close them, we can version them, we can source control them and these kinds of things. And agents can also do the same. They can open them when they need them, they can use them when they don't.

And so in the context of kernels, that means that we can give examples of how to write and how to use kernels in skills and they can open those and use them when they need. I like to say that it takes a task from being zero shot to being few shot, which in ML is quite a familiar concept. Right? We're just giving the agent examples of how to do things and we can be quite verbose and descriptive about that. At Hugging Face, we're focusing on integrating skills into their projects.

So what you'll find is that inside each project there's managed skills by that project, which we think is the best way to do this because it means that those projects, the, the maintainers of those projects are maintaining their skills. Right? That means that they're not necessarily the most like YOLO skills because they're kind of like well maintained and robust. And we have another repo for those kind of more experimental skills, which is called hugging face skills. Go and check that out.

If you want to try some of these examples you'll see today in kernels this is what the skill looks like. It focuses on benchmarking so it has scripts that allow you to benchmark and test the skill. Sorry to test the kernel and see how performant it is and references with examples of how to do this. We benchmark this skill and we generated a kernel for QEM3.8B for H100 and we found that we had a 94% speed up. This isn't a state of the art speed up on this model by any means.

It's really just about compatibility and a compatibility matrix. So in many cases these models and their kernels won't be optimized for the respective hardware or generation of hardware that you want to use them on. So you have some low hanging fruit here where you can just come and pick up some optimizations for that specific hardware. Maybe because your hardware is cheap on your cloud provider, but it's not necessarily the most ideal for that model that you're using. So my recommendation would be to come here and pick up some easy speed ups.

How do we know that these skills are any good and that we should be sharing them and telling people to use them? We use an open source library called Upskill that we're also maintaining. This is a gateway to using cheaper and open models with skills. It basically just generates skills, generates an eval for the skill, and then allows you to compare different models on the same skill so you can see things like this. So okay, GPT OSS is slightly less accurate using the same tokens.

Kimi is more accurate using less tokens. Haiku is a bit more accurate using less tokens and these kinds of things. So if you've got a skill and you're using it regularly and you're thinking to yourself, okay, how can I save a few pennies here and get a different model on the go? Then try out Upskill and it allow you to iterate on your skill and improve it. Right, let's move on to Boss two.

I'm going to go through this one pretty quickly. This is about fine tuning models if you're really into this. There was a talk yesterday by my colleague Merve that went into this deeply. There's also a blog post here where we got Claude to do this. This was from back in November, December time.

Now go and check this out. Basically you can just say fine tune Qin 36B on this data set. This is a chain of thoughts dataset and you'll improve the model's chain of thoughts. This is fully integrated to the hub now, so you can even run the GPUs on the hub. And, and it uses HF CLI skills, so it's all very available.

I would try this one out. You can also try this one out. This is uses Unsloth, so it's even cheaper. This runs with like optimized models and it's maintained by Onsloth and by us. And it's another blog post.

And there's also often free credits that you can get around these blog posts. So I go and check these out. Okay, let's move on to the, the big one. Uh, this is Auto Lab Multi Agent Research, which is a project that kind of basically keeps me up at night. Andrej Karpathy, a few weeks ago, maybe a month ago now, released a project called Auto Research, which was based on his other projects, Nano GPT and nanochat.

And it took the Nano GPT architecture and got Claude code to create, improve. To write improvements to that training script so that it would improve the training process. So we can see here the experiments going over and for each experiment there's a change in the training script which increases the efficiency measured in bits per bytes of that run. And we can see that the efficiency ends at its best at the end of the process. I, like everyone, thought this was super cool and I had to start implementing it straight away.

But one of the things that stood out to me was I found it kind of weird that we had one agent working in a single way, iterating, going and finding improvements and then implementing them. And it would make sense to kind of distribute this. So that's what I did. I distributed the task amongst a research team with four types. We have a researcher that basically looks up papers for this.

We use HF papers, but we can also use archive papers. HF Papers is cool because it has a CLI so you can just pull and search papers from the hub and it acts as a literature scout. So it just looks up for papers with ideas and, and it formulates those as hypotheses. We then have a planner which takes those hypotheses and maintains like a queue of jobs. We then have a set of workers and they pick up those hypotheses and their job is to implement them as training scripts.

So in many cases just like change the architecture or change a parameter or something. And then we have a reporter agent that goes and monitors all these jobs and maintains a dashboard that we can use. So this is what it looks like. If you see here that we have. We're working in a GitHub project, right?

So in a git project, sorry, and we have a main branch that we maintain with our train scripts that we're updating in each branch. And then like a train original that we keep. And then we have a data structure on the main branch that we use to just keep the scores. Then we implemented this in open code for this example, but in the repo, which you can also go and check out. They.

It's also implemented in Codex and Claude if you want to try those. I also implemented it in Gastown, but that's kind of Wild west stuff. So I did it in like a separate project. Um, but basically it works really anywhere because it's more just a conceptual implementation, right? And first you have your planner creating hypotheses.

You have your researchers looking at paper and then your reporter picking all of this up, handing to workers. As I said, those workers integrate with HF jobs. So they start these jobs off on the hub and that run with the hardware that they need and then they submit these patches that go back. The reporter operates in Tracheo, which is an open source dashboard that we use for all metrics. Drakio is useful with agents because it uses a completely open data layer, basically parquet.

So if you don't want the dashboard, or your agent doesn't want the dashboard for any reason, it can just get into the parquet and just do whatever you want. So if you need a Gantt chart or some other visualization, it can just go and do that. So I would say it's like the best agent dashboard tool because it is basically just a data store. It's basically just a data structure. Okay, so let's just walk through this now.

So this is implemented in open code. If you don't know open code, you have like agent configuration. So in this one I just set Auto Lab, which is the name of the agent configuration I have. It has skills. This was the prompt.

So it says like run one autonomous local research or auto research pass in the repo using defined roles. I tell it to use planner to propose up to fresh single change experiments. Use reviewer to reject duplicates or stale ideas. I also tell it to use like a HF bucket because I want all of the storage to be in the same bucket so that I don't have to upload or download the training scripts every time. And then we go and we select one of the sub agents.

There's an ISO interface in open code, but it's similar in other tools. So I select the planner and then you'll see that the planner receives this prompt and it uses a specific template which I defined in my configuration of like it's going to have current state, it's going to have a list of the jobs so far, things that have worked which were defined by the reviewer, current hyper parameters that it can change. And it's basically just defining these jobs which will go onto the job list. As I mentioned, we then switch over to a reviewer agent which will receive all of these jobs. And it has a similar kind of structure based on a template, a reference to where it should be working from and the latest score that it should be using.

It gets an overview of all the failed and successful experiments which it will then use to base its decisions of what goes into the next Q on. And it creates this little table which we don't really need to look at. It's really just for the agents to interact with each other and to get this information back. To be honest, that's a little bit of a verbose example and we maybe don't need this many tables and you could probably trim that bit down. But in general I'd recommend that if you think this is cool, go and try that out in the repo after that.

So this agent runs in parallel, sometimes for hours. And this is the Tracheo dashboard that we use. And these are all the runs that are pushed to Tracheo. As I said, the main advantage here is that this is fully open source and it's just a data layer, but we get all of these kinds of visualizations. Tracheo can also have like events and warnings, so we can have all of these events being reported by different agents and we can filter those down.

We can also even tie those up to like notifications. So you can get emails from Tracheo if you want. If like your agents are kind of going rogue or something, you need help. But best of all, Trakio just has this like just freeform structure. So you can just throw tables in that don't necessarily fit with any other structure.

And then on the hub side, all of these jobs are just run inside hugging face. So you can explore those jobs and in most cases you can tell the agents to use like labels and you can sort those labels and review through what they're doing. Or you can just look at it like this. As I mentioned, you can access that underlying data layer and just create a Gantt chart because this was a kind of convenient way to look at what the agents were doing over time. So you can see like this amber agent went off and this was the score that it got.

But you could visualize this however you want because you have access to this data layer. The kind of TLDR of the whole thing is that, yeah, you can go and just have your kind of own AI lab and you can try it out. And if you have a verifiable experiment like training a model or doing or writing CUDA kernels, then it's pretty easy to implement and set up and to learn some stuff. So let's now look at the takeaways, I'd say. So in simple terms, I'd say that agents work really well with primitives and open primitives.

And we want tools that are fully open, things like tracheo, things like kernels that we can expose to agents and they can kind of control in their own way. Even though abstracted APIs are really useful, if we have a layer that we can't necessarily get behind that is a ceiling, so we don't always need to extract, it's more about exposing. Well, and the other takeaway is that the hub is ready, the Hugging Face hub is ready for these kind of workloads. We have the fundamentals in place like storage, tracking and compute, which I think will allow us to scale our engineering to, yeah, new levels. If you found any of this interesting, I've shared it all on X, I've shared it all on Hugging Face and there's a blog post about basically each one of the examples that I just shared with you.

And they all have repos attached to them, so you can go and try that out for yourself. If you find anything that's broken, like, please take me off, if you think that this was completely wrong, come and find me afterwards and bully me, that's fine. But most of all, thank you.
