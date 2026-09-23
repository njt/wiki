---
url: https://www.kenmuse.com/blog/removing-the-noise-for-better-prompts/
date_fetched: 2026-09-23
---

When you write a prompt for artificial intelligence (AI), it is natural to write as though you are asking another person for help. You may add a greeting, explain why you need something, or say “please.” Those habits are useful in a human conversation. They do not necessarily help a model complete a task.

A useful way to review a prompt is to ask what each phrase contributes. Does it define the goal, provide relevant context, add a constraint, describe the expected output, or explain how to verify the result? If it does none of those things, it may be *noise*. Noise is not always catastrophic, but it gives the model more language to interpret without giving it a better definition of success.

Under the hood, a large language model (LLM) calculates relationships between tokens – small pieces of text that the model processes – across the prompt using *attention*. The model does not treat every token the same way. Because it must process every token, irrelevant wording can add ambiguity and make the important instructions less prominent.

This is especially important when a prompt combines several requests. A model may be able to complete each request individually, yet still produce an unsatisfying result because it must decide how the requests relate while planning its response. An AI agent typically combines the model with tools and an execution loop, so unclear instructions can also affect which actions it takes.

## A realistic prompt with several problems

Consider this prompt:

```
Please update the release notes with the changes in this pull request.
Please review the package.json to gather the dependencies.
Please check dependencies for higher version numbers.
```
This is the kind of request I often see. It sounds reasonable at first, but it asks an AI agent to make several decisions that the author has not stated. It also combines multiple unrelated goals, requiring the model to decide how the requirements fit together while planning its response.

Is the dependency list supposed to be included in the release notes? Should the agent update the dependencies if they are outdated or should it only report what it finds? The prompt does not say, leaving it to the model to reason and infer the intended behavior.

That uncertainty is more important than the fact that the prompt contains three sentences. A longer prompt can be precise, and a short prompt can be unclear. The problem is that the instructions do not establish one clear outcome or explain how the outcomes fit together.

## Why “please” is noise here

Politeness matters when you are managing a relationship with another person. A model does not need reassurance in the way a person might, so politeness usually adds little task-specific information. In this example, each instance of “please” adds no goal, context, constraint, output format, or verification step. It is therefore noise.

The effect is subtle rather than absolute. Models predict an answer from patterns in the prompt and the surrounding context. In some situations, “please” means nothing and the model ignores it. In others, it can be treated “optionally” or “if it is possible”, especially when the task is already ambiguous.

Research has also found that excessive politeness can reduce accuracy by roughly 5 percent in some evaluations. That is, the noise impacts the model’s ability to get correct results.To be clear, removing “please” does not guarantee a better answer. It simply removes noise and unnecessary tokens.

The same test applies to other conversational filler:

- “As you know, this repository is a modern application with a complex architecture” is redundant. The agent can inspect the repository and the details do not affect any task. None of the information changes what the model should do or how it should work.
- “Please make sure everything is correct” asks for quality without naming a test, standard, or acceptance criterion. The model produced the result and sees its reasoning, so it may assume that is enough to satisfy the request.
- “You are an expert software developer with decades of experience.” The first part clearly defines a role, which helps the model adopt a perspective. The second part emphasizes that it is good at what it does, seemingly reinforcing “expert.” However, it does not add any concrete instructions or constraints for the task at hand, and “expert” is already clearly stated. Odds are it is noise.
- “Make any improvements you think are necessary.” This is a vague instruction and could lead the model into a loop where it continuously questions the code and previous steps, leading it to make changes without end. WIthout criteria for what to include and when to stop, it could either perform a small number of loops to improve the current code before stopping or enter an infinite loop of modifications.

Keep a phrase when it clarifies the goal, relevant context, constraints, expected output, or verification. Unless you are trying to balance specific behaviors you are observing, remove words that do not contribute to these elements.

## Why the dependency request does not belong here

The dependency sentences are not merely wordy versions of the release-note task. They introduce a separate workflow that has its own goals, context, constraints, and verification steps. It is not related to crafting release notes.

Release notes usually summarize user-facing changes for an audience. Dependency maintenance involves discovering available versions, applying a version policy, updating manifests and lock files (files that pin the exact versions installed), checking compatibility, and running validation. The first task is about communicating what changed in a pull request. The second is about changing or reviewing the project’s supply chain – the code and packages your project depends on.

While pointing to `package.json` does help it to zero in on specific dependencies, the final phrase, “check dependencies for higher version numbers,” leaves the model with a number of decisions to make:

- Does “higher” mean the latest patch, minor, or major version?
- Should prerelease versions be considered?
- Should direct dependencies be checked, or transitive dependencies too? A direct dependency is one your project lists itself; a transitive dependency is one brought in by another dependency.
- Should the agent report the versions or modify `package.json`and the lockfile?
- If the repository also versions other code, such as Actions workflows, should it be considered as well?
- Should it run tests, create a commit, or open a pull request for this work?

These are not details an agent should have to invent. The model may spend its effort guessing the relationship instead of completing either task reliably.

Teams should monitor dependencies constantly with a maintenance policy and prioritize security updates as they are discovered. As a result, the requests do not really belong in this release-note task. Tools like Dependabot can automate many dependency-related tasks, so you do not need an agent to reason about every recurring update. Use deterministic tools whenever you can.

## Rewrite the request around one outcome

Lets look at one possible rewrite focused solely on the release notes:

```
Update the release notes with summaries of user-visible changes in this pull request.
Follow the existing format and style.
```
Every sentence now has a job. The pull request is the source of truth. “User-visible” defines what belongs in the notes. Asking the agent to inspect an existing release-note file provides repository context, and including a short example can reduce the model’s guesswork.

And you don’t even need to say “please!” 😄

## Keep the signal and remove the noise

Before sending a prompt, consider if it:

- States the expected outcome
- Includes only context that affects or guides that outcome
- Defines constraints such as scope, versions, and files
- Explains how the result will be verified
- Minimizes unnecessary details and history
- Provides paths to referenced files and folders

The best prompt is not one enormous statement. It is a focused request that clearly defines the outcome you want and provides the necessary context. Every phrase or element should, as much as possible, contribute to driving that outcome. By doing that, the model can focus on delivering the result efficiently and accurately.
