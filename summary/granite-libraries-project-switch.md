---
url: https://research.ibm.com/blog/granite-libraries-project-switch
title: "Building AI more like software"
author: Mike Murphy
date_fetched: 2026-06-09
date_published: 2026-06-04
---

# Building AI more like software

Introducing Granite Libraries and Project Granite Switch, two new tools that bring the rigor and modularity of software engineering to LLMs.

It wasn't that long ago that the state of the art in AI was generating you an image that sort of looked like the thing you'd asked for. In the interceding years, we've seen a Cambrian explosion of capabilities, with AI that can now generate flawless text, run mission-critical enterprise workflows, or orchestrate agents to run entire apps autonomously.

But even with all these advances, the ways we build and interact with AI models is quite different than any other piece of software. Enterprise users want their models to be as accurate and efficient as possible, but getting there is often out of reach for most developers. One reason is that up until now, it's been incredibly difficult to break AI models down into the same kinds of plug-and-play building blocks we use for traditional software.

A modern software application is made up of smaller, self-contained pieces that work together, rather like an object composed of many LEGO bricks, instead of one giant block of clay. When something breaks, an engineer can find the offending module, fix it, test it, and redeploy — without touching the rest of the codebase. Capabilities are separated behind various interfaces, and part of the app can be developed by different teams, tested against their specifications, and replaced as needed. The application may serve one purpose, but its internals are not an undifferentiated mass.

Today's LLMs are modern marvels, able to answer questions about the capitals of countries as easily as they can decipher an earnings report. But every capability is diffused across the entire collection of parameter weights. To change the way the model reacts to a given situation, you either need to retrain the whole thing, or write extremely detailed and accurate prompts. None of these are quick solutions, and none of them allow multiple teams to work together to improve AI models just like they improve software systems.

IBM Research has been working on technologies that bring the rigor and modularity of software engineering to LLMs through an approach called generative computing.

"Models are just code with data, just a lot more data than code," said Luis Lastras, IBM Research's director of language and multimodal models. "We haven't learned the lessons of software for LLMs — we can build pieces separately."

IBM is launching a set of coordinated tools that bring us closer to the vision of generative computing. The idea of software modularity was the spark for **Granite Libraries**, a collection of adapters that can customize AI models for specialized tasks. It enables a model to quickly execute targeted tasks without having to retrain the entire model. The core idea is the "adapter function," which has a defined input and output, like a function in a software library.

An adapter function in this context is a small model adapter that's trained to generate a different type of output than a traditional model. Instead of producing open-ended text, these adapter functions carry out a specific task, whether that's scoring a document for relevance, rewriting a query, detecting a hallucination, or making a safety decision.

The team is also introducing **Project Granite Switch**, a toolkit for existing model architectures that enables them to dynamically manage the specialized components found in Granite Libraries. Coupled with the recently released models in Granite 4.1, and Mellea, IBM's open-source library for generative computing, developers now have a tool that turns unpredictable text generation into a reliable, deterministic programming function.

## Introducing Granite Libraries

Granite Libraries is designed to bring the same kind of customization to AI models that has made software so powerful.

IBM has released three libraries designed to support common enterprise workflows. The **RAG Library** includes adapters for key retrieval-augmented generation tasks such as query rewriting, answerability assessment, hallucination detection, and citation generation. The **Core Library** provides foundational capabilities, including requirement checks, certainty scoring, and contextual attribution. Rounding out the release, the **Guardian Library** enables models to perform in-line safety, factuality and policy checks directly, without a separate guardrail model. These Granite Libraries are available for all Granite 4.1 models.

Because these libraries are modular and independently trained, enterprises can adopt them as needed and add more capabilities incrementally, a bit like how software dependencies are managed today.

Each adapter function is trained to be an expert at one task. The requirement checker, for example, takes a model response and a set of constraints, and returns whether the constraints are satisfied. When Granite 4.1 3B is explicitly prompted to do this, it achieves 51% balanced accuracy on IFEval, a popular benchmark for instruction following. If the same model is equipped with the new Granite Library requirement-check adapter function, its accuracy jumps to 84%.

