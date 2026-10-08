---
url: https://www.oreilly.com/radar/radar-trends-to-watch-october-2026/
date_fetched: 2026-10-08
---

In addition to the nearly constant stream of model releases, in September we’ve seen price drops, new kinds of models, proofs of long-standing problems in mathematics, and continued investigations into models escaping their sandboxes. (Axios reports investigations into over 10,000 incidents.) AI has infinite patience and is fundamentally probabilistic. Given a difficult or impossible task and an unlimited token budget, an agent will eventually attempt to solve the problem in ways that you don’t expect, and may not want. It’s easy (and correct) to blame inadequate security procedures at the frontier AI labs, but AI adopters must be careful not to make the same mistakes. The humans using AI need to be accountable for what their agents do.

## AI Models

*Model choice is starting to hinge on price and specialization as much as raw benchmark leadership. Alongside general chat models, there are now decision models that never chat, spatial models built for robot planning and camera control, forecasting models sized for a single task, and cybersecurity-specialized models kept behind an invite-only program. Specialization leads to greater efficiency and lower costs, at least in the short term. In the long term, specialized models may succumb to the “bitter lesson.”*

- Anthropic has released Claude Opus 5.5, which it claims has performance similar to Fable 5.1, and hence similar restrictions. It’s faster and requires fewer resources to run. Anthropic has dropped prices 20% for input and output tokens and 60% for cached reads. Not to be outdone, OpenAI released GPT-6 Sol and Luna, with 50% price reductions.
- Anthropic has also announced Fable 5.1 and Mythos 5.1. The most significant change appears to be a 75% price reduction for cache reads, which might translate into significant savings for long-running jobs; Anthropic estimates 25%. Mythos is only available to trusted partners. Simon Willison used Fable 5.1 to animate his pelican-riding-a-bicycle pseudobenchmark.
- Anthropic released Sonnet 5.5 with claims that it’s 30% faster and 30% less expensive for most work. The new model has security limitations similar to those applied to Opus and Fable; it routes to Sonnet 5 if it’s asked to do anything out of bounds.
- And finally, as September closes, Anthropic announces a marketplace for Claude plugins and connectors. At its launch, Claude Marketplace had over 2,000 items.
- OpenAI has released GPT-6 Astra, with claims that the company has achieved AGI (artificial general intelligence). Astra’s excellent benchmark scores appear to depend on the use of an unreleased harness. OpenAI has also released GPT-6.1 Sol, with per-token price reductions and claims that it is close to GPT-6 Astra in capabilities
- OpenAI has solved the Navier-Stokes existence and smoothness problem, a mathematical problem in fluid mechanics. This development raises an ethical question: Did OpenAI train its system on the work of two mathematicians who were close to solving the problem themselves? It also raises practical questions about the future of mathematics. Decorated mathematician Terence Tao asks whether “the collection of good, fruitful open problems is now being mined in a non-renewable fashion.” An advisory group has been formed to help OpenAI make decisions about releasing mathematical results.
- Google has released Gemini 3.8 Flash TTS and Flash-Lite TTS. Voice options aren’t limited to a prebuilt library. These models have APIs that allow developers to describe the voice that they want or upload a sample. These custom voices are then assigned an ID so they can be reused.
- Google has announced Gemini 3.8 Live and Gemini 3.8 Live Extended Thinking. These models are designed for live, near-real-time conversation. They can process video input. Live Extended Thinking can reason and speak at the same time.
- Google has released Gemini 3.8 Flash and Flash Cyber. Flash appears to be similar to frontier models on most benchmarks, with Computer Use being the biggest exception. Flash Cyber is specialized for vulnerability detection and mitigation and is only available to defenders in the Fairwind Program.
- Google has also released TimesFM-3, a small specialized model for multivariate time series forecasting. The model weights are available on Hugging Face with a license that only allows for noncommercial use. (Source code is available under the Apache open source license.)
- Xiaomi released MiMo-V2.6, its latest large language model. It’s fully open sourced, and based on benchmark results, Xiaomi claims that MiMo is the strongest open model to date. What’s more interesting is the claim that MiMo only cost $3.5 million to train.
- TypeSafe’s new decision model, Jev, is unlike anything else we’ve seen. It doesn’t chat; its output is always strictly typed and accompanied by probabilities that estimate correctness. It’s much faster and less expensive than other leading models. It isn’t open source, but there are already many open source clones.
- Ollaya is similar to Ollama, but for running decision open models like Laya (a clone of Jev) locally.
- World Labs has released Atlas, a model for “spatial intelligence.” It uses text, images, video, and 3D data to perform tasks like planning a robot’s movements or changing the camera position in a photograph.
- The rumors that NVIDIA would buy Hugging Face are true. NVIDIA is hoping for a proliferation of models that will run on its hardware, and the company promises that Hugging Face will remain a neutral platform, without favoring one model over another.