The adapter makes a small model comparatively better at a specific task than the base model could be through careful prompting alone. And Mellea is what allows these adapter functions to act like software: It automatically inserts the tags needed to activate specific adapters, strictly enforces formatting rules in real time, and packages everything as a standard Python function. This insulates the main application from the unpredictable nature of raw AI text.

With Granite Libraries, a Granite base model can call task-specific experts — low-rank adapters (LoRAs), or activated-LoRAs (aLoRAs) — that have been trained to perform a well-defined function through a software interface. This gives a smaller model the ability to perform narrow tasks on par with large generalist models — with much lower inference costs.

When a library adapter is active, the model can become single-mindedly excellent at that task. While the base model stays unchanged, its behavior can now be precisely prescribed as needed, with nearly no cost to switch aLoRAs in and out.

Below, you can see how much faster this process can be with Granite Libraries. This interactive is an animated replay of benchmarking two Granite Switch checkpoints — one using aLoRA, and one using standard LoRA — on the same multi-step RAG pipeline.

## The architecture layer: Project Granite Switch

Project Granite Switch is a new experimental toolkit available on GitHub that can be used to compose new models in minutes, similar to the way compilers can produce binaries from source code and software libraries.

Granite Switch allows a base Granite model and its adapter functions to act as a single model that activates them efficiently at inference time. It accomplishes this by adding a new "switching" layer to an existing core Granite model, and then gluing the adapter's weights to the base model, adding formatting tags, and preparing a new chat template. Instead of needing to spin up a brand-new AI model for every different task, Granite Switch dynamically flips the right adapter on and off exactly when it's needed. The base model is still accessible within Granite Switch, meaning the new capabilities are available without changing the underlying model in any way.

This independent switching layer allows both LoRAs and aLoRAs to run within vLLM, the open-source inferencing engine for large-scale deployments. In the real world, a single business task usually requires a chain of actions, like running safety checks, finding data, and verifying answers. And switching between different adapters forces the AI to clear its short-term memory and recalculate everything from scratch at each step, which slows things down. By using activated LoRA, a Granite Switch model can carry its memory forward from one step to the next without pausing to re-read it, dramatically speeding up multi-step workflows.

By inserting an extra transformer layer into the base model and commandeering its attention mechanism to read and save values related to the active adapter's state, a special control token signals to the model when it's time to switch experts. That token acts like a dispatcher at a railyard instructing which trains to go where, with the switching layer acting as the rails themselves.

## Granite 4.1: IBM's most capable models to date

The potential of Granite Libraries and Project Granite Switch would be lost without capable models to execute on top of. With Granite 4.1, IBM recently released its most performant suite of models yet.

The Granite 4.1 family is designed to punch above its weight class, with the 8B model matching or exceeding the performance of the previous Granite 32B mixture-of-experts (MoE) model and the 30B model competing with significantly larger models like Llama 3.3 70B on enterprise tasks. Small, performant models that can be quickly adapted to other tasks are far less costly to serve than a massive, generalist model that might be incompetent at narrow tasks.

By training on a relatively small amount of high-quality data, these models achieved competitive scores in tool calling and instruction following while maintaining lower latency and operational costs than many frontier reasoning models.

The release is part of a broader ecosystem that includes Granite Vision 4.1 for industry-leading table and chart extraction, as well as new speech and guardrail models. All were trained on approximately 15 trillion tokens and are released under the open-source Apache 2.0 license, supporting 12 major languages for global deployment.

## Toward composable AI

IBM has introduced Granite Libraries as part of a broader goal to make AI models as composable as software to deliver more value to enterprise users. By separating capabilities into modular components, developers can build AI systems that are easier to adapt, cheaper to operate, and more predictable in production.

Modularity won't solve every challenge of deploying generative AI at scale, but it offers a practical path toward more sustainable, enterprise-ready systems.

## Get started today

- [Granite Switch](https://github.com/generative-computing/granite-switch): Compose adapter functions into a deployable model
- [Granite Libraries](https://huggingface.co/collections/ibm-granite/granite-libraries): Browse and download adapter functions for RAG, safety, and core capabilities
- [Mellea](https://mellea.ai/): Call adapter functions as typed Python functions
- [Granite 4.1 models](https://huggingface.co/ibm-granite): The base models that power it all