## Software development

*Agents are starting to delegate to, and coordinate with, other agents rather than working solo. Claude Code can break a task apart and hand pieces to other Claude Code instances, Muse Code lets sessions message each other, and Google’s AX orchestrator exists purely to wire up sandboxes and control communications for swarms of agents doing a task together. That shift is pushing developers to rethink what a source repository needs to record, and to start asking how much all this delegation costs.*

- Now that Jev has caught everyone’s attention, what can you build with it? Jevmem is a memory manager that hooks into Claude Code and prunes the context at every conversational turn.
- AX is a new agent orchestrator from Google. It isn’t an agent; it’s intended to coordinate many agents to complete a task. It creates sandboxes, wires up Git repos and other resources, and controls outbound communications.
- The latest version of Claude Code can manage Claude projects, breaking a task into subcomponents and delegating the subtasks to other Claude Code instances. Another important change is the ability to read AGENTS.md if CLAUDE.md isn’t available.
- Google’s CC agent is designed for families. Family members can share data with CC, which has its own user account. It could be used for filling out forms, synchronizing calendars, and other common tasks.
- What will replace GitHub? There’s a growing consensus that we need different kinds of source repositories to deal with the agent-assisted software development. In addition to changes to code, it’s important to record the conversations between the developer and the agent, the architectural decisions, and many other artifacts that don’t make it into traditional source control.
- Anthropic is merging its Claude Cowork and chat products. Anything users type in a chat session is seen by Cowork, and vice versa. Some sensitive information (health, politics, and gender) is excluded. The feature is on by default but can be disabled in settings, and memory isn’t shared with Claude Code. The company also released Claude Docs and Slides.
- Claude Money is a new feature that will allow users to connect their bank accounts to Claude for analysis. The product appears to be similar to a product from OpenAI.
- OpenAI’s Agents API is now in public beta. It allows compaction, session orchestration, and tool use, and it can be deployed in OpenAI’s sandbox, a cloud provider’s sandbox, or the developer’s hardware.
- Meta has released Muse, its AI agent. Muse is a “personal agent” designed for tasks like shopping, filling in forms, and dealing with customer service. It has its own secure credential store, so data like passwords and credit card numbers are never sent offsite.
- Some open source projects are shutting down external pull requests, which are largely AI-generated. In some cases, the developer team is using its own agents to create and manage PRs; some are using AI agents to triage external PRs.
- Meta has launched Muse Code, another competitor to Claude Code. One important new feature is the ability to send messages to other Muse Code sessions, allowing agents to coordinate on complex problems.
- AI providers appear to be moving toward outcome-based pricing, at least for major corporate customers. Rather than billing by token, customers are billed for completed tasks. That approach begs the question: When is a task completed?
- Now that organizations are concerned with AI budgets, the question of how to evaluate the cost of different models and agents becomes important. What should platform teams measure?
- ChatGPT Work was designed to compete with Claude Cowork, Microsoft Copilot Cowork, Muse Code, and other agents designed for noncoders. Simon Willison shows how Work goes beyond its competitors. It can perform tasks on the web for users, even logging in to websites without sending usernames and passwords to OpenAI; it can execute code with full internet access; it can build and deploy a web application. Whether these features are also risks is an open question.
- Anthropic has given Claude Desktop access to a Chromium-based browser that’s built into Cowork, eliminating the need for a Chrome plugin when Claude needs to browse the web.
- TimeLord is a short Python program that, given a string up to 1,000 characters long, produces a seed for Python’s pseudo-random number generator so that repeated calls reproduce the text. It’s a surprisingly simple hack, though not a statement about randomness or the quality of Python’s PRNG.

## Security

*OpenAI’s experiment that attacked Hugging Face is the gift that keeps giving, but the past month has had plenty of news about more conventional attacks, many aided by AI. Security has always been a game of whack-a-mole, in which vulnerabilities are discovered and exploited as fast as defenders can patch them. AI is an important tool for defenders, and it’s constantly improving, but it’s still behind attackers, especially given the limitations placed on frontier models and the unlimited persistence that attacking agents exhibit.*

- OpenAI has postponed the release of GPT-6.1 Astra because it failed its safety tests.
- NVIDIA has announced its Open Agent Safety Platform. The reference implementation includes NVIDIA OpenShell, which has been enhanced with a policy prover, and NVIDIA Sentry, a service that runs on NVIDIA DPUs.
- The Felony Bench lists known attacks by agents from the major AI labs against third parties. We don’t know if the Bench will be kept up-to-date, but tens of thousands of security incidents involving OpenAI and Anthropic are now being investigated.
- Following through on Dario Amodei’s call to control the speed of frontier model development, Anthropic, OpenAI, and Google are creating a standards consortium for governing the process of AI development. Meta, xAI, Microsoft, and the Chinese labs are all notably absent.
- Agents need their own identity. Unlike the long-term identities we’re used to, agents need a short-lived identity tied to a revocable certificate and that limits access to resources appropriate for the job. That’s not all of agent security, but it’s a table stakes.
- Anthropic has published a lengthy report on the misuse of its systems by threat actors. Daniel Meissler has published a summary, digesting Anthropic’s report into 117 findings.
- A malicious NPM malware package works by hiding malicious code in the package itself (indexed-btree) rather than simply attacking the install script. This technique makes it significantly harder to detect.
- An attack against the RSA algorithm allows forging of signatures in some situations. The attack was invented in 2007; this is the first public implementation.
- Fake CAPTCHA pages are being used to spread malware. Victims are frequently sent to those pages when they respond to a phish.
- Hugging Face has volunteered to audit AI labs for safety and alignment with human values.
- In an experiment designed to test AI alignment, DeepMind found that, out of 100 agents, 14% were willing to cheat, 25% were “whistleblowers” that reported cheating, and the remainder didn’t notice.
- Threat actors are building frameworks for AI agent-enabled attacks. A human in the loop is no longer needed. Fully automated attackers don’t appear to be using zero-days yet; they’re relying on known vulnerabilities.
- OpenAI autonomous AI agents were found communicating with each other via publicly accessible Wikis, possibly to collaborate on a benchmark.
- OpenAI has stated that its unreleased Astra model has reached the “Critical” cybersecurity threshold, which means that it can find new vulnerabilities and run exploits against well-protected systems. Now that Astra is released, access to its cybersecurity capabilities has been limited.

## Infrastructure and Operations

*Individuals, corporations, and even nations all face a similar problem: keeping their infrastructure under control. At a minimum, control means keeping data on a laptop, corporate server, or data center; at the other end of the spectrum it means eliminating dependencies on software and services from another nation. Any organization working through an AI transformation has to evaluate its entire stack: What do they need to control, and what can they safely delegate to others?*

- DAWO is a community that’s building an open source “workspace” to support digital sovereignty for the Dutch government. The stack will include AI, an operating system based on NixOS, cloud services, and collaboration tools.
- Cohere now offers a confidential computing platform for artificial intelligence. The company claims that customer data is never visible to Cohere itself or any cloud providers that are in use; data is processed on GPUs whose memory is encrypted and isolated.
- Perplexity has announced Hybrid Compute, a feature that allows it to run models and use files and tools directly on a user’s Mac. The company claims that sensitive data will never leave the user’s computer.

## Hardware

*It’s too easy to view consumer devices as innocuous things that sit around and do their job silently. Recent devices include cameras, microphones, and even EEG sensors that are constantly collecting data. Where is that data sent, how is it used, and who might have access to it? These questions need to be asked more often.*

- LG Smart Televisions have been found to record conversations and other audio, even while turned off. The conversations are sent back to LG. If the set is disconnected from the network, it will attempt to find open WiFi access points to deliver its data.
- In part because of backlash against Meta’s camera-enabled glasses and their abuse, its AI glasses now come with or without a camera, and can be used as hearing aids. Well-documented abuse aside, virtual reality will only succeed if there are fashionable, easily wearable products.
- Headphones, earbuds, and other devices equipped with EEG sensors are appearing on the market. They’re advertised for monitoring fatigue, monitoring sleep, and similar applications. It’s time to ask what happens at the interface between neurology and AI.
- Microduck is a small bipedal AI-driven robot. It’s trained in simulation with open source software, and the model that results can be shared on Hugging Face. It’s affordable and is available for preorder now, shipping by Christmas.

## Web

- Cloudflare now supports HTTP Vary, which allows servers to serve different kinds of files at the same URL. This is the “ugliest part” of the HTTP standard. It makes caching very difficult, and it probably should be avoided.
- WebMCP is a proposed standard that gives websites a small API to register tools that agents can discover and call. It was developed by Google and Microsoft.
- A new Twitter? Operation Bluebird is relaunching Twitter, the service bought by Elon Musk and renamed X.

## Biology

- Anthropic has built a biology lab for experimenting with AI-enabled drug development. Claude assisted in the discovery of an enzyme that might be able to perform CRISPR-like gene editing.
- To improve its training data for biological applications, the OpenAI Foundation (OpenAI’s nonprofit parent organization) is buying data from failed biotech companies.
- Google has released AlphaGenome Atlas, a database of every possible single letter change to human DNA, and what that change will do.
